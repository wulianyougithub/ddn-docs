---
id: ddn-http-api
title: Http api
sidebar_label: Http api
original_id: ddn-http-api
---

# DDN HTTP API文档

## **1 API使用说明**   
### **1.1 请求过程说明**   
1.1 构造请求数据，用户数据按照DDN提供的接口规则，通过程序生成签名，生成请求数据集合；       
1.2 发送请求数据，把构造完成的数据集合通过POST/GET等提交的方式传递给DDN；       
1.3 DDN对请求数据进行处理，服务器在接收到请求后，会首先进行安全校验，验证通过后便会处理该次发送过来的请求；       
1.4 返回响应结果数据，DDN把响应结果以JSON的格式反馈给用户，每个响应都包含success字段，表示请求是否成功，成功为true, 失败为false。 如果失败，则还会包含一个error字段，表示错误原因；       
1.5 对获取的返回结果数据进行处理； 

### **1.2 点对点传输tansport[安全的api]**
#### **说明**   
/peer相关的api，在请求时都需要设置一个header  

 - key为magic，testnet value:0ab796cd, mainnet value:5f5b3cf5  
 - key为version，value为''  

#### **普通交易**   
ddn系统的所有写操作都是通过发起一个交易来完成的。 
交易数据通过一个叫做ddn-js的库来创建，然后再通过一个POST接口发布出去

POST接口规格如下：
payload为ddn-js创建出来的交易数据
接口地址：/peer/transactions  
请求方式：post   
支持格式：json  

#### **案例1 设置二级密码**   
请求参数说明： 

|名称	|类型   |必填 |说明              |   
|------ |-----  |---  |----              |   
|transaction|json|Y|ddn-js.signature.createSignature生成的交易数据|

返回参数说明：   

|名称	|类型   |说明              |   
|------ |-----  |----              |   
|success|boolean  |是否成功 |  
   
   
请求示例：   
```js   
var ddn = require('ddn-js');  
var password = 'measure bottom stock hospital calm hurdle come banner high edge foster cram';  
var secondPassword  = 'erjimimashezhi001';  
var transaction = ddn.signature.createSignature(password,secondPassword);       
console.log(JSON.stringify(transaction))  
{"type":1,"amount":0,"fee":500000000,"recipientId":null,"senderPublicKey":"3e6e7c90571b9f7dabc0abc2e499c2fcee8e436af3a9d5c8eadd82ac7aeae85f","timestamp":5328943,"asset":{"signature":{"publicKey":"27116db89cb5a8c02fb559712e0eabdc298480d3c79a089b803e35bc5ef7bb7b"}},"signature":"71ef98b1600f22f3b18cfcf17599db3c40727c230db817f610e86454b62df4fb830211737ff0c03c6a61ecfd4a9fcb68a30b2874060bb33b87766acf800e820a","id":"15605591820551652547"}   

// 将上面生成的设置二级密码的交易数据通过post提交给DDN server
curl -H "Content-Type: application/json" -H "nethash:0ab796cd" -H "version:''" -k -X POST -d '{"transaction":{"type":1,"amount":0,"fee":500000000,"recipientId":null,"senderPublicKey":"3e6e7c90571b9f7dabc0abc2e499c2fcee8e436af3a9d5c8eadd82ac7aeae85f","timestamp":5328943,"asset":{"signature":{"publicKey":"27116db89cb5a8c02fb559712e0eabdc298480d3c79a089b803e35bc5ef7bb7b"}},"signature":"71ef98b1600f22f3b18cfcf17599db3c40727c230db817f610e86454b62df4fb830211737ff0c03c6a61ecfd4a9fcb68a30b2874060bb33b87766acf800e820a","id":"15605591820551652547"}}' http://47.92.0.84:8001/peer/transactions   
```   
   
JSON返回示例：   
```js  
{
    "success":true  //二级密码设置成功
}	
``` 

#### **案例二 转账**   
请求参数说明：  

|名称	|类型   |必填 |说明              |   
|------ |-----  |---  |----              |   
|transaction|json|Y|ddn-js.transaction.createTransaction生成的交易数据|

返回参数说明：   

|名称	|类型   |说明              |   
|------ |-----  |----              |   
|success|boolean  |是否成功 |  
   
   
请求示例：   
```js   
var ddn = require('ddn-js');   
var targetAddress = "16358246403719868041";  
var amount = 100*100000000;   //100 DDN
var password = 'measure bottom stock hospital calm hurdle come banner high edge foster cram';  
var secondPassword  = 'erjimimashezhi001';  
var message = ''; // 转账备注

// 其中password是在用户登录的时候记录下来的，secondPassword需要每次让用户输入
// 可以通过user.secondPublicKey来判断用户是否有二级密码，如果没有，则不必输入，以下几个交易类型类似
var transaction = ddn.transaction.createTransaction(targetAddress, amount, message, password, secondPassword || undefined);       
JSON.stringify(transaction)
'{"type":0,"amount":10000000000,"fee":10000000,"recipientId":"16358246403719868041","message":"","timestamp":37002975,"asset":{},"senderPublicKey":"8065a105c785a08757727fded3a06f8f312e73ad40f1f3502e0232ea42e67efd","signature":"bd0ed22abf09a13c1778ebfb96fc8584dd209961cb603fd0d818d88df647a926795b5e3c51e23f6ed38648169f4e4c912dd854725c22cce9bbdc15ec51c23008","id":"de72b89312c7d128db28611ed36eab2ff0136912c4a67f97342417c942b055cf"}'

// 将上面生成的转账操作的交易数据通过post提交给DDN server
curl -H "Content-Type: application/json" -H "nethash:0ab796cd" -H "version:''" -k -X POST -d '{"transaction":{"type":0,"amount":10000000000,"fee":10000000,"recipientId":"16358246403719868041","message":"","timestamp":37002975,"asset":{},"senderPublicKey":"8065a105c785a08757727fded3a06f8f312e73ad40f1f3502e0232ea42e67efd","signature":"bd0ed22abf09a13c1778ebfb96fc8584dd209961cb603fd0d818d88df647a926795b5e3c51e23f6ed38648169f4e4c912dd854725c22cce9bbdc15ec51c23008","id":"de72b89312c7d128db28611ed36eab2ff0136912c4a67f97342417c942b055cf"}}' http://47.92.0.84:8001/peer/transactions
```   
   
JSON返回示例：   
```js  
{
    "success":true,  //转账成功
    "transactionId":"a95c3a5bda15f3fd38295950268c234e922aae97cf803dd8c38c73a6ccf7c561"
}		
``` 



---  
   
## **2 接口**   
## **2.1 账户accounts**   
   
### **2.1.1 登录**   
#### **2.1.1.1 本地加密后再登陆（推荐使用）**   
接口地址：/api/accounts/open2/   
请求方式：post   
支持格式：json   
接口备注：根据用户密码在本地客户端用js代码生成公钥    

请求参数说明：   

|名称	|类型   |必填 |说明              |   
|------ |-----  |---  |----              |   
|publicKey |string |Y    |DDN账户公钥      |   
   
返回参数说明：   

|名称	|类型   |说明              |   
|------ |-----  |----              |   
|success|boolean  |是否登陆成功      |    
|account|json   |账户信息          |    
请求示例：   
  
```js
var secret = 'DDN账户密码'  //在浏览器内存中保留
var DdnJS = require('ddn-js');  //ddn-js具体安装方法见附录 
var publicKey = DdnJS.crypto.getKeys(secret).publicKey;  //根据密码生成公钥 
// var address = DdnJS.crypto.getAddress(publicKey);   //根据公钥生成地址

// 将上面生成的数据通过post提交到DDN server   
curl -X POST -H "Content-Type: application/json" -k -d '{"publicKey":"bd1e78c5a10fbf1eca36b28bbb8ea85f320967659cbf1f7ff1603d0a368867b9"}' http://47.92.0.84:8001/api/accounts/open2/   
```   
   
JSON返回示例：   
```js   
{   
	"success": true,   
	"account": {   
		"address": "D2zThPTQZDNQqXbe5tikDQ24YyCQTCpbSC",  //DDN地址   
		"unconfirmedBalance": 19480000000,  //未确认和已确认的余额之和，该值大于等于balance   
		"balance": 19480000000, //余额   
		"username": ""  // 账户昵称
		"publicKey": "bd1e78c5a10fbf1eca36b28bbb8ea85f320967659cbf1f7ff1603d0a368867b9",    //公钥   
		"unconfirmedSignature": false,   
		"secondSignature": true,    //二级签名   
		"secondPublicKey": "edf30942beb74de5ed6368c792af8665e9636f32a5f1c9377bcdc3b252d3f277",  //二级密码公钥   
		"multisignatures": [],    
		"u_multisignatures": [],   
		"lockHeight: ": 0 // 锁仓高度    
	},   
	"latestBlock": {   
		"height": 114480,   //当前节点最新区块高度     
		"timestamp": 4471890   
	},   
	"version": {   
		"version": "1.0.0",   //当前节点版本号    
		"build": "17:27:19 09/12/2016", //构建日期   
		"net": "testnet"    //区块链类型，是主链还是测试链   
	}   
}   
```   
   
#### **2.1.1.2 本地不加密直接登陆**   
接口地址：/api/accounts/open/   
请求方式：post   
支持格式：json   
接口备注：将密码传入到server端，根据生成的地址去查询账户信息。不推荐在公网坏境使用！ 
请求参数说明：   

|名称	|类型   |必填 |说明              |   
|------ |-----  |---  |----              |   
|secret |string |Y    |DDN账户密码       |   
   
返回参数说明：   

|名称	|类型   |说明              |   
|------ |-----  |----              |   
|success|boolean  |是否登陆成功      |    
|account|json   |账户信息          |    
   
请求示例：   
```bash   
curl -X POST -H "Content-Type: application/json" -k -d '{"secret":"fault still attack alley expand music basket purse later educate follow ride"}' http://47.92.0.84:8001/api/accounts/open/   
```   
   
JSON返回示例：   
```js   
{   
    "success": true,    
    "account": {   
        "address": "D2zThPTQZDNQqXbe5tikDQ24YyCQTCpbSC",    
        "unconfirmedBalance": 19480000000,    
        "balance": 19480000000,    
        "publicKey": "bd1e78c5a10fbf1eca36b28bbb8ea85f320967659cbf1f7ff1603d0a368867b9",    
        "unconfirmedSignature": false,    
        "secondSignature": true,    
        "secondPublicKey": "edf30942beb74de5ed6368c792af8665e9636f32a5f1c9377bcdc3b252d3f277",    
        "multisignatures": [ ],    
        "u_multisignatures": [ ]   
    }   
}   
```   
### **2.1.2 根据地址获取账户信息**   
接口地址：/api/accounts   
请求方式：get   
支持格式：urlencoded   
请求参数说明：   

|名称	|类型   |必填 |说明              |   
|------ |-----  |---  |----              |   
|address |string |Y    |用户地址,最小长度：1      |   
   
返回参数说明：   

|名称	|类型   |说明              |   
|------ |-----  |----              |   
|success|boolean  |是否成功获得response数据 |    
|account|json  |账户信息      |    
|latestBlock|json  |该节点最新的区块信息      |    
|version|json  |版本相关信息      |    
   
请求示例：   
```bash   
curl -k -X GET http://47.92.0.84:8001/api/accounts?address=D2zThPTQZDNQqXbe5tikDQ24YyCQTCpbSC   
```   
   
JSON返回示例：   
```js   
{   
	"success": true,   
	"account": {   
		"address": "D2zThPTQZDNQqXbe5tikDQ24YyCQTCpbSC",  //DDN地址   
		"unconfirmedBalance": 19480000000,  //未确认和已确认的余额之和，该值大于等于balance   
		"balance": 19480000000, //余额   
		"publicKey": "bd1e78c5a10fbf1eca36b28bbb8ea85f320967659cbf1f7ff1603d0a368867b9",    //公钥   
		"unconfirmedSignature": false,   
		"secondSignature": true,    //二级签名   
		"secondPublicKey": "edf30942beb74de5ed6368c792af8665e9636f32a5f1c9377bcdc3b252d3f277",  //二级密码公钥   
		"multisignatures": [],    
		"u_multisignatures": [],   
		"lockHeight: ": 0 // 锁仓高度    
	},   
	"latestBlock": {   
		"height": 114480,   //当前节点最新区块高度     
		"timestamp": 4471890   
	},   
	"version": {   
		"version": "1.0.0",   //当前节点版本号    
		"build": "17:27:19 09/12/2016", //构建日期   
		"net": "testnet"    //区块链类型，是主链还是测试链   
	}   
}   
```   
### **2.1.3 获取账户余额**   
接口地址：/api/accounts/getBalance   
请求方式：get   
支持格式：urlencoded   
请求参数说明：   

|名称	|类型   |必填 |说明              |   
|------ |-----  |---  |----              |   
|address |string |Y    |用户地址,最小长度：1      |   
   
返回参数说明：   

|名称	|类型   |说明              |   
|------ |-----  |----              |   
|success|boolean  |是否成功获得response数据 |    
|balance|integer  |余额      |    
|unconfirmedBalance|integer|未确认和已确认的余额之和，该值大于等于balance|   
   
   
请求示例：   
```bash   
curl -k -X GET 'http://47.92.0.84:8001/api/accounts/getBalance?address=DQ9vRnmaZACJ43vnhZyZmCX6dfksvMaqFR'   
```   
   
JSON返回示例：   
```js   
{
	"success": true,
	"balance": 20560710000000,
	"unconfirmedBalance": 20560710000000
}  
```   
   
### **2.1.4 根据地址获取账户公钥**   
接口地址：/api/accounts/getPublickey   
请求方式：get   
支持格式：urlencoded   
请求参数说明：只有给别人转过账，db中才会存取公钥，否则是查不到的。btc也是这样

|名称	|类型   |必填 |说明              |   
|------ |-----  |---  |----              |   
|address |string |Y    |用户地址,最小长度：1      |   
   
返回参数说明：   

|名称	|类型   |说明              |   
|------ |-----  |----              |   
|success|boolean  |是否成功获得response数据 |    
|publicKey|string  |公钥      |    
   
请求示例：   
```bash   
curl -k -X GET 'http://47.92.0.84:8001/api/accounts/getPublickey?address=D792yYKHLNSpsEoRT5c3whgmCq2VYpam55'   
```   
   
JSON返回示例：   
```js   
{   
	"success": true,   
	"publicKey": "0095b28997a33bb4f16b62523bcc1902179f2a7b5a3dd83980da5c1cbae854d6"   
}   
```   
   
### **2.1.5 生成公钥**   
todo
接口地址：/api/accounts/generatePublickey   
请求方式：post   
支持格式：json   
请求参数说明：   

|名称	|类型   |必填 |说明              |   
|------ |-----  |---  |----              |   
|secret |string |Y    |DDN账户密码      |   
   
返回参数说明：   

|名称	|类型   |说明              |   
|------ |-----  |----              |   
|success|boolean  |是否成功获得response数据 |    
|publicKey|string  |公钥      |    
   
请求示例：   
```bash   
curl -k -H "Content-Type: application/json" -X POST -d '{"secret":"fault still attack alley expand music basket purse later educate follow ride"}' 'http://47.92.0.84:8001/api/accounts/generatePublickey'   
```   
   
JSON返回示例：   
```js   
{   
	"success": true,   
	"publicKey": "bd1e78c5a10fbf1eca36b28bbb8ea85f320967659cbf1f7ff1603d0a368867b9"   
}   
```   

### **2.1.6 生成新账户**   
接口地址：/api/accounts/new   
请求方式：get   
支持格式：无   
请求参数：无   

返回参数说明：   

|名称	|类型   |说明              |   
|------ |-----  |----              |   
|success|boolean  |是否成功获得response数据 |    
|secret|string  |密码      |    
|publicKey|string  |公钥      |    
|privateKey|string  |私钥      |    
|address|string  |地址      |    
   
   
请求示例：   
```bash   
curl -k -X GET 'http://47.92.0.84:8001/api/accounts/new'   
```   
   
JSON返回示例：   
```js  
{
	"success": true,
	"secret": "home feature exact vague trust garage elegant palace rare cover alley repeat", // 密码 
	"publicKey": "762c006b5b73af09a0c49c0cd7c18ee9da1aa376b08d8c09d50a227e3b3f36f5", // 公钥
	"privateKey": "66f6c0029dbacb45c40669e8e0f4fdc121d9aa44a9fa1839972fe11190865f19762c006b5b73af09a0c49c0cd7c18ee9da1aa376b08d8c09d50a227e3b3f36f5",
	"address": "DFGweM6iHjTTLnMqzMkguspTEf44FySBpD" // 地址
}
```    

### **2.1.7 获取账户排行榜前100名**   
接口地址：/api/accounts/top   
请求方式：get   
支持格式：无   
请求参数说明：如果不加请求参数则返回持币量前100名账户信息  

|名称	|类型   |必填 |说明              |   
|------ |-----  |---  |----              |   
|limit |integer |N    |限制结果集个数，最小值：0,最大值：100   |  
|offset|integer  |N      |偏移量，最小值0  |  

返回参数说明：   

|名称	|类型   |说明              |   
|------ |-----  |----              |   
|success|boolean  |是否成功获得response数据 |    
|accounts|json  |账户信息元组，每个元素包含地址、余额、公钥      |    
   
   
请求示例：   
```bash   
curl -k -X GET 'http://47.92.0.84:8001/api/accounts/top?limit=5&offset=0'  //返回前5名账户信息
```   
   
JSON返回示例：   
```js   
{
  "success": true,
  "accounts": [{
    "address": "DPesT3fMLkDVHvqLaXR3YzGkdw16vgKqPj",
    "balance": 9977192780000000,
    "publicKey": "70d0cbd2c3dccfaaa65acfa5689dbc3656ff0807cd1611e88d854fec07845ac1"
  }, {
    "address": "DQ9vRnmaZACJ43vnhZyZmCX6dfksvMaqFR",
    "balance": 20560710000000,
    "publicKey": "3b760b4182bd0c0b09737984421847757170d698375c713db500808f6cde5845"
  }, {
    "address": "DAXYT6ucAbwzm4z7Z44YoVphtULdEGPdGM",
    "balance": 4157762217799,
    "publicKey": "32fb95258499f091696fe525032c45f5bb113c0cffe7135ec8ecb8a9ac0daa86"
  }, {
    "address": "DqLGmWLxtawABxgBrcn5Zmg86v3Aq9Jnb",
    "balance": 4156962297007,
    "publicKey": "85e6257516a57f80549f214715727115032e25a9077f019ef5d94f2f0500facc"
  }, {
    "address": "DNuixyo4SSyZcuTey46VVEyayUSzuxtXcA",
    "balance": 4155363009875,
    "publicKey": "7ed04166f563c5a71cc05236f7327f32edb10839b65f63ec2eb5aecef4eccdca"
  }]
}   
```   


### **2.1.8 获取当前链上账户总个数**   
接口地址：/api/accounts/count   
请求方式：get   
支持格式：无   
请求参数：无    

返回参数说明：   

|名称	|类型   |说明              |   
|------ |-----  |----              |   
|success|boolean  |是否成功获得response数据 |    
|count|integer  |当前链上账户总个数     |    
   
   
请求示例：   
```bash   
curl -k -X GET 'http://47.92.0.84:8001/api/accounts/count'    
```   
   
JSON返回示例：   
```js   
{
	success: true,
	count: 625
}
```    
   
   
## **2.2 交易transactions**   
### **2.2.1 获取交易信息**   
todo
接口地址：/api/transactions   
请求方式：get   
支持格式：urlencoded   
接口备注：如果请求不加参数则会获取全网所有交易   
请求参数说明：   

|名称	|类型   |必填 |说明              |   
|------ |-----  |---  |----              |   
|and|integer|N|取值范围0和1，默认值0。select查询时下面这些条件都是or的关系，and=1时select是and的关系  | 
|blockId |string |N    |区块id      |   
|limit |integer |N    |限制结果集个数，最小值：0,最大值：100   |   
|type|integer  |N      |交易类型,0:普通转账，1:设置二级密码，2:注册受托人，3:投票，4:多重签名，5:DAPP，6:IN_TRANSFER，7:OUT_TRANSFER      |   
|orderBy|string  |N      |根据表中字段排序，senderPublicKey:desc  |   
|offset|integer  |N      |偏移量，最小值0  |   
|senderPublicKey|string|N|发送者公钥|   
|ownerPublicKey|string|N||   
 ownerAddress|string|N||   
|senderId|string|N|发送者地址|   
|recipientId|string|N|接收者地址,最小长度：1|   
|amount|integer|N|金额|   
|fee|integer|N|手续费|   
|aob|integer|N|是否aob，0：不是，1：是|   
|currency|string|N|资产名|   
   
返回参数说明：   

|名称	|类型   |说明              |   
|------ |-----  |----              |   
|success|boolean  |是否成功获得response数据 |    
|transactions|列表  |多个交易详情json构成的列表      |    
|count|int|获取到的交易总个数|   
   
请求示例：   
```bash   
curl -k -X GET 'http://47.92.0.84:8001/api/transactions?recipientId=D2zThPTQZDNQqXbe5tikDQ24YyCQTCpbSC&orderBy=t_timestamp:desc&limit=3'   
```   
   
JSON返回示例：   
```js   
{   
	"success": true,   
	"transactions": [{   
		"id": "17192581936339156329",   
		"height": "105951",   
		"blockId": "15051364118100195665",   
		"type": 0,   
		"timestamp": 4385190,   
		"senderPublicKey": "d39d6f26869067473d685da742339d1a9117257fe14b3cc7261e3f2ed5a339e3",   
		"senderId": "15745540293890213312",   
		"recipientId": "D2zThPTQZDNQqXbe5tikDQ24YyCQTCpbSC",   
		"amount": 10000000000,   
		"fee": 10000000,   
		"signature": "98d65df3109802c707eeed706e90a907f337bddab58cb4c1fbe6ec2179aa1c85ec2903cc0cf44bf0092926829aa5a0a6ec99458f65b6ebd11f0988772e58740e",   
		"signSignature": "",   
		"signatures": null,   
		"confirmations": "31802",   
		"asset": {   
			   
		}   
	},   
	{   
		"id": "7000452951235123088",   
		"height": "105473",   
		"blockId": "11877628176330539727",   
		"type": 0,   
		"timestamp": 4380147,   
		"senderPublicKey": "fafcd01f6b813fdeb3c086e60bc7fa9bfc8ef70ae7be47ce0ac5d06e7b1a8575",   
		"senderId": "16358246403719868041",   
		"recipientId": "D2zThPTQZDNQqXbe5tikDQ24YyCQTCpbSC",   
		"amount": 10000000000,   
		"fee": 10000000,   
		"signature": "dc84044d4f6b4779eecc3a986b6507e458cc5964f601ebeb4d3b68a96129813f4940e14de950526dd685ca1328b6e477e6c57e95aeac45859a2ea62a587d0204",   
		"signSignature": "",   
		"signatures": null,   
		"confirmations": "32280",   
		"asset": {   
			   
		}   
	},   
	{   
		"id": "14093929199102906687",   
		"height": "105460",   
		"blockId": "2237504897174225512",   
		"type": 0,   
		"timestamp": 4380024,   
		"senderPublicKey": "fafcd01f6b813fdeb3c086e60bc7fa9bfc8ef70ae7be47ce0ac5d06e7b1a8575",   
		"senderId": "16358246403719868041",   
		"recipientId": "D2zThPTQZDNQqXbe5tikDQ24YyCQTCpbSC",   
		"amount": 10000000000,   
		"fee": 10000000,   
		"signature": "73ceddc3cbe5103fbdd9eee12f7e4d9a125a3bcf2e7cd04282b7329719735aeb36936762f17d842fb14813fa8f857b8144040e5117dffcfc7e2ae88e36440a0f",   
		"signSignature": "",   
		"signatures": null,   
		"confirmations": "32293",   
		"asset": {   
			   
		}   
	}],   
	"count": 3   
}   
```   
### **2.2.2 根据交易id查看交易详情**   
接口地址：/api/transactions/get   
请求方式：get   
支持格式：urlencoded   
请求参数说明：   

|名称	|类型   |必填 |说明              |   
|------ |-----  |---  |----              |   
|Id |string |Y    |交易id      |   
   
   
返回参数说明：   

|名称	|类型   |说明              |   
|------ |-----  |----              |   
|success|boolean  |是否成功获得response数据 |    
|transactions|json  |交易详情      |    
   
请求示例：   
```bash   
curl -k -X GET 'http://47.92.0.84:8001/api/transactions/get?id=14093929199102906687'   
```   
   
JSON返回示例：   
```js   
{   
	"success": true,   
	"transaction": {   
		"id": "14093929199102906687", // 交易id  
		"height": "105460",// 该交易所在区块高度   
		"blockId": "2237504897174225512",// 所在区块id   
		"type": 0,// 交易类型，0：普通DDN转账   
		"timestamp": 4380024,// 距离阿希创世块的timestamp   
		"senderPublicKey": "fafcd01f6b813fdeb3c086e60bc7fa9bfc8ef70ae7be47ce0ac5d06e7b1a8575", // 发送者公钥   
		"senderId": "16358246403719868041",// 发送者地址   
		"recipientId": "D2zThPTQZDNQqXbe5tikDQ24YyCQTCpbSC",// 接收者地址   
		"amount": 10000000000,// 交易额，100DDN   
		"fee": 10000000, // 手续费0.1DDN  
		"signature": "73ceddc3cbe5103fbdd9eee12f7e4d9a125a3bcf2e7cd04282b7329719735aeb36936762f17d842fb14813fa8f857b8144040e5117dffcfc7e2ae88e36440a0f",   
		"signSignature": "",   
		"signatures": null,   
		"confirmations": "34268",// 确认数   
		"asset": {   
		}   
	}   
}   
```   
   
### **2.2.3 根据未确认交易id查看详情**   
接口地址：/api/transactions/unconfirmed/get   
请求方式：get   
支持格式：urlencoded   
请求参数说明：   

|名称	|类型   |必填 |说明              |   
|------ |-----  |---  |----              |   
|id|string |Y    |未确认交易id      |   
   
返回参数说明：   

|名称	|类型   |说明              |   
|------ |-----  |----              |   
|success|boolean  |是否成功获得response数据 |    
|transaction|json  |未确认交易详情      |   
   
   
请求示例：   
```bash   
curl -k -X GET http://47.92.0.84:8001/api/transactions/unconfirmed/get?id=7557072430673853692  // 正常情况，该未确认交易存在时间极短0~10秒   
```   
   
JSON返回示例：   
```js   
{
	"success": true,
	"transaction": {
		"type": 0,
		"amount": 10000,
		"senderPublicKey": "3ec1c9ec08c0512641deba37c0e95a0fe5fc3bdf58424009f594d7d6a4e28a2a",
		"requesterPublicKey": null,
		"timestamp": 5082322,
		"asset": {
			
		},
		"recipientId": "D2zThPTQZDNQqXbe5tikDQ24YyCQTCpbSC",
		"signature": "3a97f8d63509ef964bda3d816366b8e9e2d9b5d4604a660e7cbeefe210cb910f5de9a51bece06c32d010f55502c62f0f59b8224e1c141731ddfee27206a88d02",
		"id": "7557072430673853692",
		"fee": 10000000,
		"senderId": "15238461869262180695"
	}
}
```   
   
   
### **2.2.4 获取[全网所有]未确认的交易详情**   
接口地址：/api/transactions/unconfirmed   
请求方式：get   
支持格式：urlencoded   
接口说明：如果不加参数，则会获取全网所有未确认交易
请求参数说明：   

|名称	|类型   |必填 |说明              |   
|------ |-----  |---  |----              |   
|senderPublicKey |string |N    |发送者公钥      |   
|address |string |N    |地址      |   
   
   
返回参数说明：   

|名称	|类型   |说明              |   
|------ |-----  |----              |   
|success|boolean  |是否成功获得response数据 |    
|transactions|Array  |未确认交易列表      |    
   
   
请求示例：   
```bash   
curl -k -X GET 'http://47.92.0.84:8001/api/transactions/unconfirmed'   
```   
   
JSON返回示例：   
```js   
{   
	"success": true,   
	"transactions": []      //全网目前不存在未确认的交易   
}   
```   
   
### **2.2.5 创建交易并广播**   
接口地址：/api/transactions   
请求方式：PUT   
支持格式：json   
接口备注：接收者账户需在web端钱包登陆过   
请求参数说明：   

|名称	|类型   |必填 |说明              |   
|------ |-----  |---  |----              |   
|secret |string |Y    |DDN账户密码       |   
|amount|integer|Y|金额，最小值：1，最大值：10000000000000000|   
|recipientId|string|Y|接收者地址,最小长度：1|   
|publicKey|string|N|发送者公钥|   
|secondSecret|string|N|发送者二级密码，最小长度1，最大长度：100|   
|multisigAccountPublicKey|string|N|多重签名账户公钥|   
   
返回参数说明：   

|名称	|类型   |说明              |   
|------ |-----  |----              |   
|success|boolean  |是否成功获得response数据 |    
|transactionId|string  |交易id      |    
   
   
请求示例：   
```bash   
curl -k -H "Content-Type: application/json" -X PUT -d '{"secret":"unaware label emerge fancy concert long fiction report affair appear decide twenty","amount":1000000,"recipientId":"D2zThPTQZDNQqXbe5tikDQ24YyCQTCpbSC"}' 'http://47.92.0.84:8001/api/transactions'    
```   
   
JSON返回示例：   
```js   
{   
	"success": true,   
	"transactionId": "16670272591943275531"   
}   
```  
   
## **2.3 区块blocks**   
### **2.3.1 获取指定区块的详情**   
接口地址：/api/blocks/get   
请求方式：get   
支持格式：urlencoded   
请求参数说明：   

|名称	|类型   |必填 |说明              |   
|------ |-----  |---  |----              |   
|id |string |参数3选1    |区块id      |   
|height|string|参数3选1|区块高度|   
|hash|string|参数3选1|区块hash|   
   
返回参数说明：   

|名称	|类型   |说明              |   
|------ |-----  |----              |   
|success|boolean  |是否成功获得response数据 |    
|block|json  |区块详情      |    
   
   
请求示例：   
```bash   
curl -k -X GET 'http://47.92.0.84:8001/api/blocks/get?id=6076474715648888747'   
```   
   
JSON返回示例：   
```js   
{   
	"success": true,   
	"block": {   
		"id": "6076474715648888747",   
		"version": 0,   
		"timestamp": 4734070,   
		"height": 140538,   
		"previousBlock": "16033230167082515105",    //上一个区块id   
		"numberOfTransactions": 0,  //交易数   
		"totalAmount": 0,   //交易额   
		"totalFee": 0,   
		"reward": 500000000,    //奖励   
		"payloadLength": 0,   
		"payloadHash": "e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855",   
		"generatorPublicKey": "1d352950c8141e1b35daba4a974a604519d7a2ef3a1ec0a503ce2653646aa052",   
		"generatorId": "6656029629904254066",   
		"blockSignature": "a53de66922cdc2f431acd0a474beec7cf7c420a8460b7b7caf84999be7caebb59fb7fbb7166c2c7013dbb431585ea7294722166cb08bf9663abf50b6bd81cd05",   
		"confirmations": "2",   
		"totalForged": 500000000   
	}   
}   
```   
   
### **2.3.2 获取区块数据**   
接口地址：/api/blocks   
请求方式：get   
支持格式：urlencoded   
接口说明：不加参数则获取全网区块详情   
请求参数说明：   

|名称	|类型   |必填 |说明              |   
|------ |-----  |---  |----              |   
|limit |integer |N    |限制结果集个数，最小值：0,最大值：100   |   
|orderBy|string  |N      |根据表中字段排序，如height:desc  |   
|offset|integer  |N      |偏移量，最小值0  |   
|generatorPublicKey|string  |N      |区块生成者公钥  |   
|totalAmount|integer  |N       |交易总额，最小值：0，最大值：10000000000000000 |   
|totalFee|integer  |N      |手续费总额，最小值：0，最大值：10000000000000000  |   
|reward|integer  |N      |奖励金额，最小值：0  |   
|previousBlock|string  |N      |上一个区块  |   
|height|integer  |N      |区块高度  |   
   
返回参数说明：   

|名称	|类型   |说明              |   
|------ |-----  |----              |   
|success|boolean  |是否成功获得response数据 |    
|blocks|Array  |由区块详情json串构成的数组 |    
|count|integer|区块链高度|   
   
   
请求示例：   
```bash   
curl -k -X GET 'http://47.92.0.84:8001/api/blocks?limit=2&offset=0&orderBy=height:desc'   
```   
   
JSON返回示例：   
```js   
{   
	"success": true,   
	"blocks": [{   
		"id": "12634047624004615059",   
		"version": 0,   
		"timestamp": 4708080,   
		"height": 137986,   
		"previousBlock": "3498191422350401106",   
		"numberOfTransactions": 0,  // 交易数   
		"totalAmount": 0,   // 金额   
		"totalFee": 0,  // 手续费   
		"reward": 500000000,    // 奖励   
		"payloadLength": 0,   
		"payloadHash": "e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855",   
		"generatorPublicKey": "44db7bec89ef289d0def257285675ca14f2a947dfd2b70e6b1cff4392ce42ada",   
		"generatorId": "4925169939071346193",   
		"blockSignature": "83a2124e3e8201c1a6099b2ac8ab1c117ad34867978add3a90d41a64df9d2ad8fabc9ec14d27a77cd34c08a6479ef684f247c11b1cbbcb0e9767dffc85838600",   
		"confirmations": "1",   
		"totalForged": 500000000   
	},   
	{   
		"id": "3498191422350401106",   
		"version": 0,   
		"timestamp": 4708070,   
		"height": 137985,   
		"previousBlock": "14078155423801039323",   
		"numberOfTransactions": 0,   
		"totalAmount": 0,   
		"totalFee": 0,   
		"reward": 500000000,   
		"payloadLength": 0,   
		"payloadHash": "e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855",   
		"generatorPublicKey": "500b1ec025cd64d36008341ed8d2508473ecf559be213ca5f9580620a21a592c",   
		"generatorId": "16006295608945777169",   
		"blockSignature": "a0b5ed6c94b1f33c4d0f017f21a08357061493392b19e34eeedf274b77c751e3f86c92443280de09ea1754d62fe7ef00e02acbdc3bc0c1063cef344bacaa4f07",   
		"confirmations": "2",   
		"totalForged": 500000000   
	}],   
	"count": 137986   
}   
```   
   
### **2.3.3 获取区块链高度**   
接口地址：/api/blocks/getHeight   
请求方式：get   
支持格式：无   
请求参数说明：无   
   
返回参数说明：   

|名称	|类型   |说明              |   
|------ |-----  |----              |   
|success|boolean  |是否成功获得response数据 |    
|height|integer  |区块链高度      |    
   
请求示例：   
```bash   
curl -k -X GET 'http://47.92.0.84:8001/api/blocks/getheight'    
```   
   
JSON返回示例：   
```js   
{"success":true,"height":317044}   
```   
   
### **2.3.4 获取普通转账手续费**   
接口地址：/api/blocks/getFee   
请求方式：get   
支持格式：无   
请求参数说明：无   
   
返回参数说明：   

|名称	|类型   |说明              |   
|------ |-----  |----              |   
|success|boolean  |是否成功获得response数据 |    
|fee|integer  |交易手续费      |    
   
   
请求示例：   
```bash   
curl -k -X GET 'http://47.92.0.84:8001/api/blocks/getfee'   
```   
   
JSON返回示例：   
```js   
{"success":true,"fee":10000000}     //手续费为0.1 DDN   
```   
   
### **2.5 获取里程碑**   
接口地址：/api/blocks/getMilestone   
请求方式：get   
支持格式：无   
请求参数说明：无   
返回参数说明：   

|名称	|类型   |说明              |   
|------ |-----  |----              |   
|success|boolean  |是否成功获得response数据 |    
|milestone|integer  |      |    
   
   
请求示例：   
```bash   
curl -k -X GET 'http://47.92.0.84:8001/api/blocks/getMilestone'    
```   
   
JSON返回示例：   
```js   
{"success":true,"milestone":0}   
```   
   
### **2.3.6 查看单个区块奖励**   
接口地址：/api/blocks/getReward   
请求方式：get   
支持格式：无   
请求参数说明：无   
   
返回参数说明：   

|名称	|类型   |说明              |   
|------ |-----  |----              |   
|success|boolean  |是否成功获得response数据 |    
|reward|integer  |区块奖励，包含受托人奖励和手续费      |    
   
   
请求示例：   
```bash   
curl -k -X GET 'http://47.92.0.84:8001/api/blocks/getReward'   
```   
   
JSON返回示例：   
```js   
{"success":true,"reward":500000000} //每个生成一个block奖励5 DDN   
```   
   
### **2.3.7 获取COIN当前供应值**   
接口地址：/api/blocks/getSupply   
请求方式：get   
支持格式：无   
请求参数说明：无   
   
返回参数说明：   

|名称	|类型   |说明              |   
|------ |-----  |----              |   
|success|boolean  |是否成功获得response数据 |    
|supply|integer  |全网DDN个数      |    
   
   
请求示例：   
```bash   
curl -k -X GET 'http://47.92.0.84:8001/api/blocks/getSupply'   
```   
   
JSON返回示例：   
```js   
{"success":true,"supply":10158519000000000} //当前testnet共有101585190DDN   
```   
   
### **2.3.8 区块链状态**   
接口地址：/api/blocks/getStatus   
请求方式：get   
支持格式：无   
请求参数说明：无   
   
返回参数说明：   

|名称	|类型   |说明              |   
|------ |-----  |----              |   
|success|boolean  |是否成功获得response数据 |    
|height|integer  |区块链高度      |    
|fee|integer  |交易手续费      |    
|milestone|integer  |      |    
|reward|integer  |区块奖励      |    
|supply|integer  |全网DDN个数      |    
   
   
请求示例：   
```bash   
curl -k -X GET 'http://47.92.0.84:8001/api/blocks/getStatus'   
```   
   
JSON返回示例：   
```js   
{   
	"success": true,   
	"height": 317044,   
	"fee": 10000000,   
	"milestone": 0,   
	"reward": 500000000,   
	"supply": 10158519000000000   
}   
```   
   

### **2.3.9 获取指定区块的交易信息**   
接口地址：/api/blocks/full   
请求方式：get   
支持格式：无   
请求参数说明：无  
 
   
请求参数说明：

|名称	|类型   |必填    |说明              |
|------ |-----  |----   |----           |
|id |string |参数2选1    |区块id      |
|height|string|参数2选1|区块高度|

返回参数说明：   

|名称	|类型   |说明              |   
|------ |-----  |----              |   
|success|boolean  |是否成功获得response数据 |    
|block|json  |区块详情，包含交易信息transactions      |   
   
请求示例：   
```bash   
curl -k -X GET 'http://47.92.0.84:8001/api/blocks/full?height=6666'   
```   
   
JSON返回示例：   
```js   
{
	"success": true,
	"block": {
		"id": "846e29249dff5d62bb577dd7f8f9043ac473e71fc1281c6f0ca0248cd46d35df",
		"version": 0,
		"timestamp": 91140,
		"height": 6666,
		"previousBlock": "6c1027ce7b177e9ed4f9d36695789babba4526ee04dfbf894dd7616e2a9d3b81",
		"numberOfTransactions": 0,
		"totalAmount": 0,
		"totalFee": 0,
		"reward": 500000000,
		"payloadLength": 0,
		"payloadHash": "e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855",
		"generatorPublicKey": "4649a8d705d32c0ebef1ad799ce7757387a6357de930a2b75f1fdd441d901567",
		"generatorId": "DLzgMhdRTDn6bEYtzxi8JM86vAv1HkEvD4",
		"blockSignature": "241a94e226d6d32e44ca641e22996e545c5ffba18eb54963ca17eb166b3901c22b9ad437766a58753454e07ef0e716c7410d20483457cff2048f76ea7354bb08",
		"totalForged": 500000000,
		"transactions": []
	}
}
``` 
   
   
## **2.4 受托人delegates**   
   
### **2.4.1 获取受托人总个数**   
接口地址：/api/delegates/count   
请求方式：get   
支持格式：无   
请求参数说明：无   
   
返回参数说明：   
    
|名称	|类型   |说明              |   
|------ |-----  |----              |   
|success|boolean  |是否成功获得response数据 |    
|count|integer   |受托人总个数      |    
   
请求示例：   
```bash   
curl -k -X GET 'http://47.92.0.84:8001/api/delegates/count'   
```   
   
JSON返回示例：   
```js   
{    
	"success": true,    
	"count": 101    
}     
```   
   
### **2.4.2 根据受托人公钥查看哪些人为其投了票**   
接口地址：/api/delegates/voters   
请求方式：get   
支持格式：urlencoded   
请求参数说明：   

|名称	|类型   |必填 |说明              |   
|------ |-----  |---  |----              |   
|publicKey |string |Y    |受托人公钥      |   
   
返回参数说明：   

|名称	|类型   |说明              |   
|------ |-----  |----              |   
|success|boolean  |是否成功获得response数据 |    
|accounts|Array  |账户json串组成的数组      |    
   
请求示例：   
```bash   
curl -k -X GET 'http://47.92.0.84:8001/api/delegates/voters?publicKey=0095b28997a33bb4f16b62523bcc1902179f2a7b5a3dd83980da5c1cbae854d6'   
```   
   
JSON返回示例：   
```js   
{
    "success": true,
    "accounts": [{
        "username": "",
        "address": "DPesT3fMLkDVHvqLaXR3YzGkdw16vgKqPj",
        "publicKey": "70d0cbd2c3dccfaaa65acfa5689dbc3656ff0807cd1611e88d854fec07845ac1",
        "balance": 9977192780000000,
        "weight": 98.21503291966083
    }]
}  
```   
   
### **2.4.3 根据公钥或者用户名获取受托人详情**   
接口地址： /api/delegates/get   
请求方式：get   
支持格式：urlencoded   
接口备注：通过公钥或者用户名获取受托人信息   
请求参数说明：   

|名称	|类型   |必填 |说明              |   
|------ |-----  |---  |----              |   
|publickey |string |二选一    |受托人公钥      |   
|username  |string |二选一    |受托人用户名      |   
   
返回参数说明：   

|名称	|类型   |说明              |   
|------ |-----  |----              |   
|success|boolean  |是否成功获得response数据 |    
|delegate|json  |受托人详情      |    
   
   
请求示例：   
```bash   
curl -k -X GET http://47.92.0.84:8001/api/delegates/get?publicKey=0095b28997a33bb4f16b62523bcc1902179f2a7b5a3dd83980da5c1cbae854d6   
curl -k -X GET http://47.92.0.84:8001/api/delegates/get?username=ddn_88   
```   
   
JSON返回示例：   
```js   
{
    "success": true,
    "delegate": {
        "username": "ddn_88",
        "address": "D792yYKHLNSpsEoRT5c3whgmCq2VYpam55",
        "publicKey": "0095b28997a33bb4f16b62523bcc1902179f2a7b5a3dd83980da5c1cbae854d6",
        "balance": 614801267317,
        "vote": 9977192780000000,
        "producedblocks": 1537,
        "missedblocks": 1604,
        "fees": 1267317,
        "rewards": 614800000000,
        "rate": 1,
        "approval": 98.22,
        "productivity": 48.93,
        "forged": "614801267317"
    }
} 
```   
   
### **2.4.4 获取受托人列表**   
接口地址：/api/delegates   
请求方式：get   
支持格式：urlencoded   
接口说明：如果不加参数则会返回全网受托人列表   
请求参数说明：   

|名称	|类型   |必填 |说明              |   
|------ |-----  |---  |----              |   
|address |string |N    |受托人地址      |   
|limit|int  |N       |限制返回结果数据集的个数       |   
|offset|integer  |N       |偏移量，最小值：0      |   
|orderBy|string  |N       |排序字段:排序规则，如:desc      |   
   
   
   
返回参数说明：   

|名称	|类型   |说明              |   
|------ |-----  |----              |   
|success|boolean  |是否成功获得response数据 |    
|delegates|Array  |受托人详情列表      |    
   
   
请求示例：   
```bash   
curl -k -X GET 'http://47.92.0.84:8001/api/delegates?orderby=approval:desc&limit=2' //按照得票率降序排序，取出前2名   
```   
   
JSON返回示例：   
```js   
{
    "success": true,
    "delegates": [{
        "username": "ddn_88",
        "address": "D792yYKHLNSpsEoRT5c3whgmCq2VYpam55",
        "publicKey": "0095b28997a33bb4f16b62523bcc1902179f2a7b5a3dd83980da5c1cbae854d6",
        "balance": 614801267317,
        "vote": 9977192780000000,
        "producedblocks": 1537,
        "missedblocks": 1604,
        "fees": 1267317,
        "rewards": 614800000000,
        "rate": 1,
        "approval": 98.22,
        "productivity": 48.93,
        "forged": "614801267317"
    }, 
    
    {
        "username": "ddn_65",
        "address": "D2zThPTQZDNQqXbe5tikDQ24YyCQTCpbSC",
        "publicKey": "03afab2fff8e10592604db708c57b9d89b96cc5a14a80c805702b8ade8ae6f3b",
        "balance": 1573761742557,
        "vote": 9977192780000000,
        "producedblocks": 3934,
        "missedblocks": 1,
        "fees": 161742557,
        "rewards": 1573600000000,
        "rate": 2,
        "approval": 98.22,
        "productivity": 99.97,
        "forged": "1573761742557"
    }],
    "totalCount": 101
} 
```   
   
   
### **2.4.5 获取受托人设置的转账费**   
接口地址：/api/delegates/fee   
请求方式：get   
支持格式：urlencoded   
请求参数说明：   

|名称	|类型   |必填 |说明              |   
|------ |-----  |---  |----              |   
|publicKey |string |Y    |受托人公钥      |   
   
返回参数说明：   

|名称	|类型   |说明              |   
|------ |-----  |----              |   
|success|boolean  |是否成功获得response数据 |    
|fee|integer  |转账费      |    
   
   
请求示例：   
```bash   
curl -k -X GET 'http://47.92.0.84:8001/api/delegates/fee?publicKey=0095b28997a33bb4f16b62523bcc1902179f2a7b5a3dd83980da5c1cbae854d6'   
```   
   
JSON返回示例：   
```js   
{"success":true,"fee":10000000000}  //0.1 DDN   
```   
   
### **2.4.6 根据公钥查看其锻造情况**   
接口地址：/api/delegates/forging/getForgedByAccount   
请求方式：get   
支持格式：urlencoded   
请求参数说明：   

|名称	|类型   |必填 |说明              |   
|------ |-----  |---  |----              |   
|generatorPublicKey |string |Y    |区块生成者公钥      |   
   
返回参数说明：   

|名称	|类型   |说明              |   
|------ |-----  |----              |   
|success|boolean  |是否成功获得response数据 |    
|fees|integer  |收取的手续费      |    
|rewards|integer|已获得奖励|   
|forged|integer|锻造获得的总奖励|   
   
   
请求示例：   
```bash   
curl -k -X GET 'http://47.92.0.84:8001/api/delegates/forging/getForgedByAccount?generatorPublicKey=0095b28997a33bb4f16b62523bcc1902179f2a7b5a3dd83980da5c1cbae854d6'   
```   
   
JSON返回示例：   
```js   
{   
	"success": true,   
	"fees": 12589307065,   
	"rewards": 285600000000,   
	"forged": 298189307065   
}   
```   
   
### **2.4.7 注册受托人**   
接口地址：/api/delegates   
请求方式：put   
支持格式：urlencoded   
请求参数说明：   

|名称	|类型   |必填 |说明              |   
|------ |-----  |---  |----              |   
|secret |string |Y    |DDN账户密码       |   
|publicKey|string  |N      |公钥|    
|secondSecret|string|N|DDN账户二级密码，最小长度：1，最大长度：100|   
|username|string|N|受托人名字|   
   
返回参数说明：   

|名称	|类型   |说明              |   
|------ |-----  |----              |   
|success|boolean  |是否成功获得response数据 |    
|transaction|json  |注册受托人交易详情      |    
   
   
请求示例：   
```bash   
curl -k -H "Content-Type: application/json" -X PUT -d '{"secret":"unaware label emerge fancy concert long fiction report affair appear decide twenty","username":"delegate_0821"}' 'http://47.92.0.84:8001/api/delegates'   
```   
   
JSON返回示例：   
```js   
{   
	"success": true,   
	"transaction": {   
		"type": 2,  //注册受托人的交易类型为2   
		"amount": 0,   
		"senderPublicKey": "3b64f1833e6328043e1f2fee31e638bdaa6dfff5c7eb9c8577a5cefcf11261f2",   
		"requesterPublicKey": null,   
		"timestamp": 4737615,   
		"asset": {   
			"delegate": {   
				"username": "delegate_0821",   
				"publicKey": "3b64f1833e6328043e1f2fee31e638bdaa6dfff5c7eb9c8577a5cefcf11261f2"   
			}   
		},   
		"recipientId": null,   
		"signature": "7f8417e8db5f58ddff887c86c789c26b32fd3f01083ef1e3c8d4e18ed16622bf766492d78518c6c7a07aada1c98b1efc36d40c8e09394989dbde229d8e3f8103",   
		"id": "16351320834453011577",   
		"fee": 10000000000,   
		"senderId": "250438937633388106"   
	}   
}   
```   
   
### **2.4.8 受托人开启锻造**   
接口地址：/api/delegates/forging/enable   
请求方式：post   
支持格式：urlencoded   // url必须是受托人所在服务器  
请求参数说明：   

|名称	|类型   |必填 |说明              |   
|------ |-----  |---  |----              |   
|secret |string |Y    |DDN账户密码       |   
|publicKey|string  |N      |公钥|    

   
返回参数说明：   

|名称	|类型   |说明              |   
|------ |-----  |----              |   
|success|boolean  |是否成功获得response数据 |    
|address|string  |受托人地址      |    
   
   
请求示例：   
```bash   
curl -k -H "Content-Type: application/json" -X POST -d '{"secret":"motion group blossom coral upper warrior pattern fragile sister misery palm detect"}' 'http://localhost:8001/api/delegates/forging/enable'   
```   
   
JSON返回示例：   
```js   
{"success":true,"address":"16358246403719868041"}   
```      

### **2.4.9 受托人关闭锻造**   
接口地址：/api/delegates/forging/disable   
请求方式：post   
支持格式：urlencoded   // url必须是受托人所在服务器  
请求参数说明：   

|名称	|类型   |必填 |说明              |   
|------ |-----  |---  |----              |   
|secret |string |Y    |DDN账户密码       |   
|publicKey|string  |N      |公钥|    

   
返回参数说明：   

|名称	|类型   |说明              |   
|------ |-----  |----              |   
|success|boolean  |是否成功获得response数据 |    
|address|string  |受托人地址      |    
   
   
请求示例：   
```bash   
curl -k -H "Content-Type: application/json" -X POST -d '{"secret":"motion group blossom coral upper warrior pattern fragile sister misery palm detect"}' 'http://localhost:8001/api/delegates/forging/disable'   
```   
   
JSON返回示例：   
```js   
{"success":true,"address":"16358246403719868041"}     
```     
### **2.4.10 受托人锻造状态查看**   
接口地址：/api/delegates/forging/status      
请求方式：get     
支持格式：urlencoded    
请求参数说明：   

|名称	|类型   |必填 |说明              |   
|------ |-----  |---  |----              |   
|publicKey|string  |Y      |公钥|    

   
返回参数说明：   

|名称	|类型   |说明              |   
|------ |-----  |----              |   
|success|boolean  |是否成功获得response数据 |    
|enabled|string  |锻造是否开启      |    
   
   
请求示例：   
```bash   
curl -k -X GET 'http://47.92.0.84:8001/api/delegates/forging/status?publicKey=fafcd01f6b813fdeb3c086e60bc7fa9bfc8ef70ae7be47ce0ac5d06e7b1a8575'        
```   
   
JSON返回示例：   
```js   
{"success":true,"enabled":false}    
```   

### **2.4.11 给受托人投票 & 取消投票**   
接口地址：/api/accounts/delegates   
请求方式：put   
支持格式：json   
请求参数说明：   

|名称	|类型   |必填 |说明              |   
|------ |-----  |---  |----              |   
|secret |string |Y    |DDN账户密码       |   
|publicKey|string  |N|公钥      |    
|secondSecret|string|N|DDN账户二级密码，最小长度：1，最大长度：100|   
|delegates|Array|受托人公钥数组，每个公钥前需要加上+或者-号，代表增加/取消对其的投票|   
   
返回参数说明：   

|名称	|类型   |说明              |   
|------ |-----  |----              |   
|success|boolean  |是否成功获得response数据 |    
|transaction|json  |投票交易详情      |    
   
   
请求示例：   
```bash   
curl -k -H "Content-Type: application/json" -X PUT -d '{"secret":"call scissors pupil water friend timber spend brand vote obey corn size","publicKey":"3ec1c9ec08c0512641deba37c0e95a0fe5fc3bdf58424009f594d7d6a4e28a2a","delegates":["+fafcd01f6b813fdeb3c086e60bc7fa9bfc8ef70ae7be47ce0ac5d06e7b1a8575"]}' 'http://47.92.0.84:8001/api/accounts/delegates'     
```   
   
JSON返回示例：   
```js   
 {
	"success": true,
	"transaction": {
		"type": 3,  //投票的交易类型为3
		"amount": 0,
		"senderPublicKey": "3ec1c9ec08c0512641deba37c0e95a0fe5fc3bdf58424009f594d7d6a4e28a2a",
		"requesterPublicKey": null,
		"timestamp": 5056064,
		"asset": {
			"vote": {
				"votes": ["+fafcd01f6b813fdeb3c086e60bc7fa9bfc8ef70ae7be47ce0ac5d06e7b1a8575"]
			}
		},
		"recipientId": null,
		"signature": "0bff58c7311fc59b3c8b3ffc236bbfece9850c334fb0c292ab087f78cf9a6c0f4d3e541c501887a2c2ec46294c777e8f7bf7dea9cb7c9a175fdec641bb684f08",
		"id": "5630629337798595849",
		"fee": 10000000,
		"senderId": "15238461869262180695"
	}
}  
```   
  
### **2.4.12 根据地址获取其投票列表**   
接口地址：/api/accounts/delegates   
请求方式：get   
支持格式：urlencoded   
请求参数说明：   

|名称	|类型   |必填 |说明              |   
|------ |-----  |---  |----              |   
|address |string |Y    |投票人地址      |   
   
返回参数说明：   

|名称	|类型   |说明              |   
|------ |-----  |----              |   
|success|boolean  |是否成功获得response数据 |    
|delegates|Array  |已投票的受托人详情数组      |    
   
   
请求示例：   
```bash   
curl -k -X GET 'http://47.92.0.84:8001/api/accounts/delegates?address=D792yYKHLNSpsEoRT5c3whgmCq2VYpam55'   
```   
   
JSON返回示例：   
```js   
{   
	"success": true,   
	"delegates": []   
}   
```   
   
### **2.4.13 获取受托人手续费设置**   
接口地址：/api/accounts/delegates/fee   
请求方式：get   
支持格式：无   
请求参数说明：无  

返回参数说明：   

|名称	|类型   |说明              |   
|------ |-----  |----              |   
|success|boolean  |是否成功获得response数据 |    
|fee|integer  |手续费      |    
   
   
请求示例：   
```bash   
curl -k -X GET 'http://47.92.0.84:8001/api/accounts/delegates/fee  
```   
   
JSON返回示例：   
```js   
{   
	"success": true,   
	"fee": 100000000  // 0.1 DDN   
}   
```   


## **2.5 节点peers**   
   
### **2.5.1 获取本机连接的所有节点信息**   
接口地址：/api/peers   
请求方式：get   
支持格式：urlencoded   
备注：展示节点只是和本机有连接的节点，并不是全网所有的节点    
请求参数说明：   

|名称	|类型   |必填 |说明              |   
|------ |-----  |---  |----              |   
|state |integer |N    |节点状态,0: ,1:,2:,3:     |   
|os|string|N|内核版本|   
|version|string|N|DDN版本号|   
|limit |integer |N    |限制结果集个数，最小值：0,最大值：100   |   
|orderBy|string|N||   
|offset|integer  |N      |偏移量，最小值0  |   
|port|integer|N|端口，1~65535|   
   
   
返回参数说明：   

|名称	|类型   |说明              |   
|------ |-----  |----              |   
|success|boolean  |是否成功获得response数据 |    
|peers|Array  |节点信息json构成的数组     |    
|totalCount|integer|当前正在运行的节点个数|   
   
   
请求示例：   
```bash   
curl -k -X GET 'http://47.92.0.84:8001/api/peers?limit=1'   
```   
   
JSON返回示例：   
```js   
{   
	"success": true,   
	"peers": [{   
		"ip": "45.32.19.241",   
		"port": 8001,   
		"state": 2,   
		"os": "linux3.13.0-87-generic",   
		"version": "1.0.0"   
	}],   
	"totalCount": ["54"]   
}   
```   
   
### **2.5.2 获取本节点版本号等信息**   
接口地址：/api/peers/version   
请求方式：get   
支持格式：无   
请求参数说明：无参数   
   
返回参数说明：   

|名称	|类型   |说明              |   
|------ |-----  |----              |   
|success|boolean  |是否成功获得response数据 |    
|version|string  |版本号      |    
|build  |timestamp |构建时间     |    
|net    |string  |主链或者测试链     |   
   
   
请求示例：   
```bash   
curl -k -X GET http://47.92.0.84:8001/api/peers/version   
```   
   
JSON返回示例：   
```js   
{   
	"success": true,   
	"version": "1.0.0",   
	"build": "17:27:19 09/12/2016",   
	"net": "testnet"   
}   
```   
   
### **2.5.3 获取指定ip节点信息**   
接口地址：/api/peers/get   
请求方式：get   
支持格式：urlencoded   
请求参数说明：   

|名称	|类型   |必填 |说明              |   
|------ |-----  |---  |----              |   
|ip |string |Y    |待查询节点ip      |   
|port|integer|Y|待查询节点端口，1~65535|   
   
   
返回参数说明：   

|名称	|类型   |说明              |   
|------ |-----  |----              |   
|success|boolean  |是否成功获得response数据 |    
|peer|json  |      |    
   
   
请求示例：   
```bash   
curl -k -X GET 'http://47.92.0.84:8001/api/peers/get?ip=47.92.0.84&port=8001'   
```   
   
JSON返回示例：   
```js   
{   
	"success": true,   
	"peer": {   
	}   
}   
```   
   
   
## **2.6 同步和加载**   
### **2.6.1 查看本地区块链加载状态**   
接口地址：/api/loader/status   
请求方式：get   
支持格式：无   
请求参数说明：无参数   
   
返回参数说明：   

|名称	|类型   |说明              |   
|------ |-----  |----              |   
|success|boolean  |是否成功获得response数据 |    
|loaded |boolean    |          |   
|blocksCount|integer||   
   
请求示例：   
```bash   
curl -k http://47.92.0.84:8001/api/loader/status -X GET   
```   
   
JSON返回示例：   
```js   
{   
	"success": true,   
	"loaded": true,   
	"blocksCount": 0   
}   
```   
   
### **2.6.2 查看区块同步信息**   
接口地址：/api/loader/status/sync   
请求方式：get   
支持格式：无   
请求参数说明：无参数   
   
返回参数说明：   

|名称	|类型   |说明              |   
|------ |-----  |----              |   
|success|boolean  |是否成功获得response数据 |    
|height |int    |区块高度          |   
   
请求示例：   
```bash   
curl -k http://47.92.0.84:8001/api/loader/status/sync -X GET   
```   
   
JSON返回示例：   
```js   
{   
	"success": true,   
	"syncing": false,  // 是否在同步中，如果是则为true，目前没有数据可以同步所以为false   
	"blocks": 0,   
	"height": 111987   
}   
```   
   
## **2.7 二级密码signatures**   
### **2.7.1 设置二级密码**   
接口地址：/api/signatures   
请求方式：put   
支持格式：json   
请求参数说明：   

|名称	|类型   |必填 |说明              |   
|------ |-----  |---  |----              |   
|secret |string |Y    |DDN账户密码       |   
|publicKey|string  |N|公钥      |    
|secondSecret|string|Y|DDN账户二级密码，最小长度：1，最大长度：100|   
|multisigAccountPublicKey|string|N|多重签名账户公钥|   
   
   
返回参数说明：   

|名称	|类型   |说明              |   
|------ |-----  |----              |   
|success|boolean  |是否成功获得response数据 |    
|transaction|json  |设置二级密码产生的交易详情      |    
   
   
请求示例：   
```bash   
curl -k -H "Content-Type: application/json" -X PUT -d '{"secret":"unaware label emerge fancy concert long fiction report affair appear decide twenty","secondSecret":"fault still attack alley expand music basket purse later educate follow ride"}' 'http://47.92.0.84:8001/api/signatures'    
```   
   
JSON返回示例：   
```js   
{   
	"success": true,   
	"transaction": {   
		"type": 1,  //设置二级密码密码的交易类型为1   
		"amount": 0,   
		"senderPublicKey": "3b64f1833e6328043e1f2fee31e638bdaa6dfff5c7eb9c8577a5cefcf11261f2",   
		"requesterPublicKey": null,   
		"timestamp": 4872315,   
		"asset": {   
			"signature": {   
				"publicKey": "bd1e78c5a10fbf1eca36b28bbb8ea85f320967659cbf1f7ff1603d0a368867b9"   
			}   
		},   
		"recipientId": null,   
		"signature": "e76d9b25ec0fdaa88b19d59c5a222b7efdc04f738ee05896f55f4e6959229d9b1600ca25aa92fbea176668f3be7c12c506f2091e2b38c52ef0ece7a5d35e240a",   
		"id": "1614688380530105232",   
		"fee": 500000000,       //设置二级密码密码的手续费为5 DDN   
		"senderId": "250438937633388106"   
	}   
}   
```   
   
### **2.7.2 获取二级密码设置手续费**   
接口地址：/api/signatures/fee   
请求方式：get   
支持格式：无   
请求参数说明：无   
   
   
返回参数说明：   

|名称	|类型   |说明              |   
|------ |-----  |----              |   
|success|boolean  |是否成功获得response数据 |    
|fee|integer  |费用      |    
   
   
请求示例：   
```bash   
curl -k http://47.92.0.84:8001/api/signatures/fee -X GET   
```   
   
JSON返回示例：   
```js   
{   
	"success": true,   
	"fee": 500000000         //5 DDN   
}     
```   
   
   
## **2.8 多重签名multisignatures**   
### **2.8.1 设置普通账户为多重签名账户**   
接口地址：/api/multisignatures   
请求方式：put   
支持格式：json   
接口说明：返回结果只是生成交易id，还需要其他人签名后该账户才能成功设置成多重签名账户。注册多重签名账户后任意一笔转账都需要多人签名，签名最少个数为min的值（含交易发起人自身）   
请求参数说明：   

|名称	|类型   |必填 |说明              |   
|------ |-----  |---  |----              |   
|secret |string |Y    |DDN账户密码       |   
|publicKey|string  |N|公钥      |    
|secondSecret|string|N|DDN账户二级密码，最小长度：1，最大长度：100|   
|min|integer|Y|多重签名交易账户的任意一笔转账都需要多人签名的最少个数，如果是注册多重签名账户操作，这该值不生效（此时需要所有人都签名）。最小值：2，最大值：16,该值需要小于keysgroup.length+1|   
|lifetime|integer|Y|多重签名交易的最大挂起时间，最小值：1，最大值：24，暂时不生效|   
|keysgroup|array|Y|其它签名人的公钥数组，每个公钥前需要加上+或者-号，代表增加/删除多重签名账户，数组最小长度：1，数组最大长度：10|   
   
   
   
返回参数说明：   

|名称	|类型   |说明              |   
|------ |-----  |----              |   
|success|boolean  |是否成功获得response数据 |    
|transactionId|string  |多重签名交易的id      |    
   
   
请求示例：   
```bash   
curl -k -H "Content-Type: application/json" -X PUT -d '{"secret":"vanish deliver message evil canyon night extend unusual tell prosper issue antenna","min":2,"lifetime":1,"keysgroup":["+eb48b9ab7c9a34a9b7cdf860265d65b31af774355cabf1b3a387d14a1925dc97","+d5d7aa157f866c47a2a1e09e2746286ed089fd90976b54fbfa930e87d11609cb"]}' 'http://47.92.0.84:8001/api/multisignatures'  //公钥为2cef5711e61bb5361c544077aa08aebc4d962a1d656571901c48d716382ad4fd   
```   
   
JSON返回示例：   
```js   
{   
	"success": true,   
	"transactionId": "17620378998277022323"     //返回结果只是生成交易id，还需要其他人签名后该账户才能成功设置成多重签名账户   
}   
```   
   
### **2.8.2 根据公钥获取挂起的多重签名交易详情**   
接口地址：/api/multisignatures/pending   
请求方式：get   
支持格式：urlencoded   
请求参数说明：   

|名称	|类型   |必填 |说明              |   
|------ |-----  |---  |----              |   
|publicKey|string  |Y|公钥      |    
   
   
返回参数说明：   

|名称	|类型   |说明              |   
|------ |-----  |----              |   
|success|boolean  |是否成功获得response数据 |    
|transactions|Array  |交易json组成的数组      |    
   
   
请求示例：   
```bash   
curl -k -X GET http://47.92.0.84:8001/api/multisignatures/pending?publicKey=2cef5711e61bb5361c544077aa08aebc4d962a1d656571901c48d716382ad4fd   
```   
   
JSON返回示例：   
```js   
{   
	"success": true,   
	"transactions": [{      //上一步中设置账户为多重签名交易的详情，transactionId: 17620378998277022323   
		"min": 2,   
		"lifetime": 1,   
		"signed": true,   
		"transaction": {   
			"type": 4,      //4代表注册多重签名账户   
			"amount": 0,   
			"senderPublicKey": "2cef5711e61bb5361c544077aa08aebc4d962a1d656571901c48d716382ad4fd",   
			"requesterPublicKey": null,   
			"timestamp": 4879978,   
			"asset": {   
				"multisignature": {   
					"min": 2,   
					"keysgroup": ["+eb48b9ab7c9a34a9b7cdf860265d65b31af774355cabf1b3a387d14a1925dc97",   
					"+d5d7aa157f866c47a2a1e09e2746286ed089fd90976b54fbfa930e87d11609cb"],   
					"lifetime": 1   
				}   
			},   
			"recipientId": null,   
			"signature": "a42feaccd9f2a4940fc0be1a1580e786b360f189db3154328f307988e75484293eae391f2f9eee489913cc6d15984eb1f5f5a0aa1bf78ea745d5c725f161af08",   
			"id": "17620378998277022323",   
			"fee": 1500000000,   
			"senderId": "3855903394839129841"   
		}   
	}]   
}   
   
```   
   
### **2.8.3 非交易发起人对交易进行多重签名**   
接口地址：/api/multisignatures/sign   
请求方式：post   
支持格式：json   
请求参数说明：   

|名称	|类型   |必填 |说明              |   
|------ |-----  |---  |----              |   
|secret |string |Y    |DDN账户密码       |   
|secondSecret|string|N|DDN账户二级密码，最小长度：1，最大长度：100|   
|publicKey|string  |N|公钥      |    
|transactionId|string|Y|交易id|   
   
   
返回参数说明：   

|名称	|类型   |说明              |   
|------ |-----  |----              |   
|success|boolean  |是否成功获得response数据 |    
|transactionId|string  |多重签名交易id      |    
   
   
请求示例：   
```bash   
curl -k -H "Content-Type: application/json" -X POST -d '{"secret":"lemon carpet desk accuse clerk future oyster essay seminar force live dog","transactionId":"17620378998277022323"}' 'http://47.92.0.84:8001/api/multisignatures/sign'   //公钥为eb48b9ab7c9a34a9b7cdf860265d65b31af774355cabf1b3a387d14a1925dc97的用户进行签名   
```   
   
JSON返回示例：   
```js   
{   
	"success": true,   
	"transactionId": "17620378998277022323"   
}   
// 此时再次获取pending   
curl -k -X GET http://47.92.0.84:8001/api/multisignatures/pending?publicKey=2cef5711e61bb5361c544077aa08aebc4d962a1d656571901c48d716382ad4fd   
{   
	"success": true,   
	"transactions": [{   
		"min": 2,   
		"lifetime": 1,   
		"signed": true,   
		"transaction": {   
			"type": 4,   
			"amount": 0,   
			"senderPublicKey": "2cef5711e61bb5361c544077aa08aebc4d962a1d656571901c48d716382ad4fd",   
			"requesterPublicKey": null,   
			"timestamp": 4879978,   
			"asset": {   
				"multisignature": {   
					"min": 2,   
					"keysgroup": ["+eb48b9ab7c9a34a9b7cdf860265d65b31af774355cabf1b3a387d14a1925dc97",   
					"+d5d7aa157f866c47a2a1e09e2746286ed089fd90976b54fbfa930e87d11609cb"],   
					"lifetime": 1   
				}   
			},   
			"recipientId": null,   
			"signature": "a42feaccd9f2a4940fc0be1a1580e786b360f189db3154328f307988e75484293eae391f2f9eee489913cc6d15984eb1f5f5a0aa1bf78ea745d5c725f161af08",   
			"id": "17620378998277022323",   
			"fee": 1500000000,   
			"senderId": "3855903394839129841",   
			"signatures": ["b38a161264db2a23e353d3fbc4983562f6343d5ee693144543ca54e2bc67c0f73d1c761b7bfa38b2bb101ac2ab0797b674b1a9964ccd400aaa310746c3494d03"]      //新生成的多重签名   
		}   
	}]   
}   
   
// 公钥为d5d7aa157f866c47a2a1e09e2746286ed089fd90976b54fbfa930e87d11609cb的账户对该注册交易进行签名   
curl -k -H "Content-Type: application/json" -X POST -d '{"secret":"chalk among elbow piece badge try van round quality position simple teach","transactionId":"17620378998277022323"}' 'http://47.92.0.84:8001/api/multisignatures/sign'   
{"success":true,"transactionId":"17620378998277022323"}   
// 此时再次获取pending,结果为空   
curl -k -X GET http://47.92.0.84:8001/api/multisignatures/pending?publicKey=2cef5711e61bb5361c544077aa08aebc4d962a1d656571901c48d716382ad4fd   
{"success":true,"transactions":[]}   
// 查看该注册交易详情（该交易已广播并写入blockchain）,此时该账户已成功注册成为多重签名账户   
curl -k -X GET http://47.92.0.84:8001/api/transactions/get?id=17620378998277022323   
{   
	"success": true,   
	"transaction": {   
		"id": "17620378998277022323",   //注册账户为多重签名用户的交易id   
		"height": "157013",   
		"blockId": "4680888982781013372",   
		"type": 4,   
		"timestamp": 4879978,   
		"senderPublicKey": "2cef5711e61bb5361c544077aa08aebc4d962a1d656571901c48d716382ad4fd",   
		"senderId": "3855903394839129841",   
		"recipientId": "",   
		"amount": 0,   
		"fee": 1500000000,   
		"signature": "a42feaccd9f2a4940fc0be1a1580e786b360f189db3154328f307988e75484293eae391f2f9eee489913cc6d15984eb1f5f5a0aa1bf78ea745d5c725f161af08",   
		"signSignature": "",   
		"signatures": null,   
		"confirmations": "26",   
		"asset": {   
			   
		}   
	}   
}   
   
```   
   
### **2.8.4 获取多重签名账户信息**   
接口地址：/api/multisignatures/accounts   
请求方式：get   
支持格式：urlencoded   
请求参数说明：   

|名称	|类型   |必填 |说明              |   
|------ |-----  |---  |----              |   
|publicKey |string |Y    |多重签名参与者之一的公钥       |   
   
   
返回参数说明：   

|名称	|类型   |说明              |   
|------ |-----  |----              |   
|success|boolean  |是否成功获得response数据 |    
|accounts|Array  |多重签名账户详情      |    
   
   
请求示例：   
```bash   
curl -k -X GET http://47.92.0.84:8001/api/multisignatures/accounts?publicKey=eb48b9ab7c9a34a9b7cdf860265d65b31af774355cabf1b3a387d14a1925dc97   
```   
   
JSON返回示例：   
```js   
{   
	"success": true,   
	"accounts": [{   
		"address": "3855903394839129841",       //多重签名账户地址   
		"balance": 18500000000,     //多重签名账户余额   
		"multisignatures": ["eb48b9ab7c9a34a9b7cdf860265d65b31af774355cabf1b3a387d14a1925dc97",   
		"d5d7aa157f866c47a2a1e09e2746286ed089fd90976b54fbfa930e87d11609cb"],    //多重签名账户公钥   
		"multimin": 2,  //最少签名个数   
		"multilifetime": 1,   
		"multisigaccounts": [{          //签名者账户详情   
			"address": "13542769708474548631",   
			"publicKey": "eb48b9ab7c9a34a9b7cdf860265d65b31af774355cabf1b3a387d14a1925dc97",   
			"balance": 0   
		},   
		{   
			"address": "4100816257782486230",   
			"publicKey": "d5d7aa157f866c47a2a1e09e2746286ed089fd90976b54fbfa930e87d11609cb",   
			"balance": 0   
		}]   
	}]   
}   
```   



