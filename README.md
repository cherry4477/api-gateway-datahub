# api gateway&datahub 合作方案

##1  合作目标：
###1.1 形成统一 的账号资源管理。
###1.2 集成api数据发布—订购—调用—账单全流程用户体验。



##2 合作方案框架
![image](https://raw.githubusercontent.com/asiainfoLDP/api-gateway-datahub/master/object.jpg)



##3 合作内容

###3.1统一用户验证

**新增datahub和交易所区分字段sregion（武汉WH，广州GZ，哈尔滨HEB，datahub）。**

**用户请求Api gateway时，传递token时sregion字段信息以`+`带在token后面，格式：{token}`+`datahub,{token}`+`WH,{token}`+`GZ,{token}`+`HEB）。**

**Api gateway将接收的token中的sregion拆出来，以`+`拼到username前，格式：{sregion}`+`{username}，校验方式不变。**

说明：

用户请求Api gateway时：

	"token": "ef47b6d4670b90eb3cf75a39f0854b0a+datahub"
	"username": xx@aaa.com

Api gateway接收信息转换，验证用户身份：

	"token": "ef47b6d4670b90eb3cf75a39f0854b0a"
	"username": datahub+xx@aaa.com
（比如3.2发布api时url地址）

#### a 用户在datahub上登录，获得请求api的地址、调用api的方式。用户请求Api gateway时，带着用户名、token，api gateway获取到用户的请求后，现在本地查询是否存在用户名、token的匹配信息，若查询到则信任。若查询不到，则到datahub上验证用户身份，若验证身份合法，则在本地存储一份用户名、token的匹配关系。验证方式如下：   
  

校验用户Token的方法：

请求报文

	GET /valid
	Authorization: Token xa12344a
	Authuser: xxx@aaa.com
正确回复

	HTTP/1.1 200 OK
	{"code": 0,"msg": "OK","data": {}}
错误回复

	HTTP/1.1 403 OK
	{"code": 1403,"msg": "not valid","data": {}}



#### b 请求报文的header   
Authorization: Token **后续发送获取订单信息或者调用写接口时发送的请求中报头中带上token信息，datahub验证token的真实性后提交给转给具体服务** 


####c 网关获取自己token的方法
网关在后续调用datahub写服务时，需要在请求报文的header中带着自己的token，获取token的方法如下：
#####Basic认证模式通过用户名和md5(密码获取token)，访问的url为/
######请求报文的header

```
Authorization: Basic user:password的base64编码
```
#####正常情况下返回
```
HTTP/1.1 200 OK
Server: openresty/1.9.3.1
Date: Fri, 08 Jan 2016 05:53:24 GMT
Content-Type: application/json
Transfer-Encoding: chunked
Connection: keep-alive

{"code": 0,"msg": "OK","data": {"token": "ef47b6d4670b90eb3cf75a39f0854b0a"}}
```

#####用户没有激活
```
HTTP/1.1 403 Forbidden
Server: openresty/1.9.3.1
Date: Fri, 08 Jan 2016 05:53:31 GMT
Content-Type: application/json
Transfer-Encoding: chunked
Connection: keep-alive

{"code": 1102,"msg": "user inactive","data": {}}
```

#####密码错误5次内返回
```
HTTP/1.1 403 Forbidden
Server: openresty/1.9.3.1
Date: Fri, 08 Jan 2016 05:53:31 GMT
Content-Type: application/json
Transfer-Encoding: chunked
Connection: keep-alive

{"code": 1101,"msg": "username or password not correct","data": {"retry_times": "2","ttl_times":"86400"}}
```
#####密码错误5次以后，账户被锁定24小时

HTTP/1.1 403 Forbidden
Server: openresty/1.9.3.1
Date: Fri, 08 Jan 2016 05:53:35 GMT
Content-Type: application/json
Transfer-Encoding: chunked
Connection: keep-alive

{"code": 1103,"msg": "retry too many times!!","data": {"retry_times": "5","ttl_times":"86399"}}

####Token认证模式利用上一步获取的token来对需要认证的API提交
#####请求报文的header

```Authorization: Token xa12344a```

 

 
###3.2 信息发布、修改、删除

###用户在datahub上创建repository。
---

1.用户在datahub上创建item，此时单点登录到api gateway上。此时页面向api gateway传递repository名称,用户名、token(http://http://plat-dataex.app-dacp.dataos.io/dataex-plat/ldp/api?reponame=xx&username={username}&apitoken={token})，并在本地查询token的真实性，若本地没有则到datahub验证token的合法性，若存在则信任，同时存储一份到本地。

校验用户Token的方法：

请求报文

	GET /valid
	Authorization: Token xa12344a
	User: xxx@aaa.com
正确回复

	HTTP/1.1 200 OK

	{"code": 0,"msg": "OK","data": {}}
错误回复

	HTTP/1.1 403 OK

	{"code": 1403,"msg": "not valid","data": {}}

2.在api gateway上创建repository下的item,api gateway上生成api商品。

3.Api gateway调用datahub创建item服务创建item

* repo/item名称（item名称限制由英文、数字、_ 组成）
* 更新时间：包括日期 时间，如2015-01-23 11:23:12
* 详情：接口的主要内容、用途介绍。**md格式保存**（文字形式的介绍，如天气api介绍为：全国天气预报，生活指数、实况、PM2.5等信息）
* 接口描述：访问方式、接口地址（每个api的接口地址为 https://hub.dataos.io/repo name/item name,此处api name即为itemname）访问的输入输出介绍、错误代码介绍等。 **md格式保存**
* 请求示例：介绍api请求示例代码、示例返回等。包括curl、pathon、java、c、php等常见的请求示例。**md格式保存**  
如curl请求示例：curl  --get --include  'https://hub.dataos.io/credit/name/输入参数&“您的用户名”&“您的apitoken”'
示例返回：json示例*******
* 开放、私有属性：二选一。
* 价格：**元/**条，**天有效。 每个api可有6个价格包。 
* 服务形式：选择api。
* 创建item接口如下：  

POST /repositories/:repname/:itemname 


	说明【拥有者】发布DataItem ，此时请求head中带着用户的token

输入参数说明 

	ch_itemname                     dataitem中文名称
	itemaccesstype                  访问权限[public(默认), private] 
	meta                            元数据（接口描述） 
	sample                          样例数据（请求示例） 
	comment                         详情 
	price                           计费计划 
	price.units                     购买数量 
	price.money                     价格 
	price.expire                    有效期(天) 
	price.limit                     限购次数【可选】
	label.sys.supply_style          服务形式[api；batch；flow]【必选】 
	label.sys.supply_style.api      实时单条 
	label.sys.supply_style.batch    批量 
	label.sys.supply_style.flow     流式
	label.sys.dacp.id               网关dataitem id                      



Example Request： 
	
	POST /repositories/chinamobile/beijingphone HTTP/1.1  
	Authorization: Token dcabfefb6ad8feb68e6fbce876fbfe778fb 
	
	{ 
	    "itemaccesstype": "private",
	    "meta": "{}",
	    "sample": "{}",
	    "comment": "对终端使用情况、变化情况进行了全方面的分析。包括分品牌统计市场存量、新增、机型、数量、换机等情况。终端与ARPU、DOU、网龄的映射关系。终端的APP安装情况等。",
	    "price":[
	                {
	                    "units": 30,
	                    "money": 5,
	                    "expire":30,
	                    "limit":1,
	                },
	                {
	                    "units": 30,
	                    "money": 5,
	                    "expire":30,
	                    "limit":1,
	                }
	            ],
	    "label": {
	        "sys": {
	            "supply_style": "flow",
	            "dacp": {
	            	"id": "7FF7B39E-F7AC-41E4-8975-7EAE629DE6DD"
	            }
	        },
	        "opt": {},
	        "owner": {},
	        "other": {}
	     }
        }	     

####用户在datahub上修改已发布的item。
1.用户在datahub上找到要修改的item，此时单点登录到api gateway上。此时页面向api gateway传递repository名称,dataitem名称，用户名、token(http://http://plat-dataex.app-dacp.dataos.io/dataex-plat/ldp/api?reponame=xx&itemname=xx&username={username}&apitoken={token})，并在本地查询token的真实性，若本地没有则到datahub验证token的合法性，若存在则信任，同时存储一份到本地。

2.在api gateway上修改repository下的item,api gateway上更新api商品信息。

3.Api gateway调用datahub更新item服务更新item

* repo/item名称（item名称限制由英文、数字、_ 组成）
* 更新时间：包括日期 时间，如2015-01-23 11:23:12
* 详情：接口的主要内容、用途介绍。**md格式保存**（文字形式的介绍，如天气api介绍为：全国天气预报，生活指数、实况、PM2.5等信息）
* 接口描述：访问方式、接口地址（每个api的接口地址为 https://hub.dataos.io/repo name/item name,此处api name即为itemname）访问的输入输出介绍、错误代码介绍等。 **md格式保存**
* 请求示例：介绍api请求示例代码、示例返回等。包括curl、pathon、java、c、php等常见的请求示例。**md格式保存**  
如curl请求示例：curl  --get --include  'https://hub.dataos.io/credit/name/输入参数&“您的用户名”&“您的apitoken”'
示例返回：json示例*******
* 开放、私有属性：二选一。
* 价格：**元/**条，**天有效。 每个api可有6个价格包。 
* 服务形式：选择api。
* 更新item接口如下： 

PUT /repositories/:repname/:itemname

	说明【拥有者】更新DataItem

输入参数说明

	ch_itemname                     dataitem中文名称
	itemaccesstype  				访问权限[public(默认), private]
	meta							元数据
	sample							样例数据
	comment							详情
    price					        计费计划
    price.units                     购买数量
    price.money                     价格
    price.expire                    有效期(天)
    price.limit                     限购次数【可选】
			
Example Request：

	PUT /repositories/chinamobile/beijingphone HTTP/1.1 
	Authorization: Token dcabfefb6ad8feb68e6fbce876fbfe778fb
	{
	    "ch_itemname": "Dataitem中文名称",
        "itemaccesstype": "private",
        "meta": "{}",
        "sample": "{}",
		"price":[
					{
                    	"units": 30,
                        "money": 5,
                        "expire":30,
                        "limit":1,
                        "plan_id":"100000000000000000000001"
                    },
                    {
                    	"units": 30,
                        "money": 5,
                        "expire":30,
                        "limit":1,
                        "plan_id":"100000000000000000000002"
                    },
                    {
                        "units": 30,
                        "money": 5,
                        "expire":30,
                        "limit":1,
                    }
				],
        "comment": "对终端使用情况、变化情况进行了全方面的分析。包括分品牌统计市场存量、新增、机型、数量、换机等情况。终端与ARPU、DOU、网龄的映射关系。终端的APP安装情况等。"      
    }

####用户在datahub上删除已发布的item。
---
1.用户在datahub上删除的item。

2.Api gateway上同步删除api商品。


###3.3 api gateway取订单

####a 用户在datahub上订购api（每个api name即为item name），生成订单。
####b datahub向api gateway提供查询接口，查询用户的api订单。采用增量查询的方式，并存储在本地做配额控制。 
####c datahub提供的查询接口包括如下信息：订单号、订购方、api名、订单配额、订单有效期。
####d api网关根据订单配额、订单有效期来做流量控制，这两个因素中有一个达到极限值，则该订单失效。若用户继续调用对应的api，则去查询是否有新的订单生成，若有用新订单的配额；若没有则拒绝调用。

 查询订单接口如下：

 
GET /subscriptions/pull/:repname/:itemname?username={username} 

说明 

	【API网关】查询在某个用户(通过username指定)在某个dataitem上的所有尚未发送给api网关的phase=1的订购

	注意1：此api需要传递API网关自己的auth token
	注意2：当成功取得订购并存储在本地后，需调用api#3.4标识订单取走状态
	注意3：此API最多返回100条记录。
	
输入参数说明： 
	
	username: 订购的需求者。

输入样例： 

	GET /subscriptions/pull/repo001/item002?username=zhang3@example.com HTTP/1.1 
	Accept: application/json; charset=utf-8
	Authorization: Token dcabfefb6ad8feb68e6fbce876fbfe778fb

输出样例：  

	HTTP/1.1 200 OK

	{  
	
	    "total": 2,
	    "results": [
	        {
	            "subscriptionid": 1234567,
	            "sellername": "li4@example.com",
	            "supply_style":"batch",
	            "sorttime":"2015-11-10T15:04:05Z",
	            "signtime":"2015-11-10T15:04:05Z",
	            "expiretime":"2016-01-15T11:28:21Z",
	            "phase":1,
	            "plan":{
	                “id": "222222222",
	                "money":5,
	                "units":3,
	                "used":0,
	                "limit":0,
	                "expire":30
	            }
	        },
	        {
	            "subscriptionid": 1234568,
	            "sellername": li4@example.com",
	            "supply_style":"flow",
	            "sorttime":"2015-11-01T15:04:05Z",
	            "signtime":"2015-11-01T15:04:05Z",
	            "expiretime":"2015-11-04T15:04:05Z"
	            "phase":1,
	            "plan":{
	                “id": "33333333",
	                "money":5,
	                "units":3,
	                "used":0,
	                "limit":0,
	                "expire":30
	            }
	        }
	    ]
	}

输出数据说明（当phase=1的时候）：

	subscriptionid: 订单号
	sellername: 数据提供者
	repname: repository name
	itemname: data item name
	supply_style: flow | batch 
	sorttime: 排序时间（== signtime）
	signtime: 订购时间
	expiretime: 自动过期时间
	phase: 1-3, 5-10 (意义：consuming: 1, freezed: 2, finished: 3, cancelled: 5, removed: 6, applying: 7, wthdrawn: 8, denied: 9, complained: 10)
	plan.id: 价格计划id
	plan.money: 交易金额
	plan.units: 最大下载次数（supply_style=batch）,最大下载天数（supply_style=flow)
	plan.used: 已经使用量　
	plan.limit: 最多可以订购次数
	plan.expire: 交易有效期（天数）

###3.4 api gateway标识订单取走状态

每取走一个订单，则标识该订单为已经取走状态。此时以api gateway的身份置订单取走状态。

PUT /subscription/:subscriptionid

说明

	【API网关】回应已经成功取走了某个订购 (action=set_retrieved)
	
	注意：此api需要API网关传递自己的auth token

输入参数说明：
	
	action: set_retrieved
	repname: 订购的repname, 供校验用
	itemname: 订购的itemname, 供校验用
	username: 订购的需求者, 供校验用

输入样例：

	PUT /subscription/1234567 HTTP/1.1 
	Accept: application/json; charset=utf-8
	Authorization: Token dcabfefb6ad8feb68e6fbce876fbfe778fb
	
	{
		"action": "set_retrieved",
		"repname": "repo001",
		"itemname": "item002",
		"username": "zhang3@example.com"
	}

输出样例1：
        
	HTTP/1.1 200 OK
	
	{"code": 0,"msg": "OK"}

输出样例2：
        
	HTTP/1.1 200 OK
	
	{"code": 0,"msg": "OK"}


###3.5 api gateway每天将每条api订单（生效状态的订单，已经完成的订单不需要）的调用次数

写入datahub上的订单“已用量”字段中。 
每天12点写入。此时以api gateway的身份置每条订单的使用全量。  
若某条订单当天完成，则立即回写使用量。不用等到12点写入。

PUT /subscription/:subscriptionid

说明

	【API网关】同步某个订购的已使用量 (action=set_plan_used)
	
	注意：此api需要API网关传递自己的auth token

输入参数说明：
	
	action: set_plan_used
	used: 新的使用量
	repname: 订购的repname, 供校验用
	itemname: 订购的itemname, 供校验用
	username: 订购的需求者, 供校验用

输入样例1：

	PUT /subscription/1234567 HTTP/1.1 
	Accept: application/json; charset=utf-8
	Authorization: Token dcabfefb6ad8feb68e6fbce876fbfe778fb
	
	{
		"action": "set_plan_used",
		"used": 123,
		"repname": "repo001",
		"itemname": "item002",
		"username": "zhang3@example.com"
	}

输出样例1：
        
	HTTP/1.1 200 OK
	Content-Type: application/json; charset=utf-8
	Date: Fri, 22 Apr 2016 09:08:09 GMT
	Content-Length: 21
	
	{"code":0,"msg":"OK"}

输出样例2：

	HTTP/1.1 400 Bad Request
	Content-Type: application/json; charset=utf-8
	Date: Fri, 22 Apr 2016 09:05:41 GMT
	Content-Length: 92
	
	{"code":5043,"msg":"failed to modfiy subscription (subscription is not in consuming phase)"}

###3.6 apigate 按月生成每个用户调用详单。
用户在datahub上查询调用api的详单，单点登录到api gateway。 查询每月的调用量。此处ui特征需要与datahub保持一致。此处单点登录的方式与3.2一致。
详单包括 Api name  订单号  调用时间   


###3.7 datahub api访问地址
https://10.1.235.98/api
