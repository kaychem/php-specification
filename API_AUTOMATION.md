
# 自动化API文档  

- - -

工具：  
> swagger-php

## 安装（composer）
  
    composer require zircote/swagger-php

## 添加文档

接口文档推荐写在接口方法上方的注释，并多引用自定义对象  

### 项目定义

```php
/**
 * @SWG\Swagger(
 *   host="petstore.swagger.io",                项目URL*
 *   basePath="/api",                           API根路径*
 *   schemes={"http"},                          API协议（http, https, ws, wss）
 *   produces={"application/json"},             全局接收参数格式
 *   consumes={"application/json"},             全局返回参数格式
 *   @SWG\ExternalDocumentation(                外部扩展文件
 *     description="find more info here",           文件描述
 *     url="https://swagger.io/about"               文件路径URL
 *   )
 *   @SWG\Info(
 *      title="Swagger Petstore",                      API文档名*
 *      description="A sample API",                    文档描述
 *      version="1.0.0",                               版本号
 *      @SWG\Contact(                                  联系方式
 *          email="apiteam@swagger.io",
 *          name="Swagger API Team",
 *          url="http://swagger.io"
 *      ),
 *      @SWG\License(                                  授权
 *          name="MIT",
 *          url="http://github.com/gruntjs/grunt/blob/master/LICENSE-MIT"
 *      ),
 * )
 * )
 */
```

#### produces和consumes可选参数：

```php
application/json
application/xml
application/x-www-form-urlencoded
multipart/form-data
text/plain; charset=utf-8
text/html
application/pdf
image/png
```

### 接口定义  

```php
/**
 * @SWG\Get(                                            请求方法
 *     path="/pets",                                    接口URI*
 *     operationId="addPet",                            唯一操作ID
 *     summary="pets",                                  简要说明
 *     description="pets",                              描述
 *     produces={"application/json"},                   接收参数格式
 *     tags={"pets"}                                    自定义接口标签*
 *     @SWG\Parameter(                                  请求变量
 *         name="id",                                       变量名
 *         in="path",                                       变量位置
 *         type="integer",                                  类型（object,array,string,integer等）
 *         default="1",                                     默认值
 *         description="Pets id",           描述
 *         required=true,                                   是否必须
 *     ),
 *     @SWG\Parameter(                                  请求变量
 *          name="body",
 *          in="body",
 *          description="serach",
 *          @SWG\Schema(ref="#/definitions/Search")         引用对象
 *     ),
 *     @SWG\Response(                                   返回数据
 *         response=200,                                    状态码
 *         description="pet response",                      描述
 *         @SWG\Header(                                     定义响应头
 *              header="X-Expires-After",                       响应头
 *              type="string",                                  类型
 *              format="date-time",                             格式
 *              description="date in UTC when token"            描述
 *          )  
 *     ),
 *     @SWG\Response(
 *         response="default",  
 *         description="unexpected error",
 *         @SWG\Schema(ref="#/definitions/ERROR")       引用自定义对象
 *     )
 * )
 */
```

#### 请求变量里in可选参数：

```php
header      位于HTTP  header
path        URI路径   such as /users/{id}
query       URI变量   such as /users?role=admin
body        HTTP body
formData    表单请求
```

### 自定义对象引用

```php

/**
 * @SWG\Definition(                         自定义对象关键字*
 *   definition="NewPet",                   对象名*
 *   type="object",                         对象类型*
 *   required={"id"},                       必须字段
 *   @SWG\Property(                         属性*
 *      property="id",                          属性名
 *      format="int64",                         格式
 *      type="integer"                          类型
 *      description="sss"                       描述
 *      example="15"                            例子
 *   )
 *   @SWG\Property(
 *      property="title",
 *      type="string",
 *      description="desc"
 *   )
 * )
 */
 /**
  *  为了减少重复定义，可在已有自定义对象上添加字段以生成新的对象，
  *
  *  @SWG\Definition(
  *   definition="Pet",
  *   type="object",
  *   allOf={                                           嵌套引用
  *       @SWG\Schema(ref="#/definitions/NewPet"),      引用上面定义的对象
  *       @SWG\Schema(                                  添加Schema
  *           required={"color"},                       必须字段
  *           @SWG\Property(                            属性
  *             property="color",
  *             type="string")
  *       )
  *   }
  * )
  *
  *
  *
  * @SWG\Definition(
  *   definition="NewPet",
  *   type="array",                              定义数组
  *      @SWG\Items(ref="#/definitions/User")        值
  * )
  *
  *
  *
  * @SWG\Definition(
  *     definition="RotationPictureList",
  *     description="轮播图分类",
  *     allOf={
  *        @SWG\Schema(ref="#/definitions/RotationPictureCate"),
  *        @SWG\Schema(
  *              @SWG\Property(                                     picture字段为数组
  *              property="picture",
  *              type="array",
  *              @SWG\Items(ref="#/definitions/RotationPicture"),   再嵌套一个对象
  *             )
  *        )
  *      }
  * )
  *
  *        上面这种定义方式可以方便的定义复杂的多层属性的列表，这样在返回体里直接引用这个List就可以了
  */
```

### 引用  

```php
/*
 *     @SWG\Parameter(
 *          name="search",
 *          in="body",
 *          description="",
 *          @SWG\Schema(ref="#/definitions/Pet")
 *     ),
 *     @SWG\Parameter(
 *          name="search",
 *          in="body",
 *          description="",
 *          @SWG\Schema(ref="#/definitions/RotationPictureList")
 *     ),
 */
```

> 注：建议自定义对象另建目录，并以模块为一个文件来定义

#### 完整例子

```php
    /**
     * Post请求
     * @SWG\Post(
     *     path="/news",
     *     summary="添加快讯",
     *     description="添加快讯",
     *     tags={"资讯快讯"},
     *     @SWG\Parameter(
     *          name="body",
     *          in="body",
     *          description="",
     *          @SWG\Schema(ref="#/definitions/BriefNews")
     *     ),
     *      @SWG\Response(
     *          response=200,
     *          description="快讯列表",
     *          @SWG\Schema(
     *              @SWG\Property(
     *                  property="message",
     *                  type="string",
     *                  description="返回消息"
     *              )
     *          )
     *      )
     * )
     *
     * @SWG\Definition(
     *     definition="BriefNews",
     *     required={"title", "content"},
     *     description="后台快讯",
     *        @SWG\Property(
     *            property="id",
     *            type="integer",
     *            description="id"
     *        ),
     *        @SWG\Property(
     *            property="title",
     *            type="string",
     *            description="标题"
     *        ),
     *        @SWG\Property(
     *            property="content",
     *            type="string",
     *            description="内容"
     *        ),
     *        @SWG\Property(
     *            property="sync_created",
     *            type="string",
     *            description="发布时间"
     *        ),
     * )
     * */
     /**
      * Get请求
      * @SWG\Get(
      *     path="/rotation-picture-cate",
      *     summary="轮播图分类列表",
      *     tags={"轮播图"},
      *     @SWG\Parameter(
      *          name="position_code",
      *          in="query",
      *          type="string",
      *          description="轮播图分类编码",
      *      ),
      *      @SWG\Response(
      *          response=200,
      *          description="轮播图分类列表",
      *          @SWG\Schema(ref="#/definitions/RotationPictureList")
      *      )
      *      @SWG\Response(
      *          response=404,
      *          description="Not found",
      *      ),
      * )
      * @SWG\Definition(
      *     definition="RotationPictureCate",
      *     required={"title"},
      *     description="轮播图分类",
      *     @SWG\Property(
      *         property="title",
      *         type="string",
      *         description="标题"
      *      ),
      * )
      *
      * @SWG\Definition(
      *     definition="RotationPicture",
      *     description="轮播图",
      *        @SWG\Property(
      *            property="cate_id",
      *            type="integer",
      *            description="轮播图分类id"
      *        ),
      * )
      *
      * @SWG\Definition(
      *     definition="RotationPictureList",
      *     description="轮播图分类",
      *     allOf={
      *        @SWG\Schema(ref="#/definitions/RotationPictureCate"),
      *        @SWG\Schema(
      *              @SWG\Property(
      *              property="picture",
      *              type="array",
      *              @SWG\Items(ref="#/definitions/RotationPicture"),
      *             )
      *        )
      *      }
      * )
      * */
```

文档的写法请参考:
> [github](https://github.com/zircote/swagger-php)  里的Examples  
> [官网](https://swagger.io/docs/specification/2-0/basic-structure)  
> 或者参考[这里](>https://laravel-china.org/index.php/topics/7430/how-to-write-api-documents-based-on-swagger-php)

swagger
  
#### 生成API文档

1) 创建一个SwaggerController  
    添加方法getJSON  

    ```php

        public function getJSON()
        {
            // 你可以将API的'Swagger Annotation'写在实现API的代码旁，从而方便维护，
            // 'swagger-php'会扫描你定义的目录，自动合并所有定义。这里我们直接用'Controller/'
            // 文件夹。
            $swagger = \Swagger\scan(app_path('Http/Controllers/'));
            return response()->json($swagger, 200);
        }

    ```

    添加上相应的路由后访问，如有下面返回则生成成功

    ```json
    {"swagger":"2.0","info":{"title":"\u6211\u7684`Swagger`API\u6587\u6863","version":"1.0.0"},"paths":{},"definitions":{}}
    ```

2) 在项目根目录运行  
  
        ./vendor/bin/swagger ./app/http -o ./docs/swagger

    则会对/app/http目录进行扫描，并在docs目录下生成JSON格式的API文档

#### 查看文档

访问 [swagger-ui](http://lliao.net:18030/swagger)  
输入生成的json文件的地址即可

#### 相关资料

>https://github.com/zircote/swagger-php
>http://swagger.io/  
>https://www.jianshu.com/p/6840514c4c8e  
>https://laravel-china.org/index.php/topics/7430/how-to-write-api-documents-based-on-swagger-php