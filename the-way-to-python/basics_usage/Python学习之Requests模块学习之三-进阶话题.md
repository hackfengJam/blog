# Requests模块学习之三进阶话题

<pre>
GitHub API
URL:https://developer.github.com/
</pre>


## 3.1 HTTP认证

![Alt text](./images/51-1.png)

### 1.基本认证
<pre>
def base_auth():
    """基本认证
    """
    response = requests.get(construct_url('user'), auth=('hackfeng624@gmail.com', 'xxxxxx'))
    print response.text
    print response.request.headers

>>>
Authorization': 'Basic xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx', 'Connection': 'keep-alive', 'Accept-Encoding': 'gzip, deflate', 'Accept': '*/*', 'User-Agent': 'python-requests/2.18.4'}
200

</pre>

### 结果1

![Alt text](./images/51-2.png)

这个结果其实就是一个base64的简单加密，因此很不安全

![Alt text](./images/51-3.png)


### 2.OAUTH认证

![Alt text](./images/51-4.png)

#### 在GitHub上面设置一个
![Alt text](./images/51-5.png)
#### 从GitHub开发者api上找到认证方式
![Alt text](./images/51-6.png)
<pre>
def base_oauth():
    headers = {'Authorization': 'token xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx'}
    # user/emails
    response = requests.get(construct_url('user/emails'), headers=headers)
    # response = requests.get(construct_url('user/emails'),
    # params={"access_token": "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"})

    print response.request.headers
    # print response.text
    print response.status_code
</pre>

### 3.通过继承requests.auth.AuthBase进行OAUTH认证

![Alt text](./images/51-7.png)
<pre>
from requests.auth import AuthBase

class GithubAuth(AuthBase):

    def __init__(self, token):
        self.token = token

    def __call__(self, r):
        # requests 加 headers
        r.headers['Authorization'] = ' '.join(['token', self.token])

        return r


def oauth_advanced():
    auth = GithubAuth('xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx')
    print auth
    response = requests.get(construct_url('user/emails'), auth=auth)
    print response.text

>>>
<__main__.GithubAuth object at 0x0000000002E7DAC8>
[{"email":"hackfeng624@gmail.com","primary":true,"verified":true,"visibility":"public"}]

</pre>
### 结果2
![Alt text](./images/51-8.png)


## 3.2 代理（Proxy）
![Alt text](./images/52-1.png)
![Alt text](./images/52-2.png)


### 安装requests对socksv5支持，cmd下输入: pip install requests[socksv5]
![Alt text](./images/52-3.png)

### 代码实现
<pre>
#!/usr/bin/env python
# -*- coding:utf-8 -*-

__Author__ = "HackFun"
__Date__ = '2017/9/28 11:28'

import requests

proxies = {'http': 'socks5://127.0.0.1:1080', 'https': 'socks5://127.0.0.1:1080'}
url = 'https://www.facebook.com'
response = requests.get(url, proxies=proxies, timeout=10)
</pre>

## 3.3 Requests库-Session和Cookies

### 3.3.1 Cookies
![Alt text](./images/53-1.png)

### 3.3.2 Session
![Alt text](./images/53-2.png)





## 参考

[慕课网-Python-走进Requests库](http://www.imooc.com/learn/736)