#swagger格式样例和说明
	swagger: '2.0' ##必填，使用指定的swagger规范版本。
	info:          ##可以不填，api的版本信息。
 	 version: 1.0.0
 	 title: DACP api
  	description: |
	host: https://hub.dataos.io：9091  ##主机名或ip服务API，api请求地址。咱们是：https://	hub.dataos.io：9091 
	basePath: /Finance                    ##API的基本路径,这是相对的host,咱们是/repo name 。比如用户在dacp创建的api所在的repo name为 Finance，api name 为ThemeHeat ，那么basePath为 /Finance，下面的path参数为/ThemeHeat。
	schemes:       ##API的传输协议。 值必须从列表中:"http","https","ws","wss"。
 	 - http
  	- https
	consumes:    ##咱们用不到。“一个MIME类型的api可以使用列表。 这是可以覆盖全球所有API,但在特定的API调用。 值必须是所描述的Mime类型。  ” 
 	 - application/json
 	 - text/xml
	produces:      ##咱们的必填信息 ，表明api的格式。“MIME类型的api可以产生的列表。 这是可以覆盖全球所有API,但在特定的API调用。 值必须是所描述的Mime类型。”
 	 - application/json
  	- text/html
	paths:         ##必选信息。咱们是/api name 。比如用户在dacp创建的api所在的repo name为 Finance，api name 为ThemeHeat ，那么前面的basePath为 /Finance，path参数为/ThemeHeat。
  	/ThemeHeat
    get: 
      summary:api描述摘要 ##api描述摘要。没有可以空缺。
      description:api描述  ##api的具体描述。
      parameters:   ##具体的api请求参数信息
        - name: name    ##第一个参数名称
          in: query      ##对于请求参数均为in
          description: 姓名   ##第一个参数描述
          type: string   ##第一个参数类型
          required: true   ##是否为必选，若为必选则为true
          default: 测试人    ##第一个参数默认值
        - name: birthday   ##第二个请求参数
          in: query
          description: 生日
          type: integer
          required: true
          default: 33
      responses:    ##反回信息
        200:    ##第一种返回码
          description:  List all pets   ##第一种返回主题
          schema:        ##返回类型描述
            type: array     ##返回的类容的类型
            items:
              $ref: '#/definitions/Pet'  ##返回的具体内容，表达形式为#/definitions/$具体内容$

	definitions:  ##下面是具体定义，这部分在上面信息的基础上详细描述每个参数的具体含义，这部分没有可以空缺。
  	ThemeHeat:    #api名称
    type: object  ##类型
    properties:
      name:     ##第一个请求参数的名称
        type: string   ##第一个参数的描述的类型
        description:****   ##第一个参数的具体描述
      birthday:     ##第二个请求参数的名称
        type: integer    ##第二个参数的描述的类型
        format: int32  ##第二个参数的具体描述

