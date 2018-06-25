### 验证器

#####为什么要使用验证器

* 人类之所以能在地球上横行无忌，这得益于他们紧密高水准的协作，做就是人与普通动物的最主要区别之一。
* 汽车生产线可谓最复杂的生产线之一了，每年大量的汽车生产，既要保证效率，还有极为严格的品控管理，这就得把生产线工种的颗粒度到最小化。验证器便是Lavarel从生命周期中优化而来的产物，把它从控制器中抽离出来，达到解耦，复用等核心程序思维的目的。


#####验证器的使用

一个简单的验证类

```php
<?php

namespace App\Http\Requests;

use Illuminate\Foundation\Http\FormRequest; // 也可以 用项目中可以修改的基类  App\Http\Requests\Request

class StoreCommentRequest extends FormRequest
{
    /**
     * 这个方法可以用来控制访问权限，例如禁止普通用户发帖…
     *
     * @return bool
     */
    public function authorize() 
    {
        return true; 
    }

    /**
     * 这个方法返回验证规则数组，也就是Validator的验证规则
     *
     * @return array
     */
    public function rules() 
    {
        return [
            //
        ];
    }
        /**
     * 这个方法返回rules中各中验证不通过提示的数组
     *
     * @return array
     */
    public function messages()
    {
        return [
          //
        ];
    }
}
```
接着我们只需要在Controller中注入改类即可

```php
<?php

// ...

    // 之前：public function postStoreComment(Request $request)
    public function postStoreComment(\App\Http\Requests\StoreCommentRequest $request)
    {
        // ...
    }

// ...

```

* 需要注意的是表单验证失败 Lavarel通常会抛出一个 ```Illuminate\Http\Exception\HttpResponseException```异常 这时候需要在```app\Exceptions\handler.php```中统一处理

更多参考APi文档 [Lavarel5.6 Api FormRequest](https://laravel.com/api/5.6/Illuminate/Foundation/Http/FormRequest.html)

