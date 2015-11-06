# 视图

# 1、基本使用
视图包含服务于应用的 HTML 并将应用的控制器逻辑和表现逻辑进行分离。视图文件存放在 resources/views 目录。
下面是一个简单视图：

```
<!-- 该视图存放 resources/views/greeting.php -->

<html>
    <body>
        <h1>Hello, <?php echo $name; ?></h1>
    </body>
</html>
```

这个视图存放在 resources/views/greeting.php，我们可以在全局的帮助函数 view 中这样返回它：

```
Route::get('/', function ()    {
    return view('greeting', ['name' => 'James']);
});
```

传递给 view 方法的第一个参数是 resources/views 目录下相应的视图文件的名字，第二个参数是一个数组，该数组包含了在该视图中所有有效的数据。在这个例子中，我们传递了一个 name 变量，在视图中通过执行 echo 将其显示出来。
当然，视图还可以嵌套在 resources/views 的子目录中，用“.”号来引用嵌套视图，比如，如果视图存放路径是 resources/views/admin/profile.php，那我们可以这样引用它：

```
return view('admin.profile', $data);
```

**判断视图是否存在**
如果需要判断视图是否存在，可调用不带参数的 view 之后再使用 exists 方法，如果视图在磁盘存在则返回 true：

```
if (view()->exists('emails.customer')) {
    //
}
```

调用不带参数的 view 时，将会返回一个 Illuminate\Contracts\View\Factory 实例，从而可以调用该工厂的所有方法。
## 1.1 视图数据
### 1.1.1 传递数据到视图
在上述例子中可以看到，我们可以简单通过数组方式将数据传递到视图：

```
return view('greetings', ['name' => 'Victoria']);
```

以这种方式传递数据的话，$data 应该是一个键值对数组，在视图中，就可以使用相应的键来访问数据值，比如<?php echo $key; ?>。除此之外，还可以通过 with 方法添加独立的数据片段到视图：

```
$view = view('greeting')->with('name', 'Victoria');
```

### 1.1.2 在视图间共享数据
有时候我们需要在所有视图之间共享数据片段，这时候可以使用视图工厂的 share 方法，通常，需要在服务提供者的 boot 方法中调用 share 方法，你可以将其添加到 AppServiceProvider 或生成独立的服务提供者来存放它们：

```
<?php

namespace App\Providers;

class AppServiceProvider extends ServiceProvider
{
    /**
     * 启动所有应用服务
     *
     * @return void
     */
    public function boot()
    {
        view()->share('key', 'value');
    }

    /**
     * 注册服务提供者
     *
     * @return void
     */
    public function register()
    {
        //
    }
}
```

# 2、视图 Composer
视图 Composers 是当视图被渲染时的回调或类方法。如果你有一些数据要在视图每次渲染时都做绑定，可以使用视图 composer 将逻辑组织到一个单独的地方。
首先要在服务提供者中注册视图 Composer，我们将会使用帮助函数 view 来访问 Illuminate\Contracts\View\Factory 的底层实现，记住，Laravel 不会包含默认的视图 Composers 目录，我们可以按照自己的喜好组织其位置，例如可以创建一个 App\Http\ViewComposers 目录：

```
<?php

namespace App\Providers;

use Illuminate\Support\ServiceProvider;

class ComposerServiceProvider extends ServiceProvider
{
    /**
     * 在容器中注册绑定.
     *
     * @return void
     * @author http://laravelacademy.org
     */
    public function boot()
    {
        // 使用基于类的 composers...
        view()->composer(
            'profile', 'App\Http\ViewComposers\ProfileComposer'
        );

        // 使用基于闭包的 composers...
        view()->composer('dashboard', function ($view) {

        });
    }

    /**
     * 注册服务提供者.
     *
     * @return void
     */
    public function register()
    {
        //
    }
}
```

如果创建一个新的服务提供者来包含视图 composer 注册，需要添加该服务提供者到配置文件 config/app.php 的 providers 数组中。
现在我们已经注册了 composer，每次 profile 视图被渲染时都会执行 ProfileComposer@compose，接下来我们来定义该 composer 类：

```
<?php

namespace App\Http\ViewComposers;

use Illuminate\Contracts\View\View;
use Illuminate\Users\Repository as UserRepository;

class ProfileComposer
{
    /**
     * 用户仓库实现.
     *
     * @var UserRepository
     */
    protected $users;

    /**
     * 创建一个新的属性 composer.
     *
     * @param  UserRepository  $users
     * @return void
     */
    public function __construct(UserRepository $users)
    {
        // Dependencies automatically resolved by service container...
        $this->users = $users;
    }

    /**
     * 绑定数据到视图.
     *
     * @param  View  $view
     * @return void
     */
    public function compose(View $view)
    {
        $view->with('count', $this->users->count());
    }
}
```

视图被渲染前，composer 的 compose 方法被调用，同时 Illuminate\Contracts\View\View 被注入，可使用其 with 方法来绑定数据到视图。
注意：所有视图 composers 都通过服务容器被解析，所以你可以在 composer 的构造函数中声明任何你需要的依赖。
**添加 Composer 到多个视图**
你可以传递视图数组作为 composer 方法的第一个参数来一次性将视图 composer 添加到多个视图：

```
view()->composer(
    ['profile', 'dashboard'],
    'App\Http\ViewComposers\MyViewComposer'
);
```

composer 方法接受*通配符，从而允许将一个 composer 添加到所有视图：

```
view()->composer('*', function ($view) {
    //
});
```

## 2.1 视图创建器
视图创建器和视图 composer 非常类似，不同之处在于前者在视图实例化之后立即失效而不是等到视图即将渲染。使用 create 方法注册一个视图创建器：

```
view()->creator('profile', 'App\Http\ViewCreators\ProfileCreator');
```