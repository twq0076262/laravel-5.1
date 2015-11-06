# 发行版本说明

## 支持政策
LTS 版本，比如 Laravel，将会提供两年的 bug 修复和三年的安全修复支持。这些版本将会提供最长时间的支持和维护。

对于其他通用版本，只提供六个月的 bug 修复和一年的安全修复支持。

## Laravel 5.1.11
Laravel 5.1.11 引入了“开箱即用”的授权支持！使用简单的回调或策略类即可方便地管理应用的授权逻辑，并且授权动作使用简单且优雅的方法。

想要了解更多信息，请查看授权文档。

## Laravel 5.1.4
Laravel 5.1.4 将登录次数限制引入框架，更多详情请参考认证限制一节。

## Laravel 5.1
Laravel 5.1 在 5.0 的基础上继续进行优化和提升，接受 PSR-2 代码风格，新增事件广播机制，中间件参数，Artisan 优化，等等。

### PHP 5.5.9+
由于 PHP 5.4 将会在今年9月份“寿终正寝”，并且 PHP 开发组不会再提供安全更新，Laravel 5.1  要求 PHP5.5.9 或更高版本。PHP5.5.9 兼容一些最新版本的流行 PHP 库如 Guzzle 和 AWS SDK。

### LTS
Laravel 5.1 是 Laravel 第一个长期支持版本，将会提供两年的 bug 修复和安全修复，这是迄今为止，Laravel 提供的最大跨度的支持，并且将会持续为更多的企业用户及普通用户提供稳定平滑的支持。

### PSR-2
PSR-2 代码风格指南已经被 Laravel 框架采取为默认风格指南，此外，所有代码生成器已经被更新到生成兼容 PSR-2 语法的代码。

### 文档
Laravel 文档的每一个页面都进行了一丝不苟的审查和引人注目的优化，所有代码示例都被审查并且扩展到更好的支持上下文相关性。

### 事件广播
在很多现代的 web 应用中，web 套接字被用于实现实时的，即时更新的用户接口，当服务器上的某些数据更新后，通常一条消息将会通过 websocket 连接发送到客户端并进行处理。

为了帮助你构建这样类型的应用，Laravel 使得通过 websocket 连接广播事件变得简单可行。广播 Laravel 事件允许你在服务端代码和客户端 JavaScript 框架之间共享相同的事件名称。

更多关于事件广播的内容请查看事件一节。

### 中间件参数
Laravel 5.1 里，中间件可以接受额外的自定义参数，例如，如果你的应用需要在执行给定的 action 之前验证被授予指定“角色”的认证用户，可以创建一个 RoleMiddleware 来接收角色名称作为额外参数：

```
<?php

namespace App\Http\Middleware;

use Closure;

class RoleMiddleware
{
    /**
     * 运行请求过滤器.
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  \Closure  $next
     * @param  string  $role
     * @return mixed
     */
    public function handle($request, Closure $next, $role)
    {
        if (! $request->user()->hasRole($role)) {
            // 跳转...
        }

        return $next($request);
    }

}
```

中间件参数可以再定义路由时通过：分隔中间件名称和参数名称来指定，多个参数可以通过逗号进行分隔：

```
Route::put('post/{id}', ['middleware' => 'role:editor', function ($id) {
    //
}]);
```

更多关于中间件的内容，请查看中间件一节。

### 测试革新
Laravel 中内置的测试功能获得了引入注目的提升，多个新方法提供了平滑的，富有变现力的接口和应用进行交互并测试响应：

```
public function testNewUserRegistration(){
    $this->visit('/register')
         ->type('Taylor', 'name')
         ->check('terms')
         ->press('Register')
         ->seePageIs('/dashboard');
}
```

更多有关测试的内容，请查看测试一节。

### 模型工厂
Laravel 现在可以通过使用模型工厂附带一种简单的方式类创建 Eloquent 模型存根，模型工厂允许你为 Eloquent 模型定义一系列默认属性，然后为测试或数据库填充生成模型实例。模型工厂还可以利用强大的 PHP 扩展库 Faker 类生成随机的属性数据。

```
$factory->define('App\User', function ($faker) {
    return [
        'name' => $faker->name,
        'email' => $faker->email,
        'password' => str_random(10),
        'remember_token' => str_random(10),
    ];
});
```

更多关于模型工厂的内容，请查看模型工厂一节。

### Artisan 优化
Artisan 命令可以通过使用一个简单的，类似路由风格的“签名”（提供了一个非常简单的接口来定义命令行参数和选项）来定义：

```
/**
 * 命令行的名称和签名.
 *
 * @var string
 */
protected $signature = 'email:send {user} {--force}';
```

更多关于 Artisan 的内容，请查看命令行一节。

### 目录结构
为了更好地表达意图，app/Commands 目录被重命名为 app/Jobs，此外，app/Handlers 被合并到 app/Listeners 目录。然而这并不是破坏式改变所以使用 Laravel 5.1 并不强制要求更新到新的目录结构。

### 加密
在之前的 Laravel 版本中，加密通过 PHP 扩展 mcrypt 进行处理，从5.1开始，加密改由通过 PHP 的另一个扩展 openssl 进行处理，因为该扩展较前者而言维护的更加活跃。