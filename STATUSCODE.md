### 状态码

本状态码即指http状态码，服务端返回给客户端的http状态嘛，将遵循RESTful Api设计原则。

### 常见状态码及说明


```php
200 OK - [GET]：服务器成功返回用户请求的数据，该操作是幂等的（Idempotent）。
201 CREATED - [POST/PUT/PATCH]：用户新建或修改数据成功。
202 Accepted - [*]：表示一个请求已经进入后台排队（异步任务）
204 NO CONTENT - [DELETE]：用户删除数据成功。
400 INVALID REQUEST - [POST/PUT/PATCH]：用户发出的请求有错误，服务器没有进行新建或修改数据的操作，该操作是幂等的。
401 Unauthorized - [*]：表示用户没有权限（令牌、用户名、密码错误）。
403 Forbidden - [*] 表示用户得到授权（与401错误相对），但是访问是被禁止的。
404 NOT FOUND - [*]：用户发出的请求针对的是不存在的记录，服务器没有进行操作，该操作是幂等的。
406 Not Acceptable - [GET]：用户请求的格式不可得（比如用户请求JSON格式，但是只有XML格式）。
410 Gone -[GET]：用户请求的资源被永久删除，且不会再得到的。
422 Unprocesable entity - [POST/PUT/PATCH] 当创建一个对象时，发生一个验证错误。
500 INTERNAL SERVER ERROR - [*]：服务器发生错误，用户将无法判断发出的请求是否成功。
```

* 200 - 299 表示正常的请求与访问
* 400 - 499 一般表示客户端异常
* 500 - 599 一般表示服务端异常

* 已经定义好的状态码一般能满足绝大部分业务。此状态可以当成一个分类，而不能表现出具体的含义，具体的含义需在数据包中体现。
* 如【用户登录】业务;发生登录失败，如果是客户端原因，密码错误将返回401，禁止登录将返回403，其他错误将返回定义好的状态码，如402。
* 我们可以自定义未定义的状态码，如402，但定义状态码应表示一个分类的标识。如我们可以把402作为操作失败，而不应该作为验证码获取失败。

###状态码使用

一般将在控制器中向客户端返回数据包及携带状态码。 每个控制器将继承```App\Http\Traits\Responsed```Trait 。可在此Trait中增加状态码常量，命名将以```RESPONSE_CODE_```为前缀并且大写且具意义，如```define('RESPONSE_CODE_SENDEMAIL_FAIL', 477);//邮件发送失败``` 。

控制器调用
```php
//成功返回
return $this->successResponse($result，RESPONSE_CODE_CREATED);

//操作失败返回
return $this->errResponse('邮件地址不存在',RESPONSE_CODE_SENDEMAIL_FAIL);
```






