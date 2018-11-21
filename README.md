# api 的部分文档


## 基础
请求的认证是基于请求头中的 x-user-token，只要不修改密码，一次获取即可长期使用。

## token 获取方式
**请求URL：** 

- ` https://api.mtdhb.org/user/login `
  
**请求方式：**

- POST 

**参数：** 

|参数名|必选|类型|说明|
|:----    |:---|:----- |-----   |
|account |  是  |    string   |  邮箱   |
|password |  是  |    string   |  密码   |


**返回示例**

``` 
{ 
     "code" : 0, 
     "message" : null, 
     "data" : { 
          "id" : 1234, 
          "avatar" : null, 
          "name" : null, 
          "mail" : "name@example.com", 
          "phone" : null, 
          "token" : "123123FSDAGSG234268734956237894658WEYRGUIJSDHFGJSKLDFHGLSDFKYOUSI829734569238475698SDYFGIOSUDFYGOIUY23578692348576234985YSUIDYFU", 
          "locked" : false 
     } 
 }

```


## 贡献饿了么 cookie
目前只支持一次提交一条。

**请求URL：** 
- ` https://api.mtdhb.org/user/cookie `
  
**请求方式：**
- POST 

**Header：**

```
x-user-token:123123FSDAGSG234268734956237894658WEYRGUIJSDHFGJSKLDFHGLSDFKYOUSI829734569238475698SDYFGIOSUDFYGOIUY23578692348576234985YSUIDYFU

```

**参数：** 

|参数名|必选|类型|说明|
|:----    |:---|:----- |-----   |
|value |  是  |    string   | 完整 cookie |
|application |  是  |    int   |   0: 美团，1: 饿了么，2: 饿了么星选   |


**返回示例**

``` 
{ 
     "code" : 0, 
     "message" : null, 
     "data" : { 
          "id" : 112902, 
          "value" : "perf_ssid=aaa; ubt_ssid=bbb; snsInfo[101204453]=ccc; SID=ddd;", 
          "nickname" : "name", 
          "headImgUrl" : "http://thirdqq.qlogo.cn/qqapp/101204453/ZSDGSDFGSDF/40", 
          "application" : 1, 
          "valid" : true, 
          "gmtCreate" : 1542774249762, 
          "gmtModified" : null 
     } 
 }

```

## 领取饿了么
现在只支持领到大包前面一个，需要手动点进去领取最大包。提交的手机号码会被忽略。
领取的流程时：先提交，得到领取的任务 id，等待几秒后查询领取结果，根据返回的 status、messag 判断领取状况。
另外，红包链接中的 sn 是 hex(订单号)，转换时注意精度。这一点可以用于不便复制红包链接的情况（如微信公众号）。

### 提交领取任务

**请求URL：** 
- ` https://api.mtdhb.org/user/receiving `
  
**请求方式：**
- POST 

**Header：**

```
x-user-token:123123FSDAGSG234268734956237894658WEYRGUIJSDHFGJSKLDFHGLSDFKYOUSI829734569238475698SDYFGIOSUDFYGOIUY23578692348576234985YSUIDYFU

```

**参数：** 

|参数名|必选|类型|说明|
|:----    |:---|:----- |-----   |
|phone |  是  |    string   | 可以是空（会被忽略），但不能少这个参数。 |
|url |  是  |    string   | 红包链接，至少要包含 sn |
|force |  是  |    int   | 是否强制领取（不检查数据库内是否领取过），建议为 1。 |


**返回示例**

``` 
{ 
     "code" : 0, 
     "message" : null, 
     "data" : { 
          "id" : 123456, 
          "urlKey" : "2a1561111111113c", 
          "url" : "https://h5.ele.me/hongbao/#lucky_number=0&sn=2a1561111111113c&theme_id=5", 
          "phone" : "", 
          "application" : 1, 
          "type" : null, 
          "status" : 0, 
          "price" : null, 
          "message" : null, 
          "gmtCreate" : 1542775139775, 
          "gmtModified" : null 
     } 
 }

```



###  查询领取结果

提交后等待三秒左右开始查询，领取未完成时 status 为 0。等待一秒左右再次查询，直到 status 为2，且 message 为 "已领取到最佳前一个红包。下一个是最大红包，请手动打开红包链接领取"，即可视作领取完成。
这部分有一些异常需要处理，但也不容易碰到。文档待完善。

**请求URL（末尾为提交时返回的id）：** 

- ` https://api.mtdhb.org/user/receiving/123456 `
**请求方式：**
- GET 

**Header：**

```
x-user-token:123123FSDAGSG234268734956237894658WEYRGUIJSDHFGJSKLDFHGLSDFKYOUSI829734569238475698SDYFGIOSUDFYGOIUY23578692348576234985YSUIDYFU

```



**返回示例**

``` 
{ 
     "code" : 0, 
     "message" : { 
     }, 
     "data" : { 
          "id" : 123456, 
          "urlKey" : "2a1561111111113c", 
          "url" : "https://h5.ele.me/hongbao/#lucky_number=0&sn=2a1561111111113c&theme_id=5", 
          "phone" : "", 
          "application" : 1, 
          "type" : 1, 
          "status" : 2, 
          "price" : null, 
          "message" : "已领取到最佳前一个红包。下一个是最大红包，请手动打开红包链接领取", 
          "gmtCreate" : 1542775642000, 
          "gmtModified" : 1542775650000 
     } 
 }

```

