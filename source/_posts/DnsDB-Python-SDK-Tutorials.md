---
title: DnsDB-Python-SDK教程
date: 2016-10-08 12:45:24
tags: [DnsDB-Python-SDK, Python, 教程]
categories: DnsDB
---

# 目录

* [准备工作](#准备工作)
  - [环境](#环境)
  - [安装](#安装)
* [使用](#使用)
  - [验证身份](#验证身份)
  - [查询DNS](#查询DNS)
  - [获取所有DNS查询结果](#获取所有DNS查询结果)
  - [获取资源信息](#获取资源信息)
  - [配置代理](#配置代理)
    + [HTTP/HTTPS代理](#HTTP-HTTPS代理)
    + [SOCKS代理](#SOCKS代理)

# 准备工作

## 环境
1. **Python2** >= `2.7.9` 或 **Python3** >=`3.3.0`

## 安装
    
```shell
pip install --upgrade dnsdb-python-sdk
```

### 安装到Mac OS X
在Mac OS X使用上面的方法安装, 在使用时可能出现以下错误：
```python
requests.exceptions.SSLError: [SSL: SSLV3_ALERT_HANDSHAKE_FAILURE] sslv3 alert handshake failure (_ssl.c:590)
```
这是Mac OS X pyOpenSSL版本太旧导致, 您可以通过[homebrew](http://brew.sh)重新安装python解决，具体安装步骤如下：

1. 通过brew安装python

    ```shell
brew install python --with-brewed-openssl
    ```
2. 将python的bin路径加入到PATH环境变量中

    ```shell
export PATH=/usr/local/Cellar/python/2.7.11/bin/:$PATH
    ```
3. 创建符号链接

    ```shell
mkdir -p /usr/local/opt/python/bin/
ln -s /usr/local/Cellar/python/2.7.11/bin/python2.7 /usr/local/opt/python/bin/python2.7
    ```

5. 安装
    ```shell
pip install --upgrade dnsdb-python-sdk
    ```

# 使用

DnsDB Python API主要使用`dnsdb.clients.DnsDBClient`，以下内容主要都是讲该类的用法。

## 验证身份

`DnsDBClient`在使用前需要验证用户身份，您需要提供拥有开发者权限的DnsDB账号。

```python
from __future__ import print_function
from dnsdb.clients import DnsDBClient
from dnsdb.errors import AuthenticationError

client = DnsDBClient()
try:
    # 如果登陆失败则抛出AuthenticationError
    client.login("your username", "your password")
except AuthenticationError as e:
    print(e.message)
```
> 如果您还没有DnsDB账号, 你可以点击[这里](https://dnsdb.io/register)进行注册

> 购买API服务，请[查看这里](https://dnsdb.io/apiservice)

## 查询DNS

查询DNS记录可以使用`DnsDBClient.search_dns`方法, 该方法单次调用最多返回**30条查询结果**, **每成功调用一次扣除一次API查询次数**。

> `DnsDB.search_dns`至少需要指定`domain`，`host`或者`ip`中的一个参数，否则会抛出异常

查询某个顶级私有域名的DNS记录
```python
result = client.search_dns(domain='github.com')
```

查询指定host值的DNS记录
```python
result = client.search_dns(host='www.github.com')
```

查询解析到8.8.8.8的DNS记录
```python
result = client.search_dns(ip='8.8.8.8')
```
查询某个顶级私有域名的所有A记录
```python
result = client.search_dns(domain='github.com', dns_type='A)
```

指定`start`参数, 告诉client从第几条数据开始获取结果, `start`默认为`0`
```python
result = client.search_dns(domain='github.com', start=10)  # 从第十条开始取回
```
> 通过`start`最多可以查询到前10000条记录, 所以`start`最大为`9970`，如果您想获取所有查询结果请查看[获取所有DNS查询结果](#获取所有dns查询结果)

`result`是`Dnsdb.clients.SearchResult`的实例
```python
print(result.total)  # 查询结果总数
print(result.remaining_request)  # 剩余查询次数
print(len(result))  # 当前返回的结果数量
```
遍历查询结果

```python
for record in result:
    print(record.host,record.type, record.value)
```

## 获取所有DNS查询结果

如果您想获取某个DNS查询的所有结果，您可以使用`DnsDBClient.retrieve_dns`方法实现, 调用该方法不会扣除API请求次数。但是**遍历结果时会根据遍历的数据量扣除相应的API请求次数**。

`DnsDBClient.retrieve_dns`参数与`DnsDBClient.search_dns`相同, 但是不存在`start`参数

查询某个顶级域名的所有DNS记录
```python
result = client.retrieve_dns(domain='github.com')  # 这里不会扣除API请求次数
```

`result`是`dnsdb.clients.RetrieveResult`的实例。获取满足查询条件的结果总数
```python
total = len(result)  # 这里不会扣除API请求次数
```

遍历查询结果, 这里会根据你遍历的查询结果数扣除相应的API请求次数, 如果您提前结束遍历, 可能会减少扣除的API请求次数
```python
for record in result:
    print(record)
```

> 遍历`RetrieveResult`时， 它会自动多次调用`DnsDBClient.__retrieve_dns_once`， 用于获取查询结果， `DnsDBClient.__retrieve_dns_once`每次调用返回的最大查询数量与购买的API套餐相关，每成功调用一次则扣除一次API请求次数。


## 获取资源信息

通过`DnsDBClient.get_resources`获取当前账号的资源信息，资源信息包含了当前可用的API请求次数

```python
resources = client.get_resources()
print(resources.remaining_dns_request)
```

## 配置代理

### HTTP/HTTPS代理

```python
from dnsdb.clients import DnsDBClient

proxies = {
  'http': 'http://10.10.1.10:3128',
  'https': 'http://10.10.1.10:1080',
}
client = DnsDBClient(proxies=proxies)
```

代理服务器使用HTTP Basic Auth， 语法为`http://user:password@host/`:

```python
proxies = {'http': 'http://user:pass@10.10.1.10:3128/'}
```

### SOCKS代理

```python
from dnsdb.clients import DnsDBClient

proxies = {
    'http': 'socks5://user:pass@host:port',
    'https': 'socks5://user:pass@host:port'
}
client = DnsDBClient(proxies=proxies)
```
