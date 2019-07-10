## Git workflow 流程

### 目录

[1. Git workflow 流程](#1-Git-workflow-流程)  
- [1.1 分支约定](#11-分支约定)  
- [1.2 标签约定](#12-标签约定)  
- [1.3 分支和标签名命名规范](#13-分支和标签名命名规范)  

[2. 日常操作规范](#2-日常操作规范)  
- [2.1 相同分支 pull 使用 git pull --rebase](#21-相同分支-pull-使用-git-pull---rebase)  
- [2.2 不同分支merge使用 git merge BRANCH --no-ff](#22-不同分支merge使用-git-merge-branch---no-ff)  
- [2.3 提交信息 commit message 尽可能用中文描述有意义的信息](#23-提交信息-commit-message-尽可能用中文描述有意义的信息)  
    
[3. 持续集成与 Docker 镜像](#3-持续集成与-Docker-镜像)

[4. 典型开发流程介绍](#4-典型开发流程介绍)

[5. 总结](#5-总结)


### 1. Git workflow 流程

#### 1.1 分支约定

- __功能开发分支feature/\*__

以 feature/* 作为主要的功能开发分支，一般对应 JIRA 上的任务，相对独立完整的一些列功能。

- __主开发分支 dev__

以 dev（或 develop）作为主开发分支和测试分支。功能开发分支开发完成以后合并入该分支，并提交测试。
可以提交修复一些次要 bug，解决开发分支合并产生的 bug。

- __生成环境主分支 master__

以 master 作为生成环境主分支。可以 merge hotfix/\* 紧急修复分支 和 pre/\* 预发环境标签，
master 中的代码始终处于可发布状态，最新的 master 提交一般为生产环境使用的代码。

- __生产环境紧急修复分支 hotfix/\*__

以 hotfix/\* 作为生成环境紧急修复 bug 分支。修复生产环境发现的 bug，经验证后合入 master 和 dev 分支。

#### 1.2 标签约定

- __预发环境标签 pre/\*__

以 pre/\* 作为预发环境标签，标签应打在dev分支上，一般情况下以预发日期作为名字，如 pre/20180724 。


- __生产环境标签 release/\*__

以 release/\* 作为生产环境标签，标签应打在master分支上，一般情况下以发布日期作为名字，如 release/20180724 。

#### 1.3 分支和标签名命名规范

feature, hotfix, pre, release 后跟随 __/__ ，有些的规范可能使用 __-__ 连字符。

这是由于 / 可以直接在系统文件系统中映射为目录，在一些git工具中也会根据 / 进行分组归类，相比连字符有更好的组织。

在目录下，可以使用连字符，如 feature/add-some-field 。

### 2. 日常操作规范

#### 2.1 相同分支 pull 使用 git pull --rebase

多人协作共同在一个功能分支开发时，他人有提交并且已推到服务器（点 A、B、C），
本人也已有提交但暂未推送到服务器（点 D、E），直接使用 git pull 会产生 merge log（点F） 。

```
A - B - C - F
  \ D - E /
```

该 merge log 对于开发流程来说并没有明显的意义，为保持 git 树的清爽及高效，我们使用 ```git pull --rebase``` 模式避免生成 merge log。

```
A - B - C
          \ D' - E'
```

即本人已提交但暂未推送到服务器的提交，重新以服务器最新点（点C）为基点展开。如果本人的提交与远程提交产生了冲突，需要先解决冲突后完成 rebase。

为了防止git pull时忘记加上 --rebase，可以设置全局使用 rebase 模式 pull：

```
git config --global pull.rebase true
```


#### 2.2 不同分支merge使用 git merge BRANCH --no-ff

即 no-fast-forward，非快进方式 merge，并生成 merge commit 。
我们从主分支（图例A）中创建开发分支开始开发新功能，产生了一些提交（点B、C、D）。

```
 A -------- B(修改a文件) ------ C(修改b文件) ----- D(修改c文件)
(dev)                                           (feature/some)
```
此时，回到主分支且主分支没有任何新修改，欲将开发分支合入主分支，
直接 merge 开发分支将产生快进效果（fast-forward），主分支会直接指向开发分支所在的位置（点D）


```
A -------- B(修改a文件) ------ C(修改b文件) ---------- D(修改c文件)
                                                 (dev)(feature/some)
```

与相同分支使用 rebase 不产生 merge log 相反，我们希望产生 merge log，使用 ```git merge feature/some --no-ff``` ：

```
A -----------------------------------------------------------------------E(merge log:修改a,b,c文件)
   \    B(修改a文件) ----------C(修改b文件) -------------D(修改c文件)    /
                                                     (feature/some)      (dev)
```

该 merge log（点E）会概括总结先前的所有编辑，同时我们在主分支的视角下不需要去关注开发分支所作的琐碎的修改。
在调试时，如果发现了开发分支存在问题，我们可以很容易的回到起点（点A）去验证问题是否由于开发新功能而导致的。

#### 2.3 提交信息 commit message 尽可能用中文描述有意义的信息

如“修复因请求参数 page 和 size 没有验证，数据出错时无出错页面的 bug” ，而不是“修复bug”，例外是 “修复拼写错误” 或 “typo”。

信息中尽可能带有 JIRA任务 编号，Gitlab 中会自动链接到对应的 JIRA 任务中。
如果是在开发分支上开发的，可以不用每一条提交都添加，在合入 dev 分支填写 merge log 时加上即可。

一个好的提交信息应该是他人通过阅读信息能大致了解这条提交解决了什么问题。

### 3. 持续集成与 Docker 镜像

Git 提交推送到服务器后，会触发持续集成。服务器端的代码一般会自动编译对应的 Docker 镜像以供各个环境使用。

dev 分支生成 Docker 镜像标签为 testing 的镜像，pre/\* 和 release/\* 标签生成对应的 pre-\* 、 release-\* 镜像。（Docker镜像的标签中不支持 / 等符号，会自动转为 - 符号）

.gitlab-ci.yml 可以参考：

- [Getting started with GitLab CI/CD](https://docs.gitlab.com/ee/ci/quick_start/)

- [通过 .gitlab-ci.yml配置任务](https://segmentfault.com/a/1190000010442764)

### 4. 典型开发流程介绍

接到新需求，以“企业信息新增“城市”属性”为例：

```

# 一般情况下以当前的dev作为起点，并从服务器拉取最新版本
git checkout dev && git pull
git checkout -b feature/company-add-city-field #创建新分支并切换到新分支
 
 
...开发过程
git add ./
git commit -m "Company新增city字段"
git push --set-upstream origin feature/company-add-city-field #根据需要推送到服务器
 
 
...开发过程
git add ./
git commit -m "POST /company接口支持city字段"
git pull --rebase #多人协作时，如果他人已有提交，以rebase模式拉取分支最新状态
git push origin feature/company-add-city-field #根据需要推送到服务器
 
 
# 开发完成，回到dev分支合并开发成果
git checkout dev && git pull
git merge feature/company-add-city-field --no-ff # 非快进方式合并(no-fast-forward)并生成merge commit
#编辑必要的提交信息，如 [CB-123] 企业信息新增城市属性
git push origin dev #推送到服务器dev分支，服务器将自动打包该项目的testing镜像并等待测试环境部署
 
 
# 该测试版本通过验收，准备进入预发环境
git tag pre/20180724 # 在dev分支上打预发标签
git push origin pre/20180724 #推送标签到服务器，服务器将自动打包该项目的预发镜像并等待预发环境部署
 
 
# 该预发版本通过验收，准备正式发布
git checkout master && git pull
git merge pre/20180724 --no-ff
# 编辑必要的提交信息，如 [CB-123] 企业信息新增城市属性，[CB-124]XXX，[CB-125]XXX
git push origin master
git tag release/20180726  # 在master分支上打发布标签
git push origin release/20180726 #推送标签到服务器，服务器将自动打包该项目的生产镜像并等待生产环境部署

```

### 5. 总结

- 分支约定  
    - feature: 功能开发分支，开发主要使用的分支  
    - dev: 开发主版本，feather 分支开发完成 merge 过来  
    - release: dev 分支满足发布条件后建立 release 分支，此版本发不到测试／生产环境（本文未描述）  
    - master： stable 版本，release 之后两天从 release merge 过来  
    - hotfix: 生产上发现问题直接从 release 分支上建 hotfix 分支，问题解决后直接走发布流程。之后更改 merge 到 mainline 和 dev 分支上  

- 标签约定

- 日常操作规范
    - 相同分支pull使用 git pull --rebase  
    - 不同分支merge使用 git merge BRANCH --no-ff  
    - 提交信息commit message尽可能用中文描述有意义的信息  


### 感谢

- [A successful Git branching model](https://nvie.com/posts/a-successful-git-branching-model/)

- [Getting started with GitLab CI/CD](https://docs.gitlab.com/ee/ci/quick_start/)

- [通过 .gitlab-ci.yml配置任务](https://segmentfault.com/a/1190000010442764)
