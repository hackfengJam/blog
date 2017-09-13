## Nginx介绍

#### Nginx
         
- [http://nginx.org/](http://nginx.org/) ，C10k
- Igor Sysoev，Rambler Media； 
- engine X：nginx
- tengine，OpenResty


### 1.Nginx的特性

- 模块化设计、较好扩展性
- 高可靠性
  - master/worker
- 支持热部署
  - 不停机-更新配置文件、更换日志、更新服务器程序版本；（平滑升级/迁移）
- 低内存消耗
  - 10000个keep-alive连接模式下的非活动连接仅要好2.5M内存；event-driven，aio，mmap；

### 2.功能

- 基本功能
  - 静态资源的web服务器；
  - http协议的反响代理服务；
  - pop3，smpt，imap4等邮件协议的反向代理；
  - 能缓存打开的文件（元数据）、支持FastCGI（php-fpm），uWSGI（Python Web Framwork）等协议
  - 模块化（非DSO机制），过滤器zip，SSI，SSL；
- Web服务相关的功能
  - 虚拟主机（server）、keepalive、访问日志（支持基于日志缓存提高其性能）、url rewrite、路径别名、基于IP及用户的访问控制、支持速率限制及并发数限制……

### 3.Nginx的基本架构：

<strong> master/worker： </strong>  

- 一个master进程，可生成一个或多个进程；
- 事件驱动：epoll(Linux),kqueue(FreeBSD),/dev/poll(Solarls)
  - 消息通知：select，poll，rt signals
  - 支持sendfile，sendfile64
  - 支持AIO，mmap
- master：加载配置文件，管理worker进程、平滑升级，...
- worker：http服务、http代理，fastcgi代理，...

### 4.模块类型：

- 核心模块：core module
- Standard HTTP modules
- Optional HTTP modules
- 3rd party modules

### 5.用来做什么？

- 静态资源的web服务器；
- http服务器的反向代理；
- 等等

