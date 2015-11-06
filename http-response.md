# HTTP 响应

# 1、基本响应
所有路由和控制器都会返回某种被发送到用户浏览器的响应，Laravel 提供了多种不同的方式来返回响应，最基本的响应就是从路由或控制器返回一个简单的字符串：

```
Route::get('/', function () {
    return 'Hello World';
});
```

给定的字符串会被框架自动转化为 HTTP 响应。
但是大多数路由和控制器动作都会返回一个完整的 Illuminate\Http\Response 实例或视图，返回一个完整的 Response 实例允许你自定义响应的 HTTP 状态码和头信息，Response 实例继承自 Symfony\Component\HttpFoundation\Response 类，该类提供了一系列方法用于创建 HTTP 响应：

```
use Illuminate\Http\Response;

Route::get('home', function () {
    return (new Response($content, $status))
                  ->header('Content-Type', $value);
});
```

为方便起见，还可以使用帮助函数 response：

```
Route::get('home', function () {
    return response($content, $status)
                  ->header('Content-Type', $value);});
```

注意：查看完整的 Response 方法列表，请移步相应的 API 文档以及 Symfony API 文档
## 1.1 添加响应头
大部分响应方法都是可以链式调用的，从而使得可以平滑的构建响应。例如，可以使用 header 方法来添加一系列响应头：

```
return response($content)
            ->header('Content-Type', $type)
            ->header('X-Header-One', 'Header Value')
            ->header('X-Header-Two', 'Header Value');
```

## 1.2 添加 Cookies
使用 response 实例的帮助函数 withCookie 可以轻松添加 cookie 到响应，比如，可以使用 withCookie 方法来生成 cookie 并将其添加到 response 实例：

```
return response($content)->header('Content-Type', $type)
                 ->withCookie('name', 'value');
```

withCookie 方法接收额外的可选参数从而允许对 cookie 属性更多的自定义：

```
->withCookie($name, $value, $minutes, $path, $domain, $secure, $httpOnly)
```

默认情况下，Laravel 框架生成的 cookies 经过加密和签名，所以在客户端不能进行修改，如果你想要将特定的 cookies 子集在生成时取消加密，可以使用中间件 App\Http\Middleware\EncryptCookies 的$except 属性：

```
/**
 * 需要被加密的 cookies 名称
 *
 * @var array
 */
protected $except = [
    'cookie_name',
];
```

# 2、其它响应类型
帮助函数 response 可以用来方便地生成其他类型的响应实例，当无参数调用 response 时会返回 Illuminate\Contracts\Routing\ResponseFactory 契约的一个实现，该契约提供了一些有用的方法来生成响应。
## 2.1 视图响应
如果你需要控制响应状态和响应头，还需要返回一个视图作为响应内容，可以使用 view 方法：

```
return response()->view('hello', $data)->header('Content-Type', $type);
```

当然，如果你不需要传递一个自定义的 HTTP 状态码或者自定义头，只需要简单使用全局的帮助函数 view 即可。
## 2.2 JSON 响应
json 方法会自动将 Content-Type 头设置为 application/json，并使用 PHP 函数 json_encode 方法将给定数组转化为 JSON：

```
return response()->json(['name' => 'Abigail', 'state' => 'CA']);
```

如果你想要创建一个 JSONP 响应，可是添加 setCallback 到 json 方法后面：

```
return response()->json(['name' => 'Abigail', 'state' => 'CA'])
                 ->setCallback($request->input('callback'));
```

## 2.3 文件下载
download 方法用于生成强制用户浏览器下载给定路径文件的响应。download 方法接受文件名作为第二个参数，该参数决定用户下载文件的显示名称，你还可以将 HTTP 头信息作为第三个参数传递到该方法：

```
return response()->download($pathToFile);
return response()->download($pathToFile, $name, $headers);
```

注意：管理文件下载的 Symfony HttpFoundation 类要求被下载文件有一个 ASCII 文件名。
# 3、重定向
重定向响应是 Illuminate\Http\RedirectResponse 类的实例，其中包含了必须的头信息将用户重定向到另一个 URL，有很多方式来生成 RedirectResponse 实例，最简单的方法就是使用全局帮助函数 redirect：

```
Route::get('dashboard', function () {
    return redirect('home/dashboard');
});
```

有时候你想要将用户重定向到前一个位置，比如，表单提交后，验证不通过，你就可以使用 back 帮助函数返回前一个 URL：

```
Route::post('user/profile', function () {
    // 验证请求...
    return back()->withInput();
});
```

## 3.1 重定向到命名路由
如果调用不带参数的 redirect 方法，会返回一个 Illuminate\Routing\Redirector 实例，从而可以调用该实例上的任何方法。比如，为了生成一个 RedirectResponse 到命名路由，可以使用 route 方法：

```
return redirect()->route('login');
```

如果路由中有参数，可以将其作为第二个参数传递到 route 方法：

```
// For a route with the following URI: profile/{id}
return redirect()->route('profile', [1]);
```

如果要重定向到带 ID 参数的路由，并从 Eloquent 模型中取数据填充表单，可以传递模型本身，ID 会被自动解析出来：

```
return redirect()->route('profile', [$user]);
```

## 3.2 重定向到控制器动作
你还可以生成重定向到控制器动作，只需简单传递控制器和动作名到 action 方法即可。记住，你不需要指定控制器的完整命名空间，因为 Laravel 的 RouteServiceProvider 将会自动设置默认的控制器命名空间：

```
return redirect()->action('HomeController@index');
```

当然，如果控制器路由要求参数，你可以将参数作为第二个参数传递给 action 方法：

```
return redirect()->action('UserController@profile', [1]);
```

## 3.3 带一次性Session数据的重定向
重定向到一个新的 URL 并将数据存储到一次性 session中通常是同时完成的，为了方便，可以创建一个 RedirectResponse 实例然后在同一个方法链上将数据存储到 session，这种方式在 action 之后存储状态信息时特别方便：

```
Route::post('user/profile', function () {
    // 更新用户属性...
    return redirect('dashboard')->with('status', 'Profile updated!');
});
```

当然，用户重定向到新页面之后，你可以从 session 中取出并显示这些一次性信息，比如，使用Blade 语法实现如下：

```
@if (session('status'))
    <div class="alert alert-success">
        {{ session('status') }}
    </div>
@endif
```

# 4、响应宏
如果你想要定义一个自定义的响应并且在多个路由和控制器中复用，可以使用 Illuminate\Contracts\Routing\ResponseFactory 实现上的 macro 方法。
比如，在一个服务提供者的 boot 方法中：

```
<?php

namespace App\Providers;

use Illuminate\Support\ServiceProvider;
use Illuminate\Contracts\Routing\ResponseFactory;

class ResponseMacroServiceProvider extends ServiceProvider
{
    /**
     * Perform post-registration booting of services.
     *
     * @param  ResponseFactory  $factory
     * @return void
     */
    public function boot(ResponseFactory $factory)
    {
        $factory->macro('caps', function ($value) use ($factory) {
            return $factory->make(strtoupper($value));
        });
    }
}
```

micro 方法接收响应名称作为第一个参数，一个闭包函数作为第二个参数，micro 的闭包在从 ResponseFactory 实现或帮助函数 response 上调用 macro 名称的时候被执行：

```
return response()->caps('foo');
```