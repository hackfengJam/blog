## Golang 操作 etcd（下）

### 目录

[1. Lease 租约 实现 KV 过期](#1-lease-租约-实现-kv-过期)

[2. Watch 监听目录变化](#2-watch-监听目录变化)

[3. Op 取代 Get，Put，Delete 方法](#3-op-取代-getputdelete-方法)

[4. 事务txn 实现分布式锁](#4-事务txn-实现分布式锁)

--- 

[相关博文](#相关博文)

#### 1. Lease 租约 实现 KV 过期


<details>
<summary>查看源代码</summary>

```golang
package main

import (
	"context"
	"fmt"
	"go.etcd.io/etcd/clientv3"
	"time"
)

func main() {
	var (
		config         clientv3.Config
		client         *clientv3.Client
		err            error
		lease          clientv3.Lease
		leaseGrantResp *clientv3.LeaseGrantResponse
		leaseId        clientv3.LeaseID
		kv             clientv3.KV
		putResp        *clientv3.PutResponse
		getResp        *clientv3.GetResponse
		keepResp       *clientv3.LeaseKeepAliveResponse
		keepRespChan   <-chan *clientv3.LeaseKeepAliveResponse
	)

	config = clientv3.Config{
		Endpoints:   []string{"127.0.0.1:2379"}, // 集群列表
		DialTimeout: 5 * time.Second,
	}

	// 建立一个客户端
	if client, err = clientv3.New(config); err != nil {
		fmt.Println(err)
		return
	}

	// 申请一个 lease（租约）
	lease = clientv3.NewLease(client)

	// 申请一个 10 秒的租约
	if leaseGrantResp, err = lease.Grant(context.TODO(), 10); err != nil {
		fmt.Println(err)
		return
	}

	// 拿到租约ID
	leaseId = leaseGrantResp.ID

	// 自动续租
	if keepRespChan, err = lease.KeepAlive(context.TODO(), leaseId); err != nil {
		fmt.Println(err)
		return
	}

	// 处理续约应答的协程
	go func() {
		for {
			select {
			case keepResp = <-keepRespChan:
				if keepResp == nil {
					fmt.Println("租约已失效")
					goto END
				} else { // 每秒会续约一次，所以会收到一次应答
					fmt.Println("收到自动续约应答", keepResp.ID)
				}
			}
		}
	END:
	}()

	// 获得KV API子集对象
	kv = clientv3.NewKV(client)

	// Put一个KV，让它与租约关联起来，从而实现10s后自动过期
	if putResp, err = kv.Put(context.TODO(), "/cron/lock/job1", "", clientv3.WithLease(leaseId)); err != nil {
		fmt.Println(err)
	}

	fmt.Println("写入成功", putResp.Header.Revision)

	// 定时的看key是否已过期
	for {
		if getResp, err = kv.Get(context.TODO(), "/cron/lock/job1"); err != nil {
			fmt.Println(err)
			return
		}

		if getResp.Count == 0 {
			fmt.Println("kv 已过期")
			break
		}
		fmt.Println("kv 尚未过期", getResp.Kvs)
		time.Sleep(2 * time.Second)
	}

}

```

</details>


#### 2. Watch 监听目录变化


<details>
<summary>查看源代码</summary>

```golang

package main

import (
	"context"
	"fmt"
	"go.etcd.io/etcd/clientv3"
	"go.etcd.io/etcd/mvcc/mvccpb"
	"time"
)

func main() {

	var (
		config             clientv3.Config
		client             *clientv3.Client
		err                error
		kv                 clientv3.KV
		watcher            clientv3.Watcher
		getResp            *clientv3.GetResponse
		watchStartRevision int64
		watchRespChan      <-chan clientv3.WatchResponse
		watchResp          clientv3.WatchResponse
		event              *clientv3.Event
	)

	// 客户端配置
	config = clientv3.Config{
		Endpoints:   []string{"127.0.0.1:2379"}, // 集群列表
		DialTimeout: 5 * time.Second,
	}

	// 建立连接
	if client, err = clientv3.New(config); err != nil {
		fmt.Println(err)
		return
	}

	// KV
	kv = clientv3.NewKV(client)

	// 模拟etcd中KV的变化
	go func() {
		for {
			kv.Put(context.TODO(), "/cron/jobs/job7", "i am job7")

			kv.Delete(context.TODO(), "/cron/jobs/job7")

			time.Sleep(1 * time.Second)
		}
	}()

	// 先GET 到 当前的值，并监听后续变化
	if getResp, err = kv.Get(context.TODO(), "/cron/jobs/job7"); err != nil {
		fmt.Println(err)
		return
	}

	// 当前存在key
	if len(getResp.Kvs) != 0 {
		fmt.Println("当前值：", string(getResp.Kvs[0].Value))
	}

	// 当前etcd集群事务ID，单调递增
	watchStartRevision = getResp.Header.Revision + 1

	// 创建一个watcher
	watcher = clientv3.NewWatcher(client)

	// 启动监听
	fmt.Println("从该版本向后监听：", watchStartRevision)

	ctx, cancelFunc := context.WithCancel(context.TODO())
	time.AfterFunc(5*time.Second, cancelFunc)

	watchRespChan = watcher.Watch(ctx, "/cron/jobs/job7", clientv3.WithRev(watchStartRevision))
	//watchRespChan = watcher.Watch(context.TODO(), "/cron/jobs/job7", clientv3.WithRev(watchStartRevision))

	// 处理kv变化事件
	for watchResp = range watchRespChan {
		for _, event = range watchResp.Events {
			switch event.Type {
			case mvccpb.PUT:
				fmt.Println("修改为：", string(event.Kv.Value), "Revision", event.Kv.CreateRevision, event.Kv.ModRevision)
			case mvccpb.DELETE:
				fmt.Println("删除了", "Revision", event.Kv.ModRevision)
			}
		}
	}
}


```

</details>



#### 3. Op 取代 Get，Put，Delete 方法


<details>
<summary>查看源代码</summary>

```golang

package main

import (
	"context"
	"fmt"
	"go.etcd.io/etcd/clientv3"
	"time"
)

func main() {

	var (
		config clientv3.Config
		client *clientv3.Client
		err    error
		kv     clientv3.KV
		putOp  clientv3.Op
		getOp  clientv3.Op
		opResp clientv3.OpResponse
	)

	// 客户端配置
	config = clientv3.Config{
		Endpoints:   []string{"127.0.0.1:2379"}, // 集群列表
		DialTimeout: 5 * time.Second,
	}

	// 建立连接
	if client, err = clientv3.New(config); err != nil {
		fmt.Println(err)
		return
	}

	kv = clientv3.NewKV(client)

	// 创建Op: operation
	putOp = clientv3.OpPut("/cron/jobs/job8", "123123")

	// 执行OP
	if opResp, err = kv.Do(context.TODO(), putOp); err != nil {
		fmt.Println(err)
		return
	}

	// kv.Do

	//kv.Put
	//kv.Get
	//kv.Delete

	fmt.Println("写入Revision", opResp.Put().Header.Revision)

	// 创建Op
	getOp = clientv3.OpGet("/cron/jobs/job8")

	// 执行OP
	if opResp, err = kv.Do(context.TODO(), getOp); err != nil {
		fmt.Println(err)
		return
	}

	// 打印
	fmt.Println("createRevision", opResp.Get().Kvs[0].CreateRevision)
	fmt.Println("数据Revision", opResp.Get().Kvs[0].ModRevision)
	fmt.Println("数据value", string(opResp.Get().Kvs[0].Value))

}

```

</details>


#### 4. 事务txn 实现分布式锁

<details>
<summary>查看源代码</summary>

```golang

package main

import (
	"context"
	"fmt"
	"go.etcd.io/etcd/clientv3"
	"time"
)

func main() {
	// 乐观锁

	var (
		config         clientv3.Config
		client         *clientv3.Client
		err            error
		key            string
		lease          clientv3.Lease
		leaseGrantResp *clientv3.LeaseGrantResponse
		leaseId        clientv3.LeaseID
		keepRespChan   <-chan *clientv3.LeaseKeepAliveResponse
		keepResp       *clientv3.LeaseKeepAliveResponse
		ctx            context.Context
		cancelFunc     context.CancelFunc
		kv             clientv3.KV
		txn            clientv3.Txn
		txnResp        *clientv3.TxnResponse
	)

	// 客户端配置
	config = clientv3.Config{
		Endpoints:   []string{"127.0.0.1:2379"}, // 集群列表
		DialTimeout: 5 * time.Second,
	}

	// 建立连接
	if client, err = clientv3.New(config); err != nil {
		fmt.Println(err)
		return
	}

	// lease 实现锁自动过期：
	// op 操作
	// txn 事务：if else then

	// 1. 上锁（创建租约，自动续约，拿着租约去抢占一个key）

	// 申请一个 lease（租约）
	lease = clientv3.NewLease(client)

	// 申请一个 5 秒的租约
	if leaseGrantResp, err = lease.Grant(context.TODO(), 5); err != nil {
		fmt.Println(err)
		return
	}

	// 拿到租约ID
	leaseId = leaseGrantResp.ID

	// 准备一个用于取消自动续租的context
	ctx, cancelFunc = context.WithCancel(context.TODO())

	// 确保函数退出后，自动续约会停止
	defer cancelFunc()
	defer lease.Revoke(context.TODO(), leaseId)

	// 自动续租
	if keepRespChan, err = lease.KeepAlive(ctx, leaseId); err != nil {
		fmt.Println(err)
		return
	}

	// 处理续约应答的协程
	go func() {
		for {
			select {
			case keepResp = <-keepRespChan:
				if keepResp == nil {
					fmt.Println("租约已失效")
					goto END
				} else { // 每秒会续约一次，所以会收到一次应答
					fmt.Println("收到自动续约应答", keepResp.ID)
				}
			}
		}
	END:
	}()

	// 抢Key：if 不存在key,then 设置它，else 抢锁失败
	kv = clientv3.NewKV(client)

	// 创建事务
	txn = kv.Txn(context.TODO())

	// 定义事务

	// 如果key不存在
	key = "/cron/lock/job9"
	txn.If(clientv3.Compare(clientv3.CreateRevision(key), "=", 0)).
		Then(clientv3.OpPut(key, "xxx", clientv3.WithLease(leaseId))).
		Else(clientv3.OpGet(key)) // 否则抢锁失败

	// 提交事务
	if txnResp, err = txn.Commit(); err != nil {
		fmt.Println(err)
		return
	}

	// 判断是否抢到了锁
	if !txnResp.Succeeded {
		fmt.Println("锁被占用", string(txnResp.Responses[0].GetResponseRange().Kvs[0].Value))
		return
	}

	// 2. 处理业务
	fmt.Println("处理任务")
	time.Sleep(5 * time.Second)

	// 在锁内，很安全

	// 3. 释放锁（取消自动续约，释放租约）

	// defer 会租约释放，关联的KV就被删除了
}

```

</details>


### 相关博文

- [什么是 etcd?](/tech/distributed/etcd/etcd_study_1_what_is_etcd.md)

- [etcd 功能与原理](/tech/distributed/etcd/etcd_function_and_principle.md)

- [Golang 操作 etcd（上）](/tech/distributed/etcd/etcd_usage_golang_1.md)


#### 感谢

[etcd 官网](https://etcd.io/)

[Golang - 开发分布式任务调度](/design/golang_crontab/golang_crontab.md)

