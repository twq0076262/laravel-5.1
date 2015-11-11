# HTTP 路由

## 1、基本路由
大部分路由都定义在被 `App\Providers\RouteServiceProvider` 类载入的 `app/Http/routes.php` 文件中。
最基本的 Laravel 路由接收一个 URI 和一个闭包：

```
Route::get('/', function () {
    return 'Hello World';
});

Route::post('foo/bar', function () {
    return 'Hello World';
});

Route::put('foo/bar', function () {
    //
});

Route::delete('foo/bar', function () {
    //
});
```

**为多个动作注册路由**
有时候需要注册一个路由来响应多个不同的 HTTP 动作，你可以使用 `Route` 门面的 `match` 方法来实现：

```
Route::match(['get', 'post'], '/', function () {
    return 'Hello World';
});
```

或者，还可以使用 `any` 方法注册一个路由响应所有 HTTP 动作：

```
Route::any('foo', function () {
    return 'Hello World';
});
```

**生成路由对应的 URLs**
可以使用帮助函数 `url` 来生成路由对应的 URLs：

```
$url = url('foo');
```

## 2、路由参数

### 2.1 必选参数

有时我们需要在路由中捕获 URI 片段，比如，如果想要从 URL 中捕获用户 ID，可以通过如下方式定义路由参数：

```
Route::get('user/{id}', function ($id) {
    return 'User '.$id;
});
```

可以按需要定义在路由中定义多个路由参数：

```
Route::get('posts/{post}/comments/{comment}', function ($postId, $commentId) {
    //
});
```

路由参数总是通过花括号进行包裹，参数在路由被执行时会被传递到路由的闭包。
注意：路由参数不能包含’-‘字符，需要的话可以使用_替代。

### 2.2 可选参数

有时候可能需要指定路由参数，并且使得该路由参数是可选的，可以通过在参数名后加一个？来标记：

```
Route::get('user/{name?}', function ($name = null) {
    return $name;
});

Route::get('user/{name?}', function ($name = 'John') {
    return $name;
});
```

### 2.3 正则约束

可以使用路由实例上的 `where` 方法来约束路由参数的格式。`where` 方法接收参数名和一个正则表达式来定义该参数如何被约束：

```
Route::get('user/{name}', function ($name) {
    //
})->where('name', '[A-Za-z]+');

Route::get('user/{id}', function ($id) {
    //
})->where('id', '[0-9]+');

Route::get('user/{id}/{name}', function ($id, $name) {
    //
})->where(['id' => '[0-9]+', 'name' => '[a-z]+']);
```

#### 2.3.1 全局约束

如果想要路由参数在全局范围内被给定正则表达式约束，可以使用 `pattern` 方法。可以在 `RouteServiceProvider` 类的 `boot` 方法中定义约束模式：

```
/**
 * 定义路由模型绑定，模式过滤器等
 *
 * @param  \Illuminate\Routing\Router  $router
 * @return void
 * @translator  http://laravelacademy.org
 */
public function boot(Router $router){
    $router->pattern('id', '[0-9]+');
    parent::boot($router);
}
```

一旦模式被定义，将会自动应用到所有包含该参数名的路由中。
扩展阅读：[实例教程——HTTP 路由实例教程（一）—— 基本使用及路由参数](http://laravelacademy.org/post/398.html)

## 3、命名路由
命名路由使生成 URLs 或者重定向到指定路由变得很方便，在定义路由时指定路由名称，然后使用数组键 `as` 指定路由别名：

```
Route::get('user/profile', ['as' => 'profile', function () {
    //
}]);
```

还可以为控制器动作指定路由名称：

```
Route::get('user/profile', [
    'as' => 'profile', 'uses' => 'UserController@showProfile'
]);
```

### 3.1 路由分组 & 命名路由
如果你在使用路由分组，可以在路由分组属性数组中指定 as 关键字来为分组中的路由设置一个共用的路由名前缀：

```
Route::group(['as' => 'admin::'], function () {
    Route::get('dashboard', ['as' => 'dashboard', function () {
        // 路由被命名为 "admin::dashboard"
    }]);
});
```

### 3.2 为命名路由生成 URLs
一旦你为给定路由分配了名字，通过 route 函数生成 URLs 时就可以使用路由名字：

```
$url = route('profile');
$redirect = redirect()->route('profile');
```

如果路由定义了参数，可以将路由参数作为第二个参数传递给 `route` 函数。给定的路由参数将会自动插入 URL 中：

```
Route::get('user/{id}/profile', ['as' => 'profile', function ($id) {
    //
}]);
$url = route('profile', ['id' => 1]);
```

## 4、路由分组
路由分组允许我们在多个路由中共享路由属性，比如中间件和命名空间等，这样的话一大波共享属性的路由就不必再各自定义这些属性。共享属性以数组的形式被作为第一个参数传递到 Route::group 方法中。
想要了解更多路由分组，我们希望通过几个简单的应用实例来展示其特性。

### 4.1 中间件
要分配中间件给分组中的所有路由，可以在分组属性数组中使用 `middleware` 键。中间件将会按照数组中定义的顺序依次执行：

```
Route::group(['middleware' => 'auth'], function () {
    Route::get('/', function ()    {
        // 使用 Auth 中间件
    });

    Route::get('user/profile', function () {
        // 使用 Auth 中间件
    });
});
```

### 4.2 命名空间
另一个通用的例子是路由分组分配同一个 PHP 命名空间给多个控制器，可以在分组属性数组中使用 `namespace` 参数来指定分组中控制器的命名空间：

```
Route::group(['namespace' => 'Admin'], function(){
    // 控制器在 "App\Http\Controllers\Admin" 命名空间下

    Route::group(['namespace' => 'User'], function()
    {
        // 控制器在 "App\Http\Controllers\Admin\User" 命名空间下
    });
});
```

默认情况下，`RouteServiceProvider` 包含 `routes.php` 并指定其所在命名空间，因此，我们只需要指定命名空间的 `App\Http\Controllers` 之后的一部分。

### 4.3 子域名路由
路由分组还可以被用于子域名路由通配符，子域名可以像 URIs 一样被分配给路由参数，从而允许捕获子域名的部分用于路由或者控制器，子域名可以通过分组属性数组中的 `domain` 键来指定：

```
Route::group(['domain' => '{account}.myapp.com'], function () {
    Route::get('user/{id}', function ($account, $id) {
        //
    });
});
```

### 4.4 路由前缀
属性 `prefix` 可以用来为分组中每个给定 URI 添加一个前缀，比如，你想要为所有路由 URIs 前面添加前缀 `admin`：

```
Route::group(['prefix' => 'admin'], function () {
    Route::get('users', function ()    {
        // 匹配 "/admin/users" URL
    });
});
```

你还可以使用 `prefix` 参数为分组路由指定公共参数：

```
Route::group(['prefix' => 'accounts/{account_id}'], function () {
    Route::get('detail', function ($account_id)    {
        // 匹配 accounts/{account_id}/detail URL
    });
});
```

扩展阅读：[实例教程——HTTP 路由实例教程（二）—— 路由命名和路由分组](http://laravelacademy.org/post/417.html)

## 5、防止 CSRF 攻击

### 5.1 简介
Laravel 使得防止应用遭到跨站请求伪造攻击变得简单。跨站请求伪造是一种通过伪装授权用户的请求来利用授信网站的恶意漏洞。
Laravel 自动为每一个被应用管理的有效用户 Session 生成一个 CSRF“令牌”，该令牌用于验证授权用户和发起请求者是否是同一个人。想要生成包含 CSRF 令牌的隐藏输入字段，可以使用帮助函数 `csrf_field` 来实现：

```
<?php echo csrf_field(); ?>
```

帮助函数 `csrf_field` 生成如下 HTML：

```
<input type="hidden" name="_token" value="<?php echo csrf_token(); ?>">
```

当然还可以使用 Blade 模板引擎提供的方式：

```
{!! csrf_field() !!}
```

你不需要了解在 POST、PUT 或者 DELETE 请求时 CSRF 令牌是如何进行验证的，HTTP 中间件`VerifyCsrfToken` 会为我们做这项工作：将请求中输入的 `token` 值和 session 中的存储的作对比。

### 5.2 从 CSRF 保护中排除 URIs
有时候我们想要从 CSRF 保护中排除一些 URIs，比如，如果你在使用 Stripe 来处理支付并用到他们的 webhook 系统，这时候你就需要从 Laravel 的 CSRF 保护中排除 webhook 处理器路由。
你可以通过在 `VerifyCsrfToken` 中间件中将要排除的 URIs 添加到
`$except`属性：


```
<?php

namespace App\Http\Middleware;

use Illuminate\Foundation\Http\Middleware\VerifyCsrfToken as BaseVerifier;

class VerifyCsrfToken extends BaseVerifier
{
    /**
     *从 CSRF 验证中排除的 URL
     *
     * @var array
     */
    protected $except = [
        'stripe/*',
    ];
}
```

### 5.3 X-CSRF-Token
除了将 CSRF 令牌作为一个 POST 参数进行检查，Laravel 的 `VerifyCsrfToken` 中间件还会检查 `X-CSRF-TOKEN` 请求头，你可以将令牌保存在”meta”标签中：

```
<meta name="csrf-token" content="{{ csrf_token() }}">
```

创建完这个 meta 标签后，就可以在 js 库如 jQuery 中添加该令牌到所有请求头，这为基于 AJAX 的应用提供了简单、方便的方式来避免 CSRF 攻击：

```
$.ajaxSetup({
        headers: {
            'X-CSRF-TOKEN': $('meta[name="csrf-token"]').attr('content')
        }
});
```

### 5.4 X-XSRF-Token
Laravel 还将 CSRF 令牌保存到了名为 `XSRF-TOKEN` 的 cookie 中，你可以使用该 cookie 值来设置 `X-XSRF-TOKEN` 请求头。一些 JavaScript 框架，比如 Angular，将会为你自动进行设置，基本上你不太会手动设置这个值。

扩展阅读：[实例教程——HTTP 路由实例教程（三）—— CSRF 攻击原理及其防护](http://laravelacademy.org/post/525.html)

## 6、路由模型绑定
Laravel 路由模型绑定为注入类实例到路由提供了方便，例如，你可以将匹配给定 ID 的整个 `User` 类实例注入到路由中，而不是直接注入用户 ID。

首先，使用路由的 `model` 方法为给定参数指定一个类，你应该在 `RouteServiceProvider::boot` 方法中定义模型绑定：

### 绑定参数到模型

```
public function boot(Router $router)
{
    parent::boot($router);
    $router->model('user', 'App\User');
}
```

接下来，定义一个包含`{user}`参数的路由：

```
$router->get('profile/{user}', function(App\User $user) {
    //
});
```

由于我们已经绑定了`{user}`参数到 `App\User` 模型，一个 `User` 实例将会被注入到路由中。也就是说，如果请求 URL 是 `profile/1`，那么相应的将会注入 ID 为 1 的 `User` 实例到路由中。
注：如果在匹配模型实例的时候在数据库中找不到对应记录，那么就会自动抛出404异常。

如果你想要指定自己的“not found”行为，可以传递一个闭包作为第三个参数到 `model` 方法：

```
$router->model('user', 'App\User', function() {
    throw new NotFoundHttpException;
});
```

如果你想要使用自己的路由模型绑定解决方案，应该使用 `Route::bind` 方法，这样的话传递到 `bind` 方法的闭包将会接受 URI 中的参数值，然后返回你想要注入到路由的类实例：

```
$router->bind('user', function($value) {
    return App\User::where('name', $value)->first();
});
```

## 7、表单方法伪造
HTML 表单不支持 `PUT`、`PATCH` 或者 `DELETE` 动作，因此，当定义被 HTML 表单调用的 `PUT`、`PATCH` 或 `DELETE` 路由时，需要添加一个隐藏的`_method` 字段到给表单中，其值被用作 HTTP 请求方法名：

```
<form action="/foo/bar" method="POST">
    <input type="hidden" name="_method" value="PUT">
    <input type="hidden" name="_token" value="{{ csrf_token() }}">
</form>
```

## 8、抛出 404 错误
有两者方法手动从路由触发 404 错误。
第一种，使用帮助函数 `abort`，`abort` 函数会抛出一个指定状态码的 `Symfony\Component\HttpFoundation\Exception\HttpException`：

```
abort(404);
```

第二种，手动抛出 `Symfony\Component\HttpKernel\Exception\NotFoundHttpException.`的实例。

更多关于处理 404 异常的信息以及如何自定义视图显示这些错误信息，请查看错误文档一节。