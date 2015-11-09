# Session

## 1、简介
由于 HTTP 驱动的应用是无状态的，所以我们使用 session 来存储用户请求信息。Laravel 通过干净、统一的 API 处理后端各种有效 session 驱动，目前支持的流行后端驱动包括 Memcached、Redis 和数据库。

### 1.1 配置
Session 配置文件位于 config/session.php。默认情况下，Laravel 使用的 session 驱动为文件驱动，这对许多应用而言是没有什么问题的。在生产环境中，你可能考虑使用 memcached 或者 redis 驱动以便获取更快的 session 性能。

session 驱动定义请求的 session 数据存放在哪里，Laravel 可以处理多种类型的驱动：

- 	file – session 数据存储在 storage/framework/sessions 目录下； 
- 	cookie – session 数据存储在经过加密的安全的 cookie 中； 
- 	database – session 数据存储在数据库中 
- 	memcached / redis – session 数据存储在 memcached/redis 中； 
- 	array – session 数据存储在简单 PHP 数组中，在多个请求之间是非持久化的。

注意：数组驱动通常用于运行测试以避免 session 数据持久化。

### 1.2 Session 驱动预备知识

#### 1.2.1 数据库
当使用 databasesession 驱动时，需要设置表包含 session 项，下面是该数据表的表结构声明：

```
Schema::create('sessions', function ($table) {
    $table->string('id')->unique();
    $table->text('payload');
    $table->integer('last_activity');
});
```

你可以使用 Artisan 命令 session:table 来生成迁移：

```
php artisan session:table
composer dump-autoload
php artisan migrate
```

#### 1.2.2 Redis
在 Laravel 中使用 Redis session 驱动前，需要通过 Composer 安装 predis/predis 包。

### 1.3 其它 Session 相关问题
Laravel 框架内部使用 flash session 键，所以你不应该通过该名称添加数据项到 session。
如果你需要所有存储的 session 数据经过加密，在配置文件中设置 encrypt 配置为 true。

## 2、基本使用
**访问 session**
首先，我们来访问 session，我们可以通过 HTTP 请求访问 session 实例，可以在控制器方法中通过类型提示引入请求实例，记住，控制器方法依赖通过 Laravel服务容器注入：

```
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use App\Http\Controllers\Controller;

class UserController extends Controller{
    /**
     * 显示指定用户的属性
     *
     * @param  Request  $request
     * @param  int  $id
     * @return Response
     */
    public function showProfile(Request $request, $id)
    {
        $value = $request->session()->get('key');

        //
    }
}
```

从 session 中获取数据的时候，还可以传递默认值作为第二个参数到 get 方法，默认值在指定键在 session 中不存在时返回。如果你传递一个闭包作为默认值到 get 方法，该闭包会执行并返回执行结果：

```
$value = $request->session()->get('key', 'default');

$value = $request->session()->get('key', function() {
    return 'default';
});
```

如果你想要从 session 中获取所有数据，可以使用 all 方法：

```
$data = $request->session()->all();
```

还可以使用全局的 PHP 函数 session 来获取和存储 session 中的数据：

```
Route::get('home', function () {
    // 从 session 中获取数据...
    $value = session('key');

    // 存储数据到 session...
    session(['key' => 'value']);
});
```

**判断 session 中是否存在指定项**
has 方法可用于检查数据项在 session 中是否存在。如果存在的话返回 true：

```
if ($request->session()->has('users')) {
    //
}
```

**在 session 中存储数据**
获取到 session 实例后，就可以调用多个方法来与底层数据进行交互，例如，put 方法存储新的数据到 session 中：

```
$request->session()->put('key', 'value');
```

**推送数据到数组 session**
push 方法可用于推送数据到值为数组的 session，例如，如果 user.teams 键包含团队名数组，可以像这样推送新值到该数组：

```
$request->session()->push('user.teams', 'developers');
```

**获取并删除数据**
pull 方法将会从 session 获取并删除数据：

```
$value = $request->session()->pull('key', 'default');
```

从 session 中删除数据项
forget 方法从 session 中移除指定数据，如果你想要从 session 中移除所有数据，可以使用 flush 方法：

```
$request->session()->forget('key');
$request->session()->flush();
```

**重新生成 Session ID**
如果你需要重新生成 session ID，可以使用 regenerate 方法：

```
$request->session()->regenerate();
```

### 2.1 一次性数据
有时候你可能想要在 session 中存储只在下个请求中有效的数据，可以通过 flash 方法来实现。使用该方法存储的 session 数据只在随后的 HTTP 请求中有效，然后将会被删除：

```
$request->session()->flash('status', 'Task was successful!');
```

如果你需要在更多请求中保持该一次性数据，可以使用 reflash 方法，该方法将所有一次性数据保留到下一个请求，如果你只是想要保存特定一次性数据，可以使用 keep 方法：

```
$request->session()->reflash();
$request->session()->keep(['username', 'email']);
```

## 3、添加自定义Session 驱动
要为 Laravel 后端 session 添加更多驱动，可以使用 Session 门面上的 extend 方法。可以在服务提供者的 boot 方法中调用该方法：

```
<?php

namespace App\Providers;

use Session;
use App\Extensions\MongoSessionStore;
use Illuminate\Support\ServiceProvider;

class SessionServiceProvider extends ServiceProvider{
    /**
     * Perform post-registration booting of services.
     *
     * @return void
     */
    public function boot()
    {
        Session::extend('mongo', function($app) {
            // Return implementation of SessionHandlerInterface...
            return new MongoSessionStore;
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

需要注意的是自定义 session 驱动需要实现 SessionHandlerInterface 接口，该接口包含少许我们需要实现的方法，一个 MongoDB 的实现如下：

```
<?php

namespace App\Extensions;

class MongoHandler implements SessionHandlerInterface{
    public function open($savePath, $sessionName) {}
    public function close() {}
    public function read($sessionId) {}
    public function write($sessionId, $data) {}
    public function destroy($sessionId) {}
    public function gc($lifetime) {}
}
```

由于这些方法并不像缓存的 StoreInterface 接口方法那样容易理解，我们接下来快速过一遍每一个方法：

- 	 open 方法用于基于文件的 session 存储系统，由于 Laravel 已经有了一个 file session 驱动，所以在该方法中不需要放置任何代码，可以将其置为空方法。 
- 	close 方法和 open 方法一样，也可以被忽略，对大多数驱动而言都用不到该方法。 
- 	read 方法应该返回与给定$sessionId 相匹配的 session 数据的字符串版本，从驱动中获取或存储 session 数据不需要做任何序列化或其它编码，因为 Laravel 已经为我们做了序列化。 
- 	write 方法应该讲给定$data 写到持久化存储系统相应的$sessionId , 例如 MongoDB, Dynamo 等等。 
- 	destroy 方法从持久化存储中移除 $sessionId 对应的数据。 
- 	gc 方法销毁大于给定$lifetime 的所有 session 数据，对本身拥有过期机制的系统如 Memcached 和 Redis 而言，该方法可以留空。
session 驱动被注册之后，就可以在配置文件 config/session.php 中使用 mongo 驱动了。