# 事件

## 1、简介
Laravel 事件提供了简单的观察者模式实现，允许你订阅和监听应用中的事件。事件类通常存放在 app/Events 目录，监听器存放在 app/Listeners。

## 2、注册事件/监听器
Laravel 自带的 EventServiceProvider 为事件注册提供了方便之所。其中的 listen 属性包含了事件（键）和对应监听器（值）数组。如果应用需要，你可以添加多个事件到该数组。例如，让我们添加 PodcastWasPurchased 事件：

```
/**
 * 事件监听器映射
 *
 * @var array
 */
protected $listen = [
    'App\Events\PodcastWasPurchased' => [
        'App\Listeners\EmailPurchaseConfirmation',
    ],
];
```

### 2.1 生成事件/监听器类
当然，手动为每个事件和监听器创建文件是很笨重的，取而代之地，我们可见简单添加监听器和事件到 EventServiceProvider 然后使用 event:generate 命令。该命令将会生成罗列在 EventServiceProvider 中的所有事件和监听器。当然，已存在的事件和监听器不会被创建：

```
php artisan event:generate
```

## 3、定义事件
事件类是一个处理与事件相关的简单数据容器，例如，假设我们生成的 PodcastWasPurchased 事件接收一个 Eloquent ORM 对象：

```
<?php

namespace App\Events;

use App\Podcast;
use App\Events\Event;
use Illuminate\Queue\SerializesModels;

class PodcastWasPurchased extends Event{
    use SerializesModels;

    public $podcast;

    /**
     * 创建新的事件实例
     *
     * @param  Podcast  $podcast
     * @return void
     */
    public function __construct(Podcast $podcast)
    {
        $this->podcast = $podcast;
    }
}
```

正如你所看到的，该事件类不包含任何特定逻辑，只是一个存放被购买的 Podcast 对象的容器，如果事件对象被序列化的话，事件使用的 SerializesModels trait 将会使用 PHP 的 serialize 函数序列化所有 Eloquent 模型。

## 4、定义监听器
接下来，让我们看看我们的示例事件的监听器，事件监听器在 handle 方法中接收事件实例，event:generate 命令将会自动在 handle 方法中导入合适的事件类和类型提示事件。在 handle 方法内，你可以执行任何需要的逻辑以响应事件。

```
<?php

namespace App\Listeners;

use App\Events\PodcastWasPurchased;
use Illuminate\Queue\InteractsWithQueue;
use Illuminate\Contracts\Queue\ShouldQueue;

class EmailPurchaseConfirmation{
    /**
     * 创建事件监听器
     *
     * @return void
     */
    public function __construct()
    {
        //
    }

    /**
     * 处理事件
     *
     * @param  PodcastWasPurchased  $event
     * @return void
     */
    public function handle(PodcastWasPurchased $event)
    {
        // Access the podcast using $event->podcast...
    }
}
```

你的事件监听器还可以在构造器中类型提示任何需要的依赖，所有事件监听器通过服务容器解析，所以依赖会自动注入：

```
use Illuminate\Contracts\Mail\Mailer;

public function __construct(Mailer $mailer){
    $this->mailer = $mailer;
}
```

**停止事件继续往下传播**
有时候，你希望停止事件被传播到其它监听器，你可以通过从监听器的 handle 方法中返回 false 来实现。

### 4.1 事件监听器队列
需要将事件监听器放到队列中？没有比这更简单的了，只需要让监听器类实现 ShouldQueue 接口即可，通过 Artisan 命令 event:generate 生成的监听器类已经将接口导入当前命名空间，所有你可以立即拿来使用：

```
<?php

namespace App\Listeners;

use App\Events\PodcastWasPurchased;
use Illuminate\Queue\InteractsWithQueue;
use Illuminate\Contracts\Queue\ShouldQueue;

class EmailPurchaseConfirmation implements ShouldQueue{
    //
}
```

就是这么简单，当监听器被事件调用，将会使用 Laravel 的队列系统通过队列分发器自动队列化。如果通过队列执行监听器的时候没有抛出任何异常，队列任务在执行完成后被自动删除。

#### 4.1.1 手动访问队列
如果你需要手动访问底层队列任务的 delete 和 release 方法，在生成的监听器中默认导入的 Illuminate\Queue\InteractsWithQueue trait 提供了访问这两个方法的权限：

```
<?php

namespace App\Listeners;

use App\Events\PodcastWasPurchased;
use Illuminate\Queue\InteractsWithQueue;
use Illuminate\Contracts\Queue\ShouldQueue;

class EmailPurchaseConfirmation implements ShouldQueue{
    use InteractsWithQueue;

    public function handle(PodcastWasPurchased $event)
    {
        if (true) {
            $this->release(30);
        }
    }
}
```

## 5、触发事件
要触发一个事件，可以使用 Event 门面，传递一个事件实例到 fire 方法，fire 方法会分发事件到所有监听器：

```
<?php

namespace App\Http\Controllers;

use Event;
use App\Podcast;
use App\Events\PodcastWasPurchased;
use App\Http\Controllers\Controller;

class UserController extends Controller{
    /**
     * 显示指定用户属性
     *
     * @param  int  $userId
     * @param  int  $podcastId
     * @return Response
     */
    public function purchasePodcast($userId, $podcastId)
    {
        $podcast = Podcast::findOrFail($podcastId);

        // Purchase podcast logic...

        Event::fire(new PodcastWasPurchased($podcast));
    }
}
```

此外，你还可以使用全局的帮助函数 event 来触发事件：

```
event(new PodcastWasPurchased($podcast));
```

## 6、广播事件
在很多现代 web 应用中，web 套接字被用于实现实时更新的用户接口。当一些数据在服务器上被更新，通常一条消息通过 websocket 连接被发送给客户端处理。

为帮助你构建这样的应用，Laravel 让通过 websocket 连接广播事件变得简单。广播 Laravel 事件允许你在服务端和客户端 JavaScript 框架之间共享同一事件名。

### 6.1 配置
所有的事件广播配置选项都存放在 config/broadcasting.php 配置文件中。Laravel 支持多种广播驱动：Pusher、Redis以及一个服务于本地开发和调试的日志驱动。每一个驱动都有一个配置示例。

#### 6.1.1 广播预备知识
事件广播需要以下两个依赖：
- 	Pusher: pusher/pusher-php-server ~2.0 
- 	Redis: predis/predis ~1.0
-  
#### 6.1.2 队列预备知识
在开始介绍广播事件之前，还需要配置并运行一个队列监听器。所有事件广播都通过队列任务来完成以便应用的响应时间不受影响。

### 6.2 将事件标记为广播
要告诉 Laravel 给定事件应该被广播，需要在事件类上实现 Illuminate\Contracts\Broadcasting\ShouldBroadcast 接口。ShouldBroadcast 接口要求你实现一个方法：broadcastOn。该方法应该返回事件广播”频道“名称数组：

```
<?php

namespace App\Events;

use App\User;
use App\Events\Event;
use Illuminate\Queue\SerializesModels;
use Illuminate\Contracts\Broadcasting\ShouldBroadcast;

class ServerCreated extends Event implements ShouldBroadcast{
    use SerializesModels;

    public $user;

    /**
     * 创建新的事件实例
     *
     * @return void
     */
    public function __construct(User $user)
    {
        $this->user = $user;
    }

    /**
     * 获取事件广播频道
     *
     * @return array
     */
    public function broadcastOn()
    {
        return ['user.'.$this->user->id];
    }
}
```

然后，你只需要和正常一样触发该事件，事件被触发后，一个队列任务将通过指定广播驱动自动广播该事件。

### 6.3 广播数据
如果某个事件被广播，其所有的 public 属性都会按照事件负载自动序列化和广播，从而允许你从 JavaScript 中访问所有 public 数据，因此，举个例子，如果你的事件有一个单独的包含 Eloquent 模型的`$user` 属性，广播负载定义如下：

```
{
    "user": {
        "id": 1,
        "name": "Jonathan Banks"
        ...
    }
}
```

然而，如果你希望对广播负载有更加细粒度的控制，可以添加 broadcastWith 方法到事件，该方法应该返回你想要通过事件广播的数组数据：

```
/**
 * 获取广播数据
 *
 * @return array
 */
public function broadcastWith(){
    return ['user' => $this->user->id];
}
```

### 6.4 消费事件广播

#### 6.4.1 Pusher
你可以通过 Pusher 的 JavaScript SDK 方便地使用Pusher驱动消费事件广播。例如，让我们从之前的例子中消费 App\Events\ServerCreated 事件：

```
this.pusher = new Pusher('pusher-key');

this.pusherChannel = this.pusher.subscribe('user.' + USER_ID);

this.pusherChannel.bind('App\\Events\\ServerCreated', function(message) {
    console.log(message.user);
});
```

#### 6.4.2 Redis
如果你在使用 Redis 广播，你将需要编写自己的 Redis pub/sub 消费者来接收消息并使用自己选择的 websocket 技术将其进行广播。例如，你可以选择使用使用 Node 编写的流行的Socket.io库。
使用 Node 库 socket.io 和 ioredis，你可以快速编写事件广播发布所有广播事件：

```
var app = require('http').createServer(handler);
var io = require('socket.io')(app);

var Redis = require('ioredis');
var redis = new Redis();

app.listen(6001, function() {
    console.log('Server is running!');});

function handler(req, res) {
    res.writeHead(200);
    res.end('');}

io.on('connection', function(socket) {
    //
});

redis.psubscribe('*', function(err, count) {
    //
});

redis.on('pmessage', function(subscribed, channel, message) {
    message = JSON.parse(message);
    io.emit(channel + ':' + message.event, message.data);
});
```

## 7、事件订阅者
事件订阅者是指那些在类本身中订阅到多个事件的类，从而允许你在单个类中定义一些事件处理器。订阅者应该定义一个 subscribe 方法，该方法中传入一个事件分发器实例：

```
<?php

namespace App\Listeners;

class UserEventListener{
    /**
     * 处理用户登录事件
     */
    public function onUserLogin($event) {}

    /**
     * 处理用户退出事件
     */
    public function onUserLogout($event) {}

    /**
     * 为订阅者注册监听器
     *
     * @param  Illuminate\Events\Dispatcher  $events
     * @return array
     */
    public function subscribe($events)
    {
        $events->listen(
            'App\Events\UserLoggedIn',
            'App\Listeners\UserEventListener@onUserLogin'
        );

        $events->listen(
            'App\Events\UserLoggedOut',
            'App\Listeners\UserEventListener@onUserLogout'
        );
    }

}
```

### 7.1 注册一个事件订阅者
订阅者被定义后，可以通过事件分发器进行注册，你可以使用 EventServiceProvider 上的`$subcribe` 属性来注册订阅者。例如，让我们添加 UserEventListener：

```
<?php

namespace App\Providers;

use Illuminate\Contracts\Events\Dispatcher as DispatcherContract;
use Illuminate\Foundation\Support\Providers\EventServiceProvider as ServiceProvider;

class EventServiceProvider extends ServiceProvider{
    /**
     * 事件监听器映射数组
     *
     * @var array
     */
    protected $listen = [
        //
    ];

    /**
     * 要注册的订阅者
     *
     * @var array
     */
    protected $subscribe = [
        'App\Listeners\UserEventListener',
    ];
}
```