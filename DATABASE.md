###数据库操作

####花拳绣腿之DB及构造器


数据库查询构建器提供了一个方便的流接口用于创建和执行数据库查询。查询构建器可以用于执行应用中绝大部分数据库操作，并且能够在 Laravel 支持的所有数据库系统上工作。

流接口是一种设计模式，更多关于流接口模式的设计和使用方式，可查看这篇教程：[PHP 设计模式系列 —— 流接口模式](http://laravelacademy.org/post/2828.html)。

在Lavarel构造器中命名与原生sql相差无几，几乎在sql中所能出现的关键词在构造器都有对应的方法，而且命名高度一致。下面列出常见的构造语句，几乎从关键词中就能看出相关含义，并不需要过多的解释。

下面都需用到的DB门面```Illuminate\Support\Facades\DB```

```php
//查询单条
$company = DB::table('company')->where('name', 'lxtx')->first();
//多条 orderBy skip
... ->limit(1)->orderBy('created_at','asc')->skip(2)->get();
//where
... ->where('votes', '>', 100)->orWhere('name', 'John');...
... ->where(function(){
$query->where('votes', '>', 100);
$query->orWhere('name', 'John');

... ->whereIn('id',[1,2,3]); ...
... ->whereNotIn('id',[1,2,3]); ...
... ->whereNull('name'); ...
... ->whereNotNull('name'); ...

... ->whereExists(function ($query) {
            $query->select(DB::raw(1))
                  ->from('orders')
                  ->whereRaw('orders.user_id = users.id');
        }); ...
        
//groupBy having
... ->groupBy('rid')->having('max(id)'); ...

//聚合函数
... ->max('price');
//记录是否存在
... ->exists();  ... ->doesntExist();
//去重
... ->distinct();

```




```php
// join
... ->join('contacts', 'users.id', '=', 'contacts.user_id') ...

... ->join('contacts', function ($join) {
        $join->on('users.id', '=', 'contacts.user_id')
             ->where('contacts.user_id', '>', 5);
    }) ...

... ->leftJoin('contacts', 'users.id', '=', 'contacts.user_id') ...
```
```php
//查询字段增加  
$query = DB::table('users')->select('name');
$users = $query->addSelect('age')->get();

//条件语句when闭包的使用 （if lese的另一种体现）
... ->when($sortBy, function ($query) use ($sortBy) {
                        return $query->orderBy($sortBy);
                    }, function ($query) {
                        return $query->orderBy('name');
                    }); ...
```

```php
//修改删除插入
... ->where('id',1)->update(['name' => 'polaris','role_id' => 2])  ...
... ->where('id',1)->delete(); ...
... ->insert([['name' => 'kay'],['name'=>'polaris]]); ...
```

```php
//事务闭包 自动回滚
DB::transaction(function () {
    // ...
});

//手动使用事务
DB::beginTransaction();
//回滚
DB::rollBack();
//提交
DB::commit();
```
```php
//获取监听应用中每次 SQL 语句的执行
DB::listen(function ($query) {
            // $query->sql
            // $query->bindings
            // $query->time
});
```

```php
//执行原生sql 第二个参数为数据的传入
$results = DB::select('select * from users where id = :id', ['id' => 1]);
DB::insert('insert into users (id, name) values (?, ?)', [1, 'lxtx']);
$affected = DB::update('update users set votes = 100 where name = ?', ['lxtx']);
$deleted = DB::delete('delete from users');
```

```php
//执行表级别的语句 如修改删除表(字段)
DB::statement('drop table users');
```
更多用法及详细介绍 
[数据库操作 —— 查询构建器](http://laravelacademy.org/post/8834.html)


#### 内功心法之ORM
看了上诉Lavarel查询构造器的方法后，由衷感叹，只能用多和强大来形同，因为你不再需要写长而烦人的sql,也不需要去做各种判断与字符串拼接。

如果把数据操作当做九阳神功的话，那么上诉构造器就是基本的站桩，俯卧撑及基本的花式的对打了，与普通人对战能轻易取胜，那么如果想要学到九阳神功五重以上，想要上天入地隔空取物的话，就要用到今天的主角Eloquent ORM了。

通常每张数据表都需对应一个与该表进行交互的模型（Model），通过模型类，可以对数据表进行查询、插入、更新、删除等操作。这就是上诉普通构造器招式，有了这些基本功，我就可以往下看。

**华而不实之招式一  save的尴尬**

```php
    /**
     * 创建一个新的航班实例
     *
     * @param  Request  $request
     * @return Response
     */
    public function store(Request $request)
    {
        // 验证请求...

        $flight = new Flight;
        //$flight = Flight::find($request->id);

        $flight->name = $request->name;

        $flight->save();
    }
```
上诉例子中，我们只是简单分配 HTTP 请求中的 ```name``` 参数值给 ```App\Flight``` 模型实例的 ```name``` 属性，当我们调用 ```save``` 方法时，一条记录将会被插入数据库。从而使数据持久化入库。

而```created_at``` 和 ```updated_at``` 时间戳在 ```save``` 方法被调用时会自动被设置，所以没必要手动设置它们。

这样看来，它的'弊端'就直接给体现出来：当数据表需要大量字段（属性）赋值并随带的各种判断，这种写法就变的不那么优雅了，缺代之的便是```create``` 或是```insert```方法的数组操作更为优雅些。
```php
    /**
     * 创建一个新的航班实例
     *
     * @param  Request  $request
     * @return Response
     */
    public function store(Request $request)
    {
        // 验证请求...

        $flight = new Flight;

        $flight->name = $request->name;
        $flight->hid = $request->hid;
        $flight->distance = $request->distance;
        $flight->price = $request->price;
        
        if(false) {
            $flight->pid = $request->pid;
        }
        // ......

        $flight->save();
    }

```

```php
    /**
     * 创建一个新的航班实例
     *
     * @param  Request  $request
     * @return Response
     */
    public function store(Request $request)
    {
        // 验证请求...
        
        $flight = $request->only('a','b');
        //$flight = [];
        //....

        Flight::create($flight);
        //Flight::insert($flight);
    }

```

但它不是没有应用场景，比如在小型项目中的查询附带更新的操作。
```php
    /**
     * 创建一个新的航班实例
     *
     * @param  Request  $request
     * @return Response
     */
    public function store(Request $request)
    {
        // 验证请求...

        $flight = Flight::find($request->id);
        
        //取hid做飞行任务
        $this->flyman($flight->hid);

        $flight->name = $request->name;

        $flight->save();
    }

```

**隔空取物之招式二  with的强大**

总而言之，关系型数据库最擅长的就是处理各种表，字段的关联关系了，这些将会在Model中体现。

最简单的一对一对应关系
```php
namespace App;

use Illuminate\Database\Eloquent\Model;

class User extends Model{
    /**
     * 获取关联到用户的手机
     */
    public function phone()
    {
        return $this->belongsTo('App\Phone','id','phone_id');
    }
}
```
构造器中的体现 结果集中的```Phone```数据将会以```phone```对象的方式体现
```php
    /**
     * 查询用户的手机
     *
     * @return Response
     */
    public function index()
    {
       $users =  \App\User::with('phone')->get();
    }

```

除此之外，还有一对多，多对多，远层一对多，多态关联，多对多的多态关联。 详细用法详见[Eloquent ORM —— 关联关系](http://laravelacademy.org/post/8867.html)

**最强内功之招式三  项目中正确使用model的姿势**

在一个项目中，数据操作可视为最底层操作。数据又具有私密性，高度准确性，有规则性等特点，这个环节可尤为重要。

数据操作本是由业务决定的，假设我们业务操作在```Server```层，入口在```Controller```层，数据操作在Model层，与```Model```及```Server```层交互在```Repositories```层。这样看似复杂，其实更简单，这符合现代化不同工种的的协作，如一个互联网产品发布所需要的工种，以及造汽车流水线。
这样使得各层高度解耦，且具高替代性。就像是我们不会因为一位前端开发的离职而找不到第二位可替代人选。

那么，此时```Model```的任务就变的尤为简单。Model中只需写字段构造，模型关联。或是作用域，以及颗粒度极小的筛选查询等等。而不应该在此做大量的查询供其他层调用。

