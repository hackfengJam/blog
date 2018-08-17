# nginx.conf 配置

---
## nginx.conf
<pre>

main配置段

event{
  ...
}

http{
    ...
   server{
       server_name
           root
           location /uri/ {
               ... 
           }
    }
    server{
        ... 
    }
}

</pre>
### 1.main配置段

### 2.event
<pre>
event{
  ...
}
</pre>
<pre>
1.worker_connections #;
    没个worker进程能够响应的最大并发请求数量；

整个nginx接受最大并发连接数 = 
worker_processes（worker进程数） * worker_connections（单个worker能最多接受多少个并发请求）

2.use[epoll| rgs| select| poll];
    定义使用的事件模型；建议让nginx自动选择；

3.accept_mutex [on| off];  (mutex--英译-->互斥)
    （使序列化的响应新请求）
    各worker接受用户的请求的负载均衡器；启用时，表示用于让多个worker轮流地、序列化的响应新请求；

4.lock_file /PATH/TO/LOCK_FILE;

</pre>

### 3.http配置段
<pre>
http{
    ...
}
</pre>

#### 3.1 套接字或主机相关的指令
<pre>
1.server(){
     list PORT;
     server_name NAME;
     root /PATH/TO/DOCUMENTROOT;
 }
    需要多个虚拟主机，只需要多个server即可；
    例：server(){
            ...
            listen 8080;
            server_name nodel.hackfun.com;
            root /data/www/vhost2;
        }
       ...
    注意：
        （1）基于port：
                 listen 指令监听在不同的端口；
        （2）基于hostname：
                 server_name指令指向不同主机名；

2.listen
    listen address [:port] [default_server] [ssl] [http2| spdy]
    listen port [default_server] [ssl] [http2: spdy]

    例:
    server{
        listen 80 default_server;
        server_name www.c.org;
        root /data/www/vhost2;
    }
    default_server: 设置默认虚拟主机；用基于IP地址，或使用了任意不能对应于任何一个server的name时返回站点；
    ssl: 用于限制只能通过ssl连接提供服务（即只服务https）
    spdy: SPDY protocol(speedy),在编译时，编译了spdy模块的情况下，用于支持SPDY协议；
    http2: http version 2;

3.server_name NAME [...];（可以多个）
    后可跟一个或多个主机名；名称还可以使用通配符和正则表达式（二者一定要分开使用）；
    （1）首先做精确匹配，例如：www.hackfun.cn
    （2）左侧通配符
    （3）右侧通配符
    （4）正则表达式：例如：-^.*\.hackfun\.cn$;
    （5）default_server

4.tcp_nodelay on| off;
    对keepalive模式下的连接是否使用TCP_NODELAY选项；（ligoals算法合并多个小报文，等待多个报文）；

5.tcp_nopush on| off;
    是否启用TCP_NOPUSH(FREEBSE)或TCP_CORK(Linux)选项；仅在sendfile为on时有用;

6.sendfile on| off;
    是否启用sendfile功能；

7.root
    设置web资源的路径映射；用于指明请求的URL所对应的文档的目录路径；

    例：
    server{
        root /data/www/vhost2;
    }
    http://www.hackfun.cn/images/logo.jpg--> /data/www/vhost/images/logo.jpg

    server{
        server_name www.hackfun.cn;
        location /images/ {
            root /data/imgs/;
            ...
        }
    }
    http://www.hackfun.cn/images/logo.jpg--> /data/imgs/images/logo.jpg

8.location [=| ~| ~*| ^~] uri{...}
  location @name{...}
    功能：允许根据用户请求的URI来匹配定义的个location，匹配到时，
    此请求将被相应的location快中的配置所处理；简言之，即用于为需要用到专用的配置uri提供特定配置；

	例:
    server{
        ...
        server_name www.hackfun.cn;
        root /data/www;
        (location /{
             ...
        })
        location /admin{
            ...
        }
    }
    注：
    =：URI的精确匹配；
    ~：做正则表达式匹配，区分字符大小写；
    ~*：做正则表达式匹配，不区分字符大小写；
    ^~：URI的左半部分匹配，不区分字符大小写；

    匹配优先级：精确匹配=、^-、~或~*、不带符号的URL；

9.ailas
    只能用于location配置段，定义路径别名；
    
    例：
    location /images/{
        ...
        root /data/imgs/;
    }
    location /images/{
        ...
        alias /data/imgs/;
    }
    注意：
        root指令：路径为对应的location的"/"这个URL;
                 /images/test.jpg--> /data/imgs/images/test.jpg;
        alias指令：路径对应的location的"/url"这个URL;
                 /images/test.jpg-> /data/imgs/test.jpg;
10.index
    index file...;
    默认主页

11.error_page
<pre>
    Syntax: error_page code... [=[response]] uri;
    Default: ---
    Context: http, server, location, if in location
</pre>
    根据http状态码重定向至页面
    例：
    error_page 404 /404.html
    error_page 500 502 503 /50x.html
    error_page 404 =200 /empty.jpg

    error_page 500 502 503 504 /50x.html;
    location = /50x.html{
        root htm;
    }

12.try_files
<pre>
    Syntax: try_files file... uri;
            try_files file... =code;
    Default: ---
    Context: server, location
</pre>
    尝试查找第1至第N-1个文件，第一个即为返回给请求者的资源；若1~N-1个资源均不存在，则跳转至最后一个URI（由其他location定义，而应该匹配至其它location，否则会导致死循环）；














</pre>
