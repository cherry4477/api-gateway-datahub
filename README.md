# api gateway&datahub 合作方案

##1  合作目标：
###1.1 形成统一 的账号资源管理。
###1.2 集成api数据发布—订购—调用—账单全流程用户体验。



##2 合作方案框架
![image](https://raw.githubusercontent.com/asiainfoLDP/api-gateway-datahub/master/object.jpg)



##3 合作内容

###3.1统一用户验证
#### a 用户在datahub上登录，获得请求api的地址、调用api的方式。用户请求Api gateway时，带着用户名、密码，api gateway获取到用户的请求后，以basic方式生成token。 

#### b basic方式生成token
Basic base64(用户名:md5(密码))

#### c 请求报文的header   
Authorization: Token **token信息为上一步用basic生成，后续发送获取订单信息或者调用写接口时发送的请求中报头中带上token信息，datahub验证token的真实性后提交给转给具体服务** 

 

 
###3.2 信息发布
####a 用户在datahub上创建repository。

####b 用户在datahub上创建item，此时单点登录到api gateway上。此时页面向api gateway传递用户tocken、repository名称。Api gateway向现在本地查询tocken的真实性，若本地没有则到datahub登陆服务验证tocken的合法性，若存在则信任，同时存储一份到本地。

####c 在api gateway上创建repository下的item,api gateway上生成api商品。

####d Api gateway调用datahub创建item服务创建item

* repo/item名称（item名称限制由英文、数字、_ 组成）
* 更新时间：包括日期 时间，如2015-01-23 11:23:12
* 详情：接口的主要内容、用途介绍。**md格式保存**（文字形式的介绍，如天气api介绍为：全国天气预报，生活指数、实况、PM2.5等信息）
* 接口描述：访问方式、接口地址（每个api的接口地址为 https://hub.dataos.io/repo name/item name,此处api name即为itemname）访问的输入输出介绍、错误代码介绍等。 **md格式保存**
* 请求示例：介绍api请求示例代码、示例返回等。包括curl、pathon、java、c、php等常见的请求示例。**md格式保存**  
如curl请求示例：curl  --get --include  'https://hub.dataos.io/crdit/name/输入参数=“您的用户名”&“您的密码”'
示例返回：json示例*******
* 开放、私有属性：二选一。
* 价格：**元/**条，**天有效。 每个api可有6个价格包。 
* 服务形式：选择api。
* 创建item接口如下：  


POST /repositories/:repname/:itemname 


说明【拥有者】发布DataItem 


输入参数说明 


itemaccesstype                  访问权限[public(默认), private] 


meta                            元数据（详细说明） 


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
            "supply_style": "flow"
        },
        "opt": {},
        "owner": {},
        "other": {}
    
}

###3.3 api gateway取订单
####a 用户在datahub上订购api（每个api name即为item name），生成订单。
####b datahub向api gateway提供查询接口，查询用户的api订单。采用增量查询的方式，并存储在本地做配额控制。 
####c datahub提供的查询接口包括如下信息：订单号、订购方、api名、订单配额、订单有效期。
####d api网关根据订单配额、订单有效期来做流量控制，这两个因素中有一个达到极限值，则该订单失效。若用户继续调用对应的api，则去查询是否有新的订单生成，若有用新订单的配额；若没有则拒绝调用。
 查询订单接口如下： 

 
GET /subscriptions/pull/:repname/:itemname?groupbydate=[0|1]&legal=[0|1]&phase={phase}&page={page}&size={size} 


说明 


【需求者】查询在某个dataitem上的所有订购 


输入参数说明： 


groupbydate: (可选，默认为0) 是否按日期分组。  **该字段不需要写**

legal: (可选) 整数(1表示正式签署的订购，等价于phase为1,2或3。0表示未正式签订的订购，等价于phase不等于1,2和3)。如果指定，将压制phase参数。  **该字段不需要写**

phase: (可选) 整数(consuming: 1, freezed: 2, finished: 3, cancelled: 5, removed: 6, applying: 7, wthdrawn: 8, denied: 9, agreed_but_insufficient_balance: 10)。如果此参数未指定或者legal参数被指定，此参数将被忽略。  **此处选择1，为生效状态的订单**

page: (可选, 默认为1) 第几页，最小值为1。  **该字段不需要写**

size: (可选, 默认为30) 每页最多返回多少条数据, 最小值为1, 最大值为100。  **该字段不需要写**

输入样例： 

GET /subscriptions/pull/repo001/item002 HTTP/1.1  

Accept: application/json 

Authorization: Token dcabfefb6ad8feb68e6fbce876fbfe778fb 

输出样例：  

{  

    "total": 100,
    "results": [
        {
            "subscriptionid": 1234567,
            "sellername": "li4@example.com",
            "supply_style":"batch",
            "sorttime":"2015-11-10T15:04:05Z",
            "signtime":"2015-11-10T15:04:05Z",
            "expiretime":"2016-01-15T11:28:21Z",
            "freezetime":"2015-12-11T10:51:11Z",
            "finishtime":"2016-01-10T10:51:11Z",
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
            "freezetime":"2015-12-04T15:04:05Z",
            "finishtime":"2016-01-04T15:04:05Z",
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


###4 api gateway标识订单取走状态
每取走一个订单，则标识该订单为已经取走状态。
***此处datahub新增一个写接口。待接口定义好后补充。***  


###5 api gateway每天将每条api订单（生效状态的订单，已经完成的订单不需要）的调用次数 
写入datahub上的订单“已用量”字段中。每天12点写入。
***此处datahub新增一个写接口，api gateway确保写入成功。待接口定义好后补充。***  


###6 apigate 按月生成每个用户调用详单。
用户在datahub上查询调用api的详单，单点登录到api gateway。 查询每月的调用量。此处ui特征需要与datahub保持一致。
详单包括 Api name  订单号  调用时间   


###7 datahub api访问地址
https://10.1.235.98/api
