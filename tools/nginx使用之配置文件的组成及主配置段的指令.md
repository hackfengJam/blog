# Nginx使用之配置文件解析
---
> 配置文件的组成、主配置段的指令
<!-- TOC -->


1. [配置文件的组成部分](#1配置文件的组成部分)
   - 1.1.[主配置文件](#11-主配置文件nginxconf)
   - 1.2.[fastcgi的配置文件](#12-fastcgi的配置文件) 
2. [配置命令](#2配置指令必须以分号结尾)
   - 2.1 [指令格式](#21-指令格式)
   - 2.2 [支持使用变量](#22-支持使用变量)
3. [.配置文件组织结构](#3配置文件组织结构)
   - 3.1[http配置段](#31-http配置段)
   - 3.2[main配置段](#32-main配置段)
<!-- /TOC -->
---

##  1.配置文件的组成部分
- nginx.conf
- mine.type
- fastcgi_params
- proxy.conf
- site.conf

---

### 1.1 主配置文件：nginx.conf

- conf.d/\*.conf
- include conf.d/\*.conf  (include host/\*.conf) 
- /etc/nginx/conf.d/\*.conf

### 1.2 fastcgi的配置文件

- fastcgi_params
- uwsgi_params

---

## 2.配置指令（必须以分号结尾）

### 2.1 指令格式

<strong>directive value1 (value2...);</strong>

### 2.2 支持使用变量
- 内置变量：由模块引入；
- 自定义变量： set variable value;
- 引用变量： $variable

## 3.配置文件组织结构


<pre>
#main block
	event{
	  …               
	}
	http{
	  … 
	}
</pre>

---

### 3.1 http配置段
<pre>
	http{
	    …
	   (upstream{
	      …  
	   }) #（定义一个负载均衡器的容器的，反向代理，可不写）
	   server{
	      server_name    
	         root
	         alias
	         location /uri/{
				
	         }(location可以出现多个，映射机制不同于alias)…
	
	    }
	    server{
	          …
	    }
	}
</pre>
### 3.2 main配置段
<pre>
<strong>类别：</strong>
 1.正常运行必备的配置；
 2.优化性能相关的配置；
 3.用于调试，定位问题的配置；

I.正常运行必备的配置
 1.user USERNAME [GROUPNAME];
    指定用于运行worker进程的用户和组
    例：  user nginx nginx
 2.pid /PATH/TO/PID_FILE;
    指定nginx进程的pid文件路径
    例：  pid /var/run/nginx.pid
 3.worker_rlimit_nofile #; （number of files）
    指定一个（/单个）worker进程所能够打开的最大文件描述符数量
 4.worker_rlimit_sigpending #;
    指定每个用户能够发往worker进程的信号的数量；  

II.优化性能相关的配置
 1.worker_processes #;
    worker进程的个数；
    通常应该为物理CPU核心数量减1(或减2，比较好)；
    可以为"auto"，实现自动设定；
 2、worker_cpu_affinity CPUMASK CPUMASK ...;
    （affinity--英译-->密切关系）
    CPUMASK:
	    0001
	    0010
	    0100
	    1000
    例： worker_cpu_affinity 00000001 00000010 00000100;
         （这样绑定cpu，并不能隔离cpu，隔离CPU后面会讲）
 3.worker_priority nice（默认为0）;
    [-20,19]  大->小    

III.用于调试，定位问题的配置
 1.daemon  on|off
    是否已守护方式启动nginx；
 2.master_process on|off
    是否以master/worker模型运行nginx；     
 3.error_log /PATH/TO/ERROR_LOG level;
    错误日志文件及其级别；
    出于调试需要，可以设定为debug；
    但debug仅在编译时使用了"--with-debug"选项是才有效；
<strong>
注：整个nginx接受最大并发连接数 = 
worker_processes（worker进程数） * worker_connections（单个worker能最多接受多少个并发请求）
</strong>
</pre>
