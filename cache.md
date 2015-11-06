# 缓存

# 1、配置
Laravel 为不同的缓存系统提供了统一的 API。缓存配置位于 config/cache.php。在该文件中你可以指定在应用中默认使用哪个缓存驱动。Laravel 目前支持流行的缓存后端如 Memcached 和 Redis 等。
缓存配置文件还包含其他文档化的选项，确保仔细阅读这些选项。默认情况下，Laravel 被配置成使用文件缓存，这会将序列化数据和缓存对象存储到文件系统。对大型应用，建议使用内存缓存如 Memcached 或 APC，你甚至可以为同一驱动配置多个缓存配置。
## 1.1 缓存预备知识
### 1.1.1 数据库
使用 database 缓存驱动时，你需要设置一张表包含缓存缓存项。下面是该表的 Schema 声明：

```
Schema::create('cache', function($table) {
    $table->string('key')->unique();
    $table->text('value');
    $table->integer('expiration');
});
```

### 1.1.2 Memcached
使用 Memcached 缓存要求安装了Memcached PECL 包，即 PHP Memcached 扩展。
Memcached::addServer默认配置使用 TCP/IP 协议：

```
'memcached' => [
    [
        'host' => '127.0.0.1',
        'port' => 11211,
        'weight' => 100
    ],
],
```

你还可以设置 host 选项为 UNIX socket 路径，如果你这样做，port 选项应该置为 0：

```
'memcached' => [
    [
        'host' => '/var/run/memcached/memcached.sock',
        'port' => 0,
        'weight' => 100
    ],
],
```

### 1.1.3 Redis
使用 Laravel 的 Redis 缓存之前，你需要通过 Composer 安装 predis/predis 包（~1.0）。
了解更多关于 Redis 的配置，查看Larave 的 Redis 文档。
# 2、缓存使用
## 2.1 获取缓存实例
Illuminate\Contracts\Cache\Factory 和 Illuminate\Contracts\Cache\Repository契约提供了访问 Laravel 的缓存服务的方法。Factory 契约提供了所有访问应用定义的缓存驱动的方法。Repository 契约通常是应用中 cache 配置文件中指定的默认缓存驱动的一个实现。
然而，你还可以使用 Cache门面，这也是我们在整个文档中使用的方式，Cache 门面提供了简单方便的方式对底层 Laravel 缓存契约实现进行访问。
例如，让我们在控制器中导入 Cache 门面：

```
<?php

namespace App\Http\Controllers;

use Cache;
use Illuminate\Routing\Controller;

class UserController extends Controller{
    /**
     * 显示应用所有用户列表
     *
     * @return Response
     */
    public function index()
    {
        $value = Cache::get('key');

        //
    }
}
```

### 2.1.1 访问多个缓存存储
使用 Cache 门面，你可以使用 store 方法访问不同的缓存存储器，传入 store 方法的键就是 cache 配置文件中 stores 配置数组里列出的相应的存储器：

```
$value = Cache::store('file')->get('foo');
Cache::store('redis')->put('bar', 'baz', 10);
```

## 2.2 从缓存中获取数据
Cache 门面的 get 方法用于从缓存中获取缓存项，如果缓存项不存在，返回 null。如果需要的话你可以传递第二个参数到 get 方法指定缓存项不存在时返回的自定义默认值：

```
$value = Cache::get('key');
$value = Cache::get('key', 'default');
```

你甚至可以传递一个闭包作为默认值，如果缓存项不存在的话闭包的结果将会被返回。传递闭包允许你可以从数据库或其它外部服务获取默认值：

```
$value = Cache::get('key', function() {
    return DB::table(...)->get();
});
```

### 2.2.1 检查缓存项是否存在
has 方法用于判断缓存项是否存在：

```
if (Cache::has('key')) {
    //
}
```

### 2.2.2 数值增加/减少
increment 和 decrement 方法可用于调整缓存中的整型数值。这两个方法都可以接收第二个参数来指明缓存项数值增加和减少的数目：

```
Cache::increment('key');
Cache::increment('key', $amount);

Cache::decrement('key');
Cache::decrement('key', $amount);
```

### 2.2.3 获取或更新
有时候你可能想要获取缓存项，但如果请求的缓存项不存在时给它存储一个默认值。例如，你可能想要从缓存中获取所有用户，或者如果它们不存在的话，从数据库获取它们并将其添加到缓存中，你可以通过使用 Cache::remember 方法实现：

```
$value = Cache::remember('users', $minutes, function() {
    return DB::table('users')->get();});
```

如果缓存项不存在，传递给 remember 方法的闭包被执行并且将结果存放到缓存中。
你还可以联合 remember 和 forever 方法：

```
$value = Cache::rememberForever('users', function() {
    return DB::table('users')->get();});
```

### 2.2.4 获取并删除
如果你需要从缓存中获取缓存项然后删除，你可以使用 pull 方法，和 get 方法一样，如果缓存项不存在的话返回 null：

```
$value = Cache::pull('key');
```

## 2.3 存储缓存项到缓存
你可以使用 Cache 门面上的 put 方法在缓存中存储缓存项。当你在缓存中存储缓存项的时候，你需要指定数据被缓存的时间（分钟数）：

```
Cache::put('key', 'value', $minutes);
```

除了传递缓存项失效时间，你还可以传递一个代表缓存项有效时间的 PHP Datetime 实例：

```
$expiresAt = Carbon::now()->addMinutes(10);
Cache::put('key', 'value', $expiresAt);
```

add 方法只会在缓存项不存在的情况下添加缓存项到缓存，如果缓存项被添加到缓存返回 true，否则，返回 false：

```
Cache::add('key', 'value', $minutes);
```

forever 方法用于持久化存储缓存项到缓存，这些值必须通过 forget 方法手动从缓存中移除：

```
Cache::forever('key', 'value');
```

## 2.4 从缓存中移除数据
你可以使用 Cache 门面上的 forget 方法从缓存中移除缓存项：

```
Cache::forget('key');
```

# 3、添加自定义缓存驱动
要使用自定义驱动扩展 Laravel 缓存，我们使用 Cache 门面的 extend 方法，该方法用于绑定定义驱动解析器到管理器，通常，这可以在服务提供者中完成。
例如，要注册一个新的命名为“mongo”的缓存驱动：

```
<?php

namespace App\Providers;

use Cache;
use App\Extensions\MongoStore;
use Illuminate\Support\ServiceProvider;

class CacheServiceProvider extends ServiceProvider{
    /**
     * Perform post-registration booting of services.
     *
     * @return void
     */
    public function boot()
    {
        Cache::extend('mongo', function($app) {
            return Cache::repository(new MongoStore);
        });
    }

    /**
     * Register bindings in the container.
     *
     * @return void
     */
    public function register()
    {
        //
    }
}
```

传递给 extend 方法的第一个参数是驱动名称。该值对应配置文件 config/cache.php 中的 driver 选项。第二个参数是返回 Illuminate\Cache\Repository 实例的闭包。该闭包中被传入一个$app 实例，也就是服务容器的一个实例。
调用 Cache::extend 可以在默认 App\Providers\AppServiceProvider 中的 boot 方法中完成，或者你也可以创建自己的服务提供者来存放该扩展——只是不要忘了在配置文件 config/app.php 中注册该提供者。
要创建自定义的缓存驱动，首先需要实现 Illuminate\Contracts\Cache\Store 契约，所以，我们的 MongoDB 缓存实现看起来像这样子：

```
<?php

namespace App\Extensions;

class MongoStore implements \Illuminate\Contracts\Cache\Store{
    public function get($key) {}
    public function put($key, $value, $minutes) {}
    public function increment($key, $value = 1) {}
    public function decrement($key, $value = 1) {}
    public function forever($key, $value) {}
    public function forget($key) {}
    public function flush() {}
    public function getPrefix() {}
}
```

我们只需要使用 MongoDB 连接实现每一个方法，实现完成后，我们可以完成自定义驱动注册：

```
Cache::extend('mongo', function($app) {
    return Cache::repository(new MongoStore);
});
```

扩展完成后，只需要更新配置文件 config/cache.php 的 driver 选项为你的扩展名称。
如果你在担心将自定义缓存驱动代码放到哪，考虑将其放到 Packgist！或者，你可以在 app 目录下创建一个 Extensions 命名空间。然而，记住 Laravel 并没有一个严格的应用目录结构，你可以基于你的需要自由的组织目录结构。
# 4、缓存标签
注意：缓存标签不支持 file 或 database 缓存驱动，此外，在“永久”存储缓存中使用多个标签时，memcached 之类的驱动有着最佳性能，因为它可以自动清除过期的记录。
## 4.1 存储打上标签的缓存项
缓存标签允许你给相关的缓存项打上同一个标签，然后可以输出被分配同一个标签的所有缓存值。你可以通过传递一个有序的标签名数组来访问被打上标签的缓存。例如，让我们访问一个被打上标签的缓存并将其值放到缓存中：

```
Cache::tags(['people', 'artists'])->put('John', $john, $minutes);
Cache::tags(['people', 'authors'])->put('Anne', $anne, $minutes);
```

然而，并不只限于使用 put 方法，你可以在处理标签时使用任何混存存储器提供的方法。
## 4.2 访问打上标签的缓存项
要获取被打上标签的缓存项，传递同样的有序标签名数组到 tags 方法：

```
$john = Cache::tags(['people', 'artists'])->get('John');
$anne = Cache::tags(['people', 'authors'])->get('Anne');
```

通过上面的语句你可以输出所有分配了该标签或标签列表的缓存项，例如，下面这个语句将会移除被打上 people，authors 标签的缓存，或者，Anne 和 John 都会从缓存中移除：

```
Cache::tags(['people', 'authors'])->flush();
```

相比之下，下面这个语句只会移除被打上 authors 标签的缓存，所以 John 会被移除，而 Anne 不会：

```
Cache::tags('authors')->flush();
```

# 5、缓存事件
要在每次缓存操作时执行代码，你可以监听缓存触发的事件，通常，你可以将这些缓存处理器代码放到 EventServiceProvider 的 boot 方法中：

```
/**
 * 注册应用任意其他事件
 *
 * @param  \Illuminate\Contracts\Events\Dispatcher  $events
 * @return void
 */
public function boot(DispatcherContract $events){
    parent::boot($events);

    $events->listen('cache.hit', function ($key, $value) {
        //
    });

    $events->listen('cache.missed', function ($key) {
        //
    });

    $events->listen('cache.write', function ($key, $value, $minutes) {
        //
    });

    $events->listen('cache.delete', function ($key) {
        //
    });
}
```