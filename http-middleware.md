# HTTP 中间件

## 1、简介

HTTP 中间件提供了一个便利的机制来过滤进入应用的 HTTP 请求。例如，Laravel 包含了一个中间件来验证用户是否经过授权，如果用户没有经过授权，中间件会将用户重定向到登录页面，否则如果用户经过授权，中间件就会允许请求继续往前进入下一步操作。

当然，除了认证之外，中间件还可以被用来处理更多其它任务。比如：CORS 中间件可以用于为离开站点的响应添加合适的头（跨域）；日志中间件可以记录所有进入站点的请求。

Laravel 框架内置了一些中间件，包括维护模式中间件、认证、CSRF 保护中间件等等。所有的中间件都位于 app/Http/Middleware 目录。

## 2、定义中间件

想要创建一个新的中间件，可以通过 Artisan 命令 make:middleware：

```
php artisan make:middleware OldMiddleware
```

这个命令会在 app/Http/Middleware 目录下创建一个新的中间件类 OldMiddleware，在这个中间件中，我们只允许提供的 age 大于 200 的访问路由，否则，我们将用户重定向到主页：

```
<?php

namespace App\Http\Middleware;

use Closure;

class OldMiddleware
{
    /**
     * 返回请求过滤器
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  \Closure  $next
     * @return mixed
     */
    public function handle($request, Closure $next)
    {
        if ($request->input('age') <= 200) {
            return redirect('home');
        }

        return $next($request);
    }

}
```

正如你所看到的，如果 age<=200，中间件会返回一个 HTTP 重定向到客户端；否则，请求会被传递下去。将请求往下传递可以通过调用回调函数 `$next`。

理解中间件的最好方式就是将中间件看做 HTTP 请求到达目标之前必须经过的“层”，每一层都会检查请求甚至会完全拒绝它。

### 2.1 中间件之前/之后

一个中间件是否请求前还是请求后执行取决于中间件本身。比如，以下中间件会在请求处理前执行一些任务：

```
<?php

namespace App\Http\Middleware;

use Closure;

class BeforeMiddleware
{
    public function handle($request, Closure $next)
    {
        // 执行动作

        return $next($request);
    }
}
```

然而，下面这个中间件则会在请求处理后执行其任务：

```
<?php

namespace App\Http\Middleware;

use Closure;

class AfterMiddleware
{
    public function handle($request, Closure $next)
    {
        $response = $next($request);

        // 执行动作

        return $response;
    }
}
```

## 3、注册中间件

### 3.1 全局中间件

如果你想要中间件在每一个 HTTP 请求期间被执行，只需要将相应的中间件类放到 app/Http/Kernel.php 的数组属性`$middleware` 中即可。

### 3.2 分配中间件到路由

如果你想要分配中间件到指定路由，首先应该在 app/Http/Kernel.php 文件中分配给该中间件一个简写的 key，默认情况下，该类的`$routeMiddleware` 属性包含了 Laravel 内置的入口中间件，添加你自己的中间件只需要将其追加到后面并为其分配一个 key：

```
// 在 App\Http\Kernel 里中
protected $routeMiddleware = [
    'auth' => \App\Http\Middleware\Authenticate::class,
    'auth.basic' => \Illuminate\Auth\Middleware\AuthenticateWithBasicAuth::class,
    'guest' => \App\Http\Middleware\RedirectIfAuthenticated::class,];
```

中间件在 HTTP kernel 中被定义后，可以在路由选项数组中使用`$middleware` 键来指定中间件：

```
Route::get('admin/profile', ['middleware' => 'auth', function () {
    //
}]);
```

## 4、中间件参数

中间件还可以接收额外的自定义参数，比如，如果应用需要在执行动作之前验证认证用户是否拥有指定的角色，可以创建一个 RoleMiddleware 来接收角色名作为额外参数。
额外的中间件参数会在`$next` 参数之后传入中间件：

```
<?php

namespace App\Http\Middleware;

use Closure;

class RoleMiddleware
{
    /**
     * 运行请求过滤器
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  \Closure  $next
     * @param  string  $role
     * @return mixed
     * translator http://laravelacademy.org
     */
    public function handle($request, Closure $next, $role)
    {
        if (! $request->user()->hasRole($role)) {
            // Redirect...
        }

        return $next($request);
    }

}
```

中间件参数可以在定义路由时通过：分隔中间件名和参数名来指定，多个中间件参数可以通过逗号分隔：

```
Route::put('post/{id}', ['middleware' => 'role:editor', function ($id) {
    //
}]);
```

## 5、可终止的中间件
有时候中间件可能需要在 HTTP 响应发送到浏览器之后做一些工作。比如，Laravel 内置的“session”中间件会在响应发送到浏览器之后将 session 数据写到存储器中，为了实现这个，定义一个可终止的中间件并添加 terminate 方法到这个中间件：


```
<?php

namespace Illuminate\Session\Middleware;

use Closure;

class StartSession
{
    public function handle($request, Closure $next)
    {
        return $next($request);
    }

    public function terminate($request, $response)
    {
        // 存储 session 数据...
    }
}
```

terminate 方法将会接收请求和响应作为参数。一旦你定义了一个可终止的中间件，应该将其加入到 HTTP kernel 的全局中间件列表中。
当调用中间件上的 terminate 方法时，Laravel 将会从服务容器中取出该中间件的新的实例，如果你想要在调用 handle 和 terminate 方法时使用同一个中间件实例，则需要使用容器的 singleton 方法将该中间件注册到容器中。