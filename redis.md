# Redis

## 1、简介
Redis 是一个开源的、高级的键值对存储系统，经常被用作数据结构服务器，因为其支持字符串、Hash、列表、集合和有序集合等数据结构。在 Laravel 中使用 Redis 之前，需要通过 Composer 安装 predis/predis 包（~1.0）。

### 1.1 配置
应用的 Redis 配置位于配置文件 config/database.php。在这个文件中，可以看到包含被应用使用的 Redis 服务器的 redis 数组：

```
'redis' => [

    'cluster' => false,

    'default' => [
        'host'     => '127.0.0.1',
        'port'     => 6379,
        'database' => 0,
    ],

],
```

默认服务器配置可以满足开发需要，然而，你可以基于环境随意修改该数组，只需要给每个 Redis 服务器一个名字并指定该 Redis 服务器使用的主机和接口。

cluster 选项告诉 Laravel Redis 客户端在多个 Redis 节点间执行客户端分片，从而形成节点池并创建大量有效的 RAM。然而，客户端分片并不处理故障转移，所以，非常适合从另一个主数据存储那里获取有效的缓存数据。

此外，你可以在 Redis 连接定义中定义 options 数组值，从而允许你指定一系列 Predis 客户端选项。

如果 Redis 服务器要求认证信息，你可以通过添加 password 配置项到 Redis 服务器配置数组来提供密码。

注意：如果你通过 PECL 安装 PHP 的 Redis 扩展，需要在 config/app.php 文件中修改 Redis 的别名。

## 2、基本使用
你可以通过调用 Redis门面上的多个方法来与 Redis 进行交互，该门面支持动态方法，所以你可以任何Redis 命令，该命令将会直接传递给 Redis，在本例中，我们通过调用 Redis 门面上的 get 方法来调用 Redis 上的 GET 命令：

```
<?php

namespace App\Http\Controllers;

use Redis;use App\Http\Controllers\Controller;

class UserController extends Controller{
    /**
     * 显示指定用户属性
     *
     * @param  int  $id
     * @return Response
     */
    public function showProfile($id)
    {
        $user = Redis::get('user:profile:'.$id);
        return view('user.profile', ['user' => $user]);
    }
}
```

当然，如上所述，可以在 Redis 门面上调用任何 Redis 命令。Laravel 使用魔术方法将命令传递给 Redis 服务器，所以只需简单传递参数和 Redis 命令如下：

```
Redis::set('name', 'Taylor');
$values = Redis::lrange('names', 5, 10);
```

此外还可以使用 command 方法传递命令到服务器，该方法接收命令名作为第一个参数，参数值数组作为第二个参数：

```
$values = Redis::command('lrange', ['name', 5, 10]);
```

**使用多个 Redis 连接**
你可以通过调用 Redis::connection 方法获取 Redis 实例：

```
$redis = Redis::connection();
```

这将会获取默认 Redis 服务器实例，如果你没有使用服务器集群，可以传递服务器名到 connection 方法来获取指定 Redis 配置中定义的指定服务器：

```
$redis = Redis::connection('other');
```

### 2.1 管道命令
当你需要在一次操作中发送多个命令到服务器的时候应该使用管道，pipeline 方法接收一个参数：接收 Redis 实例的闭包。你可以将所有 Redis 命令发送到这个 Redis 实例，然后这些命令会在一次操作中被执行：

```
Redis::pipeline(function ($pipe) {
    for ($i = 0; $i < 1000; $i++) {
        $pipe->set("key:$i", $i);
    }
});
```

## 3、发布/订阅
Redis 还提供了调用 Redis 的 publish 和 subscribe 命令的接口。这些 Redis 命令允许你在给定“频道”监听消息，你可以从另外一个应用发布消息到这个频道，甚至使用其它编程语言，从而允许你在不同的应用/进程之间轻松通信。

首先，让我们使用 subscribe 方法通过 Redis 在一个频道上设置监听器。由于调用 subscribe 方法会开启一个常驻进程，我们将在Artisan 命令中调用该方法：

```
<?php

namespace App\Console\Commands;

use Redis;
use Illuminate\Console\Command;

class RedisSubscribe extends Command{
    /**
     * 控制台命令名称
     *
     * @var string
     */
    protected $signature = 'redis:subscribe';

    /**
     * 控制台命令描述
     *
     * @var string
     */
    protected $description = 'Subscribe to a Redis channel';

    /**
     * 执行控制台命令
     *
     * @return mixed
     */
    public function handle()
    {
        Redis::subscribe(['test-channel'], function($message) {
            echo $message;
        });
    }
}
```

现在，我们可以使用 publish 发布消息到该频道：

```
Route::get('publish', function () {
    // 路由逻辑...
    Redis::publish('test-channel', json_encode(['foo' => 'bar']));
});
```

### 3.1 通配符订阅
使用 psubscribe 方法，你可以订阅到一个通配符定义的频道，这在所有相应频道上获取所有消息时很有用。`$channel` 名将会作为第二个参数传递给提供的回调闭包：

```
Redis::psubscribe(['*'], function($message, $channel) {
    echo $message;
});

Redis::psubscribe(['users.*'], function($message, $channel) {
    echo $message;
});
```