# 服务容器

## 1、简介
Laravel 服务容器是一个用于管理类依赖和执行依赖注入的强大工具。依赖注入听上去很花哨，其实质是通过构造函数或者某些情况下通过 set 方法将类依赖注入到类中。

让我们看一个简单的例子：

```
<?php

namespace App\Jobs;

use App\User;
use Illuminate\Contracts\Mail\Mailer;
use Illuminate\Contracts\Bus\SelfHandling;

class PurchasePodcast implements SelfHandling{
    /**
     * 邮件实现
     */
    protected $mailer;

    /**
     * 创建一个新的实例
     *
     * @param  Mailer  $mailer
     * @return void
     */
    public function __construct(Mailer $mailer)
    {
        $this->mailer = $mailer;
    }

    /**
     * 购买一个播客
     *
     * @return void
     */
    public function handle()
    {
        //
    }
}
```

在本例中，当播客被购买后 PurchasePodcast 任务需要发送邮件，因此，你需要注入一个可以发送邮件的服务。由于该服务是被注入的，我们可以方便的使用其另一个实现来替换它，在测试的时候我们还可以”模拟“或创建一个假的邮件实现。

深入理解 Laravel 服务容器对于构建功能强大的大型 Laravel 应用而言至关重要，对于贡献代码到 Laravel 核心也很有帮助。

## 2、绑定
几乎所有的服务容器绑定都是在服务提供者中完成。因此本章节的演示例子用到的容器都是在这种上下文环境中，如果一个类没有基于任何接口那么就没有必要将其绑定到容器。容器并不需要被告知如何构建对象，因为它会使用 PHP 的反射服务自动解析出具体的对象。

在一个服务提供者中，可以通过`$this->app` 变量访问容器，然后使用 bind 方法注册一个绑定，该方法需要两个参数，第一个参数是我们想要注册的类名或接口名称，第二个参数是返回类的实例的闭包：

```
$this->app->bind('HelpSpot\API', function ($app) {
    return new HelpSpot\API($app['HttpClient']);
});
```

注意到我们接受容器本身作为解析器的一个参数，然后我们可以使用该容器来解析我们正在构建的对象的子依赖。

**绑定一个单例**
singleton 方法绑定一个只需要解析一次的类或接口到容器，然后接下来对容器的调用将会返回同一个实例：

```
$this->app->singleton('FooBar', function ($app) {
    return new FooBar($app['SomethingElse']);
});
```

**绑定实例**
你还可以使用 instance 方法绑定一个已存在的对象实例到容器，随后对容器的调用将总是返回给定的实例：

```
$fooBar = new FooBar(new SomethingElse);

$this->app->instance('FooBar', $fooBar);
```

### 2.1 绑定接口到实现
服务容器的一个非常强大的特性是其绑定接口到实现的能力。我们假设有一个 EventPusher 接口及其 RedisEventPusher 实现，编写完该接口的 RedisEventPusher 实现后，就可以将其注册到服务容器：

```
$this->app->bind('App\Contracts\EventPusher', 'App\Services\RedisEventPusher');
```

这段代码告诉容器当一个类需要 EventPusher 的实现时将会注入 RedisEventPusher，现在我们可以在构造器或者任何其它通过服务容器注入依赖的地方进行 EventPusher 接口的类型提示：

```
use App\Contracts\EventPusher;

/**
 * 创建一个新的类实例
 *
 * @param  EventPusher  $pusher
 * @return void
 */
public function __construct(EventPusher $pusher){
    $this->pusher = $pusher;
}
```

## 2.2 上下文绑定
有时侯我们可能有两个类使用同一个接口，但我们希望在每个类中注入不同实现，例如，当系统接到一个新的订单的时候，我们想要通过 PubNub 而不是 Pusher 发送一个事件。Laravel 定义了一个简单、平滑的方式来定义这种行为：

```
$this->app->when('App\Handlers\Commands\CreateOrderHandler')
          ->needs('App\Contracts\EventPusher')
          ->give('App\Services\PubNubEventPusher');
你甚至还可以传递一个闭包到 give 方法：
$this->app->when('App\Handlers\Commands\CreateOrderHandler')
          ->needs('App\Contracts\EventPusher')
          ->give(function () {
                  // Resolve dependency...
              });
```

## 2.3 标签
少数情况下我们需要解析特定分类下的所有绑定，比如，也许你正在构建一个接收多个不同 Report 接口实现的报告聚合器，在注册完 Report 实现之后，可以通过 tag 方法给它们分配一个标签：

```
$this->app->bind('SpeedReport', function () {
    //
});

$this->app->bind('MemoryReport', function () {
    //
});

$this->app->tag(['SpeedReport', 'MemoryReport'], 'reports');
```

这些服务被打上标签后，可以通过 tagged 方法来轻松解析它们：

```
$this->app->bind('ReportAggregator', function ($app) {
    return new ReportAggregator($app->tagged('reports'));
});
```

# 3、解析
有很多方式可以从容器中解析对象，首先，你可以使用 make 方法，该方法接收你想要解析的类名或接口名作为参数：

```
$fooBar = $this->app->make('FooBar');
```

其次，你可以以数组方式访问容器，因为其实现了 PHP 的 ArrayAccess 接口：

```
$fooBar = $this->app['FooBar'];
```

最后，也是最常用的，你可以简单的通过在类的构造函数中对依赖进行类型提示来从容器中解析对象，包括控制器、事件监听器、队列任务、中间件等都是通过这种方式。在实践中，这是大多数对象从容器中解析的方式。
容器会自动为其解析类注入依赖，比如，你可以在控制器的构造函数中为应用定义的仓库进行类型提示，该仓库会自动解析并注入该类：

```
<?php

namespace App\Http\Controllers;

use Illuminate\Routing\Controller;
use App\Users\Repository as UserRepository;

class UserController extends Controller{
    /**
     * 用户仓库实例
     */
    protected $users;

    /**
     * 创建一个控制器实例
     *
     * @param  UserRepository  $users
     * @return void
     */
    public function __construct(UserRepository $users)
    {
        $this->users = $users;
    }

    /**
     * 通过指定 ID 显示用户
     *
     * @param  int  $id
     * @return Response
     */
    public function show($id)
    {
        //
    }
}
```

# 4、容器事件
服务容器在每一次解析对象时都会触发一个事件，可以使用 resolving 方法监听该事件：

```
$this->app->resolving(function ($object, $app) {
    // 容器解析所有类型对象时调用
});

$this->app->resolving(function (FooBar $fooBar, $app) {
    // 容器解析“FooBar”对象时调用
});
```

正如你所看到的，被解析的对象将会传递给回调，从而允许你在对象被传递给消费者之前为其设置额外属性。