# 淘宝x-sign算法解密分析

大家都知道只要有了x-sign基本上所有事情都可以干，包括但不仅限于商品信息，商品评价，秒杀活动等等  
本文将演示如何获取淘宝商品评价信息，以iphone11为例 [https://detail.tmall.com/item.htm?id=602659642364](https://detail.tmall.com/item.htm?id=602659642364)   
联系qq: 951263019

## 抓包分析
通过charles手机抓包分析得出评价获取参数为如下几个：  
url：http://guide-acs.m.taobao.com/gw/mtop.taobao.rate.detaillist.get/4.0  
参数：data={"rateType":"","hasPic":"1","foldFlag":"0","pageNo":"1","pageSize":"10","auctionNumId":"602659642364"}  
头信息：有好多头信息，最重要的x-sign  

## 签名接口调用
先放一个postman的图片
![image.png](https://upload-images.jianshu.io/upload_images/9203913-1ff775893867c7c8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


## 使用说明：
1. 图片中的请求地址并不是真实的请求地址，需要联系qq获取
2. 发请求的时候必须是post json格式，可能需要协议头Content-Type:application/json
3. token是接口校验参数，需要联系qq获取
4. 获取签名的时候参数值都不需要转义，发请求抓数据的时候可能需要转义
5. 所有参数必须使用```""```包起来，必须是字符串

## 参数说明
1. data：就是参数data，需要注意转义符号，最好是md5之后的，像图中一样。如果data是md5之后的数据，那么需要设置参数```dataMd5:"true"```，否者接口将再次进行md5处理导致签名错误。如果data是原始值比如```"{\"pageSize\":\"10\",\"foldFlag\":\"0\",\"hasPic\":\"1\",\"pageNo\":\"1\",\"auctionNumId\":\"602659642364\"}"```，那么参数dataMd5可以设置为false或者不传，当data是原始值的时候需要注意转义符号，否者将可能签名出错，推荐使用md5之后的值
2. appKey：默认```"21646297"```，且只能是```"21646297"```，不管你是其他什么值，即使传了其他值也会被换掉，也就是说可以不传这个参数
3. pv：默认```"6.3"```，可选```"6.2"```或者```"6.3"``` 
4. useMiniWua：默认```"false"``` 需要```x-mini-wua```的时候，设置为```"true"```，当```pv="6.3"```的时候，都是带```x-mini-wua```返回值的
5. useWua：默认```"false"``` 需要```wua```的时候，设置为```"true"```
6. ```如有其他疑问，或者需要帮助的请联系qq: 951263019```

## 返回值说明
返回值有x-sign，x-mini-wua，wua等
需要自己发请求测试，此处不再说明

## python 版本demo
运行条件: python3 + requests 库
```python
import hashlib
import json
import time
from urllib import parse

import requests

# 请求签名的校验参数
token='1234'

# 请求淘宝的api接口
api='mtop.taobao.rate.detaillist.get'

# 请求淘宝的api接口版本
v='4.0'

# 获取签名信息 参数data 要注意转义符号
data = "{\"pageSize\":\"10\",\"foldFlag\":\"0\",\"hasPic\":\"1\",\"pageNo\":\"1\",\"auctionNumId\":\"602659642364\"}"

# 淘宝请求地址，所有post请求基本都可以转换成get请求
taobao_url = "http://acs.m.taobao.com/gw/"+api+"/"+v+"?data=" + data

# sid 需要登陆淘宝后获取，一般长度为32位
sid = '1d80'

# pv 对应签名版本
pv = '6.2'

# appKey 固定值，固定为'21646297'
appKey = '21646297'

# ttid 手机淘宝版本号
ttid = '701186@taobao_android_9.1.0'

# x_features 对应api功能
x_features = '27'

# deviceId 设备id，可44位随机
deviceId = 'AiN8kvbdEkyD1P3CqFhYct1a7PmMab5dj804e192TcrV'

# utdid 淘宝uuid，可以24位随机
utdid = 'XcZJFF61gMADAep76BgfX2AD'


def get_sign(data):
    url = 'http://api.xsign.com/api/sign'  # 获取签名的地址
    params = {
        'sid': sid,
        'data': hashlib.md5(data.encode(encoding='UTF-8')).hexdigest(),  # 获取签名需要将data进行md5处理，以方便数据传输
        'api': api,
        'v': v,
        't': str(int(time.time())),  # 时间戳，单位：秒
        'ttid': ttid,
        'utdid': utdid,
        'deviceId': deviceId,
        'x-features': x_features,
        'appKey': appKey,
        'pv': pv,
        'dataMd5': 'true',  # 如果data进行了md5处理，那这里需要设置为'true',
        'token': token,  # 获取签名参数所需的token值，长度为32位，有需要的请联系qq951263019申请
    }
    header = {'Content-Type': 'application/json'}  # 协议头
    return requests.post(url, data=json.dumps(params), headers=header).json()


def fetch_rate(sign_res, data):
    header = {
        'x-sid': sid,
        'user-Agent': 'MTOPSDK%2F3.1.1.7+%28Android%3B4.4.2%3BXiaomi%3BMI+6%29',
        'x-appkey': appKey,
        'x-ttid': parse.quote(ttid), # 进行URL encode处理，比如@符号要转换成%40
        'x-devid': parse.quote(deviceId),
        'x-features': x_features,
        'x-utdid': parse.quote(utdid),
        'x-pv': pv,
        'x-location': '%2C',  # 如果获取参数的时候有参数lat和lng，那这里就是lng%2Clat，本例为空则设置为%2C
        'x-t': sign_res['x-t'],
        'x-sign': sign_res['x-sign'],
    }
   
    return requests.get(taobao_url, headers=header).json()


if __name__ == '__main__':
    sign_res = get_sign(data)  # 获取x-sign有需要的请联系qq951263019
    print(sign_res)
    rate = fetch_rate(sign_res, data)
    print(rate)


```

## 技术支持
联系qq: 951263019  
