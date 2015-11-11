# 错误&日志

## 1、简介
开始一个 `Laravel `项目的时候，错误和异常处理已经默认为你配置好了。此外，`Laravel `还集成了提供各种功能强大日志处理器的 `Monolog `日志库。

## 2、配置

### 2.1 错误详情显示
配置文件 `config/app.php `中的 `debug `配置选项控制浏览器显示的错误详情数量。默认情况下，该配置选项被设置在`.env `文件中的环境变量 `APP_DEBUG`。

对本地开发而言，你应该设置环境变量 `APP_DEBUG `值为 `true`。在生产环境，该值应该被设置为 `false`。

### 2.2 日志模式
`Laravel` 支持日志方法 `single`, `daily`, `syslog `和 `errorlog`。例如，如果你想要日志文件按日生成而不是生成单个文件，应该在配置文件 `config/app.php `中设置 `log `值如下：

```
'log' => 'daily'
```

### 2.3 自定义 Monolog 配置
如果你想要在应用中完全控制 `Monolog `的配置，可以使用应用的 `configureMonologUsing `方法。你应该在 `bootstrap/app.php `文件返回`$app` 变量之前调用该方法：

```
$app->configureMonologUsing(function($monolog) {
    $monolog->pushHandler(...);
});

return $app;
```

## 3、异常处理器
所有异常都由类 `App\Exceptions\Handler `处理，该类包含两个方法：`report `和 `render`。下面我们详细阐述这两个方法。

### 3.1 report 方法
`report` 方法用于记录异常并将其发送给外部服务如 `Bugsnag `。默认情况下，`report `方法只是将异常传递给异常被记录的基类，你可以随心所欲的记录异常。

例如，如果你需要以不同方式报告不同类型的异常，可使用 `PHP `的 `instanceof `比较操作符：

```
/**
 * 报告或记录异常
 *
 * This is a great spot to send exceptions to Sentry, Bugsnag, etc.
 *
 * @param  \Exception  $e
 * @return void
 */
public function report(Exception $e){
    if ($e instanceof CustomException) {
        //
    }

    return parent::report($e);
}
```

#### 3.1.1 通过类型忽略异常
异常处理器的`$dontReport` 属性包含一个不会被记录的异常类型数组，默认情况下，404 错误异常不会被写到日志文件，如果需要的话你可以添加其他异常类型到这个数组。

### 3.2 render 方法
`render` 方法负责将给定异常转化为发送给浏览器的 `HTTP `响应，默认情况下，异常被传递给为你生成响应的基类。然而，你可以随心所欲地检查异常类型或者返回自定义响应：

```
/**
 * 将异常渲染到 HTTP 响应中
 *
 * @param  \Illuminate\Http\Request  $request
 * @param  \Exception  $e
 * @return \Illuminate\Http\Response
 */
public function render($request, Exception $e){
    if ($e instanceof CustomException) {
        return response()->view('errors.custom', [], 500);
    }

    return parent::render($request, $e);
}
```

## 4、HTTP 异常
有些异常描述来自服务器的 `HTTP `错误码，例如，这可能是一个“页面未找到”错误（404），“认证失败错误”（401）亦或是程序出错造成的 500 错误，为了在应用中生成这样的响应，使用如下方法：

```
abort(404);
```

`abort` 方法会立即引发一个会被异常处理器渲染的异常，此外，你还可以像这样提供响应描述：

```
abort(403, 'Unauthorized action.');
```

该方法可在请求生命周期的任何时间点使用。

### 4.1 自定义 HTTP 错误页面
Laravel 使得返回多种 `HTTP `状态码的错误页面变得简单，例如，如果你想要自定义 404 错误页面，创建一个 `resources/views/errors/404.blade.php `文件，给文件将会渲染程序生成的所有 404 错误。
改目录下的视图命名应该和相应的 HTTP 状态码相匹配。

## 5、日志
`Laravel` 日志工具基于强大的 `Monolog `库，默认情况下，Laravel 被配置为在 `storage/logs `目录下每日为应用生成日志文件，你可以使用 `Log `门面编写日志信息到日志中：

```
<?php

namespace App\Http\Controllers;

use Log;
use App\User;
use App\Http\Controllers\Controller;

class UserController extends Controller{
    /**
     * 显示指定用户的属性
     *
     * @param  int  $id
     * @return Response
     */
    public function showProfile($id)
    {
        Log::info('Showing user profile for user: '.$id);
        return view('user.profile', ['user' => User::findOrFail($id)]);
    }
}
```

该日志记录器提供了 RFC 5424中定义的八种日志级别：**emergency, alert, critical, error,warning, notice, info** 和 **debug**。

```
Log::emergency($error);
Log::alert($error);
Log::critical($error);
Log::error($error);
Log::warning($error);
Log::notice($error);
Log::info($error);
Log::debug($error);
```

### 5.1 上下文信息
上下文数据数组也会被传递给日志方法。上下文数据将会和日志消息一起被格式化和显示：

```
Log::info('User failed to login.', ['id' => $user->id]);
```

### 5.2 访问底层 Monolog 实例
Monolog 有多个可用于日志的处理器，如果需要的话，你可以访问底层 Monolog 实例：

```
$monolog = Log::getMonolog();
```