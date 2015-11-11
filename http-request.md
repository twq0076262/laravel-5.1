# HTTP 请求

## 1、访问请求
通过依赖注入获取当前 HTTP 请求实例，应该在控制器的构造函数或方法中对 `Illuminate\Http\Request` 类进行类型提示，当前请求实例会被服务容器自动注入：

```
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use Illuminate\Routing\Controller;

class UserController extends Controller
{
    /**
     * 存储新用户
     *
     * @param  Request  $request
     * @return Response
     */
    public function store(Request $request)
    {
        $name=$request->input('name'); 
        
        //
    }
}
```

如果你的控制器方法还期望获取路由参数输入，只需要将路由参数置于其它依赖之后即可，例如，如果你的路由定义如下：

```
Route::put('user/{id}','UserController@update');
```

你仍然可以对 `Illuminate\Http\Request` 进行类型提示并通过如下方式定义控制器方法来访问路由参数：

```
<?php
    
namespace App\Http\Controllers;
    
use Illuminate\Http\Request;
use Illuminate\Routing\Controller;

classUser Controller extends Controller
{
    /**
     * 更新指定用户
     *
     * @param  Request  $request
     * @param  int  $id
     * @return Response
     */
     public function update(Request $request,$id)
    { 
        //
    }
}
```

### 1.1 基本请求信息
`Illuminate\Http\Request` 实例提供了多个方法来检测应用的 HTTP 请求，Laravel 的 `Illuminate\Http\Request` 继承自 `Symfony\Component\HttpFoundation\Request` 类，这里列出了一些该类中的有用方法：

#### 1.1.1 获取请求 URI
path 方法将会返回请求的 URI，因此，如果进入的请求路径是 `http://domain.com/foo/bar`，则 `path` 方法将会返回 `foo/bar`：

```
$uri=$request->path();
```

`is` 方法允许你验证进入的请求是否与给定模式匹配。使用该方法时可以使用`*`通配符：

```
if($request->is('admin/*')){ 
    //
}
```

想要获取完整的 URL，而不仅仅是路径信息，可以使用请求实例中的 `url` 方法：

```
$url=$request->url();
```

#### 1.1.2 获取请求方法
`method` 方法将会返回请求的 HTTP 请求方式。你还可以使用 `isMethod` 方法来验证 HTTP 请求方式是否匹配给定字符串：

```
$method=$request->method();
if($request->isMethod('post')){ 
    //
}
```

### 1.2 PSR-7 请求
PSR-7 标准指定了 HTTP 消息接口，包括请求和响应。如果你想要获取 PSR-7 请求实例，首先需要安装一些库，Laravel 使用 Symfony HTTP Message Bridge 组件将典型的 Laravel 请求和响应转化为 PSR-7 兼容的实现：

```
composer require symfony/psr-http-message-bridge
composer require zendframework/zend-diactoros
```

安装完这些库之后，你只需要在路由或控制器中通过对请求类型进行类型提示就可以获取 PSR-7 请求：

```
use Psr\Http\Message\ServerRequestInterface;

Route::get('/', function (ServerRequestInterface $request) {
    //
});
```

如果从路由或控制器返回的是 PSR-7 响应实例，则其将会自动转化为 Laravel 响应实例并显示出来。

## 2、获取输入
**获取输入值**
使用一些简单的方法，就可以从 `Illuminate\Http\Request` 实例中访问用户输入。你不需要担心请求所使用的 HTTP 请求方法，因为对所有请求方式的输入访问接口都是一致的：

```
$name = $request->input('name');
```

你还可以传递一个默认值作为第二个参数给 `input` 方法，如果请求输入值在当前请求未出现时该值将会被返回：

```
$name = $request->input('name', 'Sally');
```

处理表单数组输入时，可以使用”.”来访问数组：

```
$input = $request->input('products.0.name');
```

**判断输入值是否出现**
判断值是否在请求中出现，可以使用 `has` 方法，如果值出现过了且不为空，`has` 方法返回 `true`：

```
if ($request->has('name')) {
    //
}
```

**获取所有输入数据**
你还可以通过 `all` 方法获取所有输入数据：

```
$input = $request->all();
```

**获取输入的部分数据**
如果你需要取出输入数据的子集，可以使用 `only` 或 `except` 方法，这两个方法都接收一个数组作为唯一参数：

```
$input = $request->only('username', 'password');
$input = $request->except('credit_card');
```

### 2.1 上一次请求输入
Laravel 允许你在两次请求之间保存输入数据，这个特性在检测校验数据失败后需要重新填充表单数据时很有用，但如果你使用的是 Laravel 内置的验证服务，则不需要手动使用这些方法，因为一些 Laravel 内置的校验设置会自动调用它们。

#### 2.1.1 将输入存储到一次性 Session
`Illuminate\Http\Request` 实例的 `flash` 方法会将当前输入存放到一次性 session（所谓的一次性指的是从 session 中取出数据中，对应数据会从 session 中销毁）中，这样在下一次请求时数据依然有效：

```
$request->flash();
```

你还可以使用 `flashOnly` 和 `flashExcept` 方法将输入数据子集存放到 session 中：

```
$request->flashOnly('username', 'email');
$request->flashExcept('password');
```

#### 2.1.2 将输入存储到一次性 Session 然后重定向
如果你经常需要一次性存储输入并重定向到前一页，可以简单使用 `withInput` 方法来将输入数据链接到 `redirect` 后面：

```
return redirect('form')->withInput();
return redirect('form')->withInput($request->except('password'));
```

#### 2.1.3 取出上次请求数据
要从 session 中取出上次请求的输入数据，可以使用 Request 实例的 `old` 方法。`old` 方法提供了便利的方式从 session 中取出一次性数据：

```
$username = $request->old('username');
```

Laravel 还提供了一个全局的帮助函数 `old`，如果你是在 Blade 模板中显示老数据，使用帮助函数 `old` 更方便：

```
{{ old('username') }}
```

### 2.2 Cookies

#### 2.2.1 从请求中取出 Cookies
Laravel 框架创建的所有 cookies 都经过加密并使用一个认证码进行签名，这意味着如果客户端修改了它们则需要对其进行有效性验证。我们使用 `Illuminate\Http\Request` 实例的 cookie 方法从请求中获取 cookie 的值：

```
$value = $request->cookie('name');
```

#### 2.2.2 新增 Cookie
Laravel 提供了一个全局的帮助函数 cookie 作为一个简单工厂来生成新的 Symfony\Component\HttpFoundation\Cookie 实例，新增的 cookies 通过 `withCookie` 方法被附加到 `Illuminate\Http\Response` 实例：

```
$response = new Illuminate\Http\Response('Hello World');
$response->withCookie(cookie('name', 'value', $minutes));
return $response;
```

想要创建一个长期有效的 cookie，可以使用 cookie 工厂的 forever 方法：

```
$response->withCookie(cookie()->forever('name', 'value'));
```

### 2.3 文件上传
#### 2.3.1 获取上传的文件
可以使用 `Illuminate\Http\Request` 实例的 `file` 方法来访问上传文件，该方法返回的对象是 `Symfony\Component\HttpFoundation\File\UploadedFile` 类的一个实例，该类继承自 PHP 标准库中提供与文件交互方法的 `SplFileInfo` 类：

```
$file = $request->file('photo');
```

#### 2.3.2 验证文件是否存在
使用 hasFile 方法判断文件在请求中是否存在：

```
if ($request->hasFile('photo')) {
    //
}
```

#### 2.3.3 验证文件是否上传成功
使用 isValid 方法判断文件在上传过程中是否出错：

```
if ($request->file('photo')->isValid()){
    //
}
```

#### 2.3.4 保存上传的文件
使用 `move` 方法将上传文件保存到新的路径，该方法将上传文件从临时目录（在 PHP 配置文件中配置）移动到指定新目录：

```
$request->file('photo')->move($destinationPath);
$request->file('photo')->move($destinationPath, $fileName);
```

#### 2.3.5 其它文件方法
`UploadedFile` 实例中很有很多其它方法，查看该类的 API 了解更多相关方法。