### Lavarel 资源控制器

Laravel 的资源控制器可以让我们很便捷地构建基于资源的 RESTful 控制器，例如，你可能想要在应用中创建一个控制器，用于处理关于文章存储的 HTTP 请求，使用 Artisan 命令 make:controller，我们可以快速创建这样的控制器：

```php
php artisan make:controller PostController --resource
```

同理，仓库模式```prettus/l5-repository```所提供的方法也能创建同样的资源控制器，前提是需先配置模板。

```php
php artisan make:entity Post
```

资源控制器处理的动作

<table>
    <tr>
        <th>请求方式</th>
        <th>URI路径</th>
        <th>控制器方法</th>
        <th>路由名称</th>
    </tr>
    <tr>
        <td>GET</td>
        <td>/posts</td>
        <td>index</td>
        <td>posts.index</td>
    </tr>
    <tr>
        <td>GET</td>
        <td>/posts/create</td>
        <td>create</td>
        <td>posts.create</td>
    </tr>
    <tr>
        <td>POST</td>
        <td>/posts</td>
        <td>store</td>
        <td>posts.store</td>
    </tr>
    <tr>
        <td>GET</td>
        <td>/posts/{id}</td>
        <td>show</td>
        <td>posts.show</td>
    </tr>
    <tr>
        <td>GET</td>
        <td>/posts/{id}/edit</td>
        <td>edit</td>
        <td>posts.edit</td>
    </tr>
    <tr>
        <td>PUT/PATCH</td>
        <td>/posts/{id}</td>
        <td>update</td>
        <td>posts.update</td>
    </tr>
    <tr>
        <td>DELETE</td>
        <td>/posts/{id}</td>
        <td>destroy</td>
        <td>posts.destroy</td>
    </tr>
    
</table>

###路由注册

resource路由能满足普通增删查改的业务员操作，一般将先注册resource路由。如果有额外的业务操作，应将注册写在resource之前。

```php
//注册一个完整的资源控制路由
Route::resource('post', 'PostController');
```

```php
//声明资源路由时可以指定该路由处理的动作子集
Route::resource('post', 'PostController', ['only' => 
    ['index', 'show']
]);

Route::resource('post', 'PostController', ['except' => 
    ['create', 'store', 'update', 'destroy']
]);
```

```php
//注册额外业务的路由，如邀请业务
Route::post('post/apply/{id}', 'PostController@apply');
Route::resource('post', 'PostController');
```


