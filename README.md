# 淘宝x-sign算法解密分析

大家都知道只要有了x-sign基本上所有事情都可以干，包括但不仅限于商品信息，商品评价，秒杀活动等等  
本文将演示如何获取淘宝商品评价信息，以iphone11为例 [https://detail.tmall.com/item.htm?id=602659642364](https://detail.tmall.com/item.htm?id=602659642364) 联系qq: 951263019

## 抓包分析
通过charles手机抓包分析得出评价获取参数为如下几个：  
url：http://guide-acs.m.taobao.com/gw/mtop.taobao.rate.detaillist.get/4.0  
参数：data={"rateType":"","hasPic":"1","foldFlag":"0","pageNo":"1","pageSize":"10","auctionNumId":"602659642364"}  
头信息：有好多头信息，最重要的x-sign  

## 获取x-sign值
通过调用接口 http://xsign.iceftech.com/api/sign 获取，对应参数以json流发送，下图是postman请求对应的参数  
![1.png](https://upload-images.jianshu.io/upload_images/9203913-c4458576f9e9acda.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 获取评价内容
使用postman构造请求发送获取结果  
![2.png](https://upload-images.jianshu.io/upload_images/9203913-c2b698b5821bfca1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 技术支持
联系qq: 951263019  
接口费用: 100次/元，10元起购  
完整的技术解决方案：4999元  
