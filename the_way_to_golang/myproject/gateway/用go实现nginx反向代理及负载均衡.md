# 用go实现nginx反向代理及负载均衡

## 1.介绍

### 1.1 正向代理与反向代理：

[Forward-Proxy-vs-Reverse-Proxy](https://www.jscape.com/blog/bid/87783/Forward-Proxy-vs-Reverse-Proxy) / [正向代理与反向代理的区别](http://blog.csdn.net/m13666368773/article/details/8060481)

### 1.2 负载均衡：
[维基百科](https://en.wikipedia.org/wiki/Load_balancing_(computing)) / [百度百科](https://baike.baidu.com/item/%E8%B4%9F%E8%BD%BD%E5%9D%87%E8%A1%A1/932451?fr=aladdin)

## 2.实现

### 2.1 负载均衡算法

#### 2.1.1 数据结构：
<pre>
type Convert struct {
	Req_type   string   // (host / location)
	Backends string
	Address string
	Protocol string
	Weight float32
	Cur_weight float32
}
</pre>

#### 2.1.2 负载均衡实现：
1. 我在globalvar里面定义了两个全局变量：```globalvar.go```

	<pre>
	var Cache map[string][]models.Convert
	var L_Lock *sync.RWMutex
	</pre>

2. 轮询算法实现```polling.go```
 
	<pre>
	func GetNextServerConvert(host string) (convert models.Convert, err error){
		globalvar.L_Lock.Lock()
		defer globalvar.L_Lock.Unlock()
	
		var converts []models.Convert
		converts = globalvar.Cache[host]
		if converts == nil || len(converts) == 0 {
			converts = GetConvertsByHost(host)
			if converts == nil || len(converts) == 0 {
				return convert, errors.New("not found convert can handle request from host:" + host)
			}
		}
		globalvar.Cache[host] = converts
		convert = getNextServerConvert(converts, len(converts))
		globalvar.Cache[host] = converts
		return convert, nil
	}
	</pre>

剩下的回头在写......
