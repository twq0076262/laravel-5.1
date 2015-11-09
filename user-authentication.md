# 用户认证

## 1、简介

Laravel 中实现用户认证非常简单。实际上，几乎所有东西都已经为你配置好了。配置文件位于 config/auth.php，其中包含了用于调整认证服务行为的文档友好的选项配置。

### 1.1 数据库考量
默认情况下，Laravel 在 app 目录下包含了一个 Eloquent 模型 App\User，这个模型可以和默认的 Eloquent 认证驱动一起使用。如果你的应用不使用 Eloquent，你可以使用 database 认证驱动，该驱动使用了 Laravel 查询构建器。

为 App\User 模型构建数据库表结构的时候，确保 password 字段长度至少有 60 位。

还有，你应该验证 users 表包含了可以为空的、字符串类型的 remember_token 字段长度为 100，该字段用于存储被应用维护的”记住我（remember me）“的 session 令牌，这可以通过在迁移中使用`$table->rememberToken();`来实现。

## 2、用户认证快速入门
Laravel 处理两个认证控制器，位于 App\Http\Controllers\Auth命名空间下，AuthController 处理新用户注册和认证，PasswordController 包含帮助用户找回密码的逻辑。每个控制器都使用 trait 来引入它们需要的方法。对很多应用而言，你根本不需要修改这两个控制器。

### 2.1 路由
默认情况下，没有路由将请求指向用户认证控制器，你要手动在 app/Http/routes.php 文件中添加它们：

```
// 认证路由...
Route::get('auth/login', 'Auth\AuthController@getLogin');
Route::post('auth/login', 'Auth\AuthController@postLogin');
Route::get('auth/logout', 'Auth\AuthController@getLogout');
// 注册路由...
Route::get('auth/register', 'Auth\AuthController@getRegister');
Route::post('auth/register', 'Auth\AuthController@postRegister');
```

### 2.2 视图
尽管框架包含了用户认证控制器，你还是需要提供这些控制器可以渲染的视图。这些视图位于 resources/views/auth 目录，你可以按需自定义这些视图文件。登录视图是 resources/views/auth/login.blade.php，注册视图是 resources/views/auth/register.blade.php。

#### 2.2.1 登录表单示例

```
<!-- resources/views/auth/login.blade.php -->

<form method="POST" action="/auth/login">
    {!! csrf_field() !!}

    <div>
        Email
        <input type="email" name="email" value="{{ old('email') }}">
    </div>

    <div>
        Password
        <input type="password" name="password" id="password">
    </div>

    <div>
        <input type="checkbox" name="remember"> Remember Me
    </div>

    <div>
        <button type="submit">Login</button>
    </div>
</form>
```

#### 2.2.2 注册表单示例

```
<!-- resources/views/auth/register.blade.php -->

<form method="POST" action="/auth/register">
    {!! csrf_field() !!}

    <div>
        Name
        <input type="text" name="name" value="{{ old('name') }}">
    </div>

    <div>
        Email
        <input type="email" name="email" value="{{ old('email') }}">
    </div>

    <div>
        Password
        <input type="password" name="password">
    </div>

    <div>
        Confirm Password
        <input type="password" name="password_confirmation">
    </div>

    <div>
        <button type="submit">Register</button>
    </div>
</form>
```

### 2.3 认证
既然你已经为自带的认证控制器设置好了路由和视图，接下来就准备为应用注册新用户并进行登录认证。你可以在浏览器中访问定义好的路由，认证控制器已经实现了认证已存在用户以及存储新用户到数据库中的业务逻辑（通过 trait）。

当一个用户成功进行登录认证后，将会跳转到/home 链接，你需要事先注册一个路由来处理该跳转。你可以通过在 AuthController 中设置 redirectPath 属性来自定义 post 认证之后的跳转路径：

```
protected $redirectPath = '/dashboard';
```

当一个用户登录认证失败后，将会跳转到/auth/login 链接。你可以通过定义 AuthController 的 loginPath 属性来自定义 post 认证失败后的跳转路径：

```
protected $loginPath = '/login';
```

#### 2.3.1 自定义
要修改新用户注册所必需的表单字段，或者自定义新用户字段如何存储到数据库，你可以修改 AuthController 类。该类负责为应用验证和创建新用户。

AuthController 的 validator 方法包含了新用户的验证规则，你可以随意按需要自定义该方法。

AuthController 的 create 方法负责使用Eloquent ORM在数据库中创建新的 App\User 记录。你可以基于自己的需要随意自定义该方法。

### 2.4 获取认证用户
你可以通过 Auth 门面访问认证用户：

```
$user = Auth::user();
```

一旦用户通过认证后，你还可以通过 Illuminate\Http\Request 实例访问认证用户：

```
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use Illuminate\Routing\Controller;

class ProfileController extends Controller{
    /**
     * 更新用户属性.
     *
     * @param  Request  $request
     * @return Response
     */
    public function updateProfile(Request $request)
    {
        if ($request->user()) {
            // $request->user() 返回认证用户实例...
        }
    }
}
```

#### 2.4.1 判断当前用户是否通过认证
要判断某个用户是否登录到应用，可以使用 Auth门面的 check 方法，如果用户通过认证则返回 true：

```
if (Auth::check()) {
    // The user is logged in...
}
```

此外，你还可以在用户访问特定路由/控制器之前使用中间件来验证用户是否通过认证，想要了解更多，可以查看路由保护文档。

### 2.5 路由保护
路由中间件可用于只允许通过认证的用户访问给定路由。Laravel 通过定义在 app\Http\Middleware\Authenticate.php 的 auth 中间件来处理这一操作。你所要做的仅仅是将该中间件加到相应的路由定义中：

```
// 使用路由闭包...
Route::get('profile', ['middleware' => 'auth', function() {
    // 只有认证用户可以进入...
}]);
// 使用控制器...
Route::get('profile', [
    'middleware' => 'auth',
    'uses' => 'ProfileController@show'
]);
```

当然，如果你正在使用控制器类，也可以在控制器的构造方法中调用 middleware 方法而不是在路由器中直接定义：

```
public function __construct(){
    $this->middleware('auth');
}
```

### 2.6 登录失败次数限制
如果你正在使用 Laravel 内置的 AuthController 类，Illuminate\Foundation\Auth\ThrottlesLogins trait 可以用于限制用户登录失败次数。默认情况下，用户在几次登录失败后将在一分钟内不能登录，这种限制基于用户的用户名/邮箱地址+IP 地址：

```
<?php

namespace App\Http\Controllers\Auth;

use App\User;use Validator;
use App\Http\Controllers\Controller;
use Illuminate\Foundation\Auth\ThrottlesLogins;
use Illuminate\Foundation\Auth\AuthenticatesAndRegistersUsers;

class AuthController extends Controller{
    use AuthenticatesAndRegistersUsers, ThrottlesLogins;

    // AuthController 类的其它部分...
}
```

## 3、手动认证用户
当然，你也可以不使用 Laravel 自带的认证控制器。如果你选择移除这些控制器，你需要直接使用 Laravel 认证类来管理用户认证。别担心，这很简单！

我们将会通过 Auth 门面来访问认证服务，因此我们需要确保在类的顶部导入了 Auth门面，让我们看看 attempt 方法：

```
<?php

namespace App\Http\Controllers;

use Auth;
use Illuminate\Routing\Controller;

class AuthController extends Controller{
    /**
     * 处理登录认证
     *
     * @return Response
     */
    public function authenticate()
    {
        if (Auth::attempt(['email' => $email, 'password' => $password])) {
            // 认证通过...
            return redirect()->intended('dashboard');
        }
    }
}
```

attempt 方法接收键值数组对作为第一个参数，数组中的值被用于从数据表中查找用户，因此，在上面的例子中，用户将会通过 email 的值获取，如果用户被找到，经哈希运算后存储在数据中的密码将会和传递过来的经哈希运算处理的密码值进行比较。如果两个经哈希运算的密码相匹配那么一个认证 session 将会为这个用户开启。

如果认证成功的话 attempt 方法将会返回 true。否则，返回 false。
重定向器上的 intended 方法将会将用户重定向到登录之前用户想要访问的 URL，在目标 URL 无效的情况下备用 URI 将会传递给该方法。
如果你想的话，除了用户邮件和密码之外还可以在认证查询时添加额外的条件，例如，我们可以验证被标记为有效的用户：

```
if (Auth::attempt(['email' => $email, 'password' => $password, 'active' => 1])) {
    // The user is active, not suspended, and exists.
}
```

要退出应用，可以使用 Auth 门面的 logout 方法，这将会清除用户 session 中的认证信息：

```
Auth::logout();
```

注意：在这些例子中，email 并不是必须选项，在这里只不过是作为一个例子。你可以在自己的数据库使用任何其他与“用户名”相对应的字段。

### 3.1 记住用户
如果你想要在应用中提供“记住我”的功能，可以传递一个布尔值作为第二个参数到 attempt 方法，这样用户登录认证状态就会一直保持直到他们手动退出。当然，你的 users 表必须包含 remember_token 字段，该字段用于存储“记住我”令牌。

```
if (Auth::attempt(['email' => $email, 'password' => $password], $remember)) {
    // The user is being remembered...
}
```

如果你要“记住”用户，可以使用 viaRemember 方法来判断用户是否使用“记住我”cookie 进行认证：

```
if (Auth::viaRemember()) {
    //
}
```

### 3.2 其它认证方法

#### 3.2.1 认证用户实例
如果你需要将一个已存在的用户实例登录到应用中，可以调用用户实例上的 login 方法，给定实例必须是 Illuminate\Contracts\Auth\Authenticatable契约的实现，当然，Laravel 自带的 App\User 模型已经实现了该接口：

```
Auth::login($user);
```

#### 3.2.2 通过 ID 认证用户
要通过用户 ID 登录到应用，可以使用 loginUsingId 方法，该方法接收你想要认证用户的主键作为参数：

```
Auth::loginUsingId(1);
```

#### 3.2.3 一次性认证用户
你可以使用 once 方法只在单个请求中将用户登录到应用，而不存储任何 session 和 cookie，这在构建无状态的 API 时很有用。once 方法和 attempt 方法用法差不多：

```
if (Auth::once($credentials)) {
    //
}
```

## 4、基于HTTP的基本认证
HTTP 基本认证能够帮助用户快速实现登录认证而不用设置专门的登录页面，首先要在路由中加上 auth.basic中间件。该中间件是 Laravel 自带的，所以不需要自己定义：

```
Route::get('profile', ['middleware' => 'auth.basic', function() {
    // 只有认证用户可以进入...
}]);
```

中间件加到路由中后，当在浏览器中访问该路由时，会自动提示需要认证信息，默认情况下，auth.basic 中间件使用用户记录上的 email 字段作为“用户名”。

**FastCGI 上注意点**
如果你使用 PHP FastCGI，HTTP 基本认证将不能正常工作，需要在.htaccess 文件加入如下内容：

```
RewriteCond %{HTTP:Authorization} ^(.+)$
RewriteRule .* - [E=HTTP_AUTHORIZATION:%{HTTP:Authorization}]
```

### 4.1 无状态的 HTTP 基本认证
使用 HTTP 基本认证也不需要在 session 中设置用户标识 cookie，这在 API 认证中非常有用。要实现这个，需要定义一个调用 onceBasic 方法的中间件。如果该方法没有返回任何响应，那么请求会继续走下去：

```
<?php

namespace Illuminate\Auth\Middleware;

use Auth;
use Closure;

class AuthenticateOnceWithBasicAuth{
    /**
     * 处理输入请求.
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  \Closure  $next
     * @return mixed
     */
    public function handle($request, Closure $next)
    {
        return Auth::onceBasic() ?: $next($request);
    }

}
```

接下来，注册路由中间件并将其添加到路由中：

```
Route::get('api/user', ['middleware' => 'auth.basic.once', function() {
    // 只有认证用户可以进入...
}]);
```

## 5、重置密码

### 5.1 数据库考量
大多数 web 应用提供了用户重置密码的功能，Laravel 提供了便利方法用于发送密码提示及执行密码重置而不需要你在每个应用中重新实现。

开始之前，先验证 App\User 模型实现了 Illuminate\Contracts\Auth\CanResetPassword 契约。当然，Laravel 自带的 App\User 模型已经实现了该接口，并使用 Illuminate\Auth\Passwords\CanResetPassword trait 来包含实现该接口需要的方法。

#### 5.1.1 生成重置令牌表迁移
接下来，用来存储密码重置令牌的表必须被创建，Laravel 已经自带了这张表的迁移，就存放在 database/migrations 目录。所有，你所要做的仅仅是运行迁移：

```
php artisan migrate
```

### 5.2 路由
Laravel 自带了 Auth\PasswordController，其中包含重置用户必须的逻辑。然而，你需要定义一个路由将请求转发到该控制器：

```
// 密码重置链接请求路由...
Route::get('password/email', 'Auth\PasswordController@getEmail');
Route::post('password/email', 'Auth\PasswordController@postEmail');
// 密码重置路由...
Route::get('password/reset/{token}', 'Auth\PasswordController@getReset');
Route::post('password/reset', 'Auth\PasswordController@postReset');
```

### 5.3 视图
除了为 Auth\PasswordController 定义路由之外，还需要提供相应视图，别担心，我们将会提供示例视图来帮助你开始，当然，你也可以自定义表单样式。

#### 5.3.1 密码重置链接请求表单示例
你需要为密码重置请求表单 提供 HTML 视图，该视图文件应该放在 resources/views/auth/password.blade.php，表单提供了一个输入用户邮箱地址的字段，从而允许用户从邮件中访问密码重置链接：

```
<!-- resources/views/auth/password.blade.php -->

<form method="POST" action="/password/email">
    {!! csrf_field() !!}

    <div>
        Email
        <input type="email" name="email" value="{{ old('email') }}">
    </div>

    <div>
        <button type="submit">
            Send Password Reset Link
        </button>
    </div>
</form>
```

当一个用户提交了重置密码请求后，将会收到一封电子邮件，其中包含了一个链接，该链接指向 PasswordController 的 getReset 方法，你需要为该电子邮件创建一个视图 resources/views/emails/password.blade.php。该视图将会获取包含密码重置令牌的`$token` 变量，用于和用户重置密码请求进行匹配。下面是一个电子邮件视图的例子：

```
<!-- resources/views/emails/password.blade.php -->
Click here to reset your password: {{ url('password/reset/'.$token) }}
```

#### 5.3.2 密码重置表单示例
当用户点击电子邮件中的链接来重置密码时，需要提交一个密码重置表单，该视图位于 resources/views/auth/reset.blade.php。
下面是一个密码重置表单示例：

```
<!-- resources/views/auth/reset.blade.php -->

<form method="POST" action="/password/reset">
    {!! csrf_field() !!}
    <input type="hidden" name="token" value="{{ $token }}">

    <div>
        <input type="email" name="email" value="{{ old('email') }}">
    </div>

    <div>
        <input type="password" name="password">
    </div>

    <div>
        <input type="password" name="password_confirmation">
    </div>

    <div>
        <button type="submit">
            Reset Password
        </button>
    </div>
</form>
```

### 5.4 重置密码后
如果你已经定义好路由和视图来重置用户密码，只需要在浏览器中访问这些路由即可。框架自带的 PasswordController 已经包含了发送密码重置链接邮件以及更新数据库中密码的逻辑。
密码被重置后，用户将会自动登录到应用并重定向到/home。你可以通过定义上 PasswordController 的 redirectTo 属性来自定义 post 密码重置跳转链接：

```
protected $redirectTo = '/dashboard';
```

注意：默认情况下，密码重置令牌一小时内有效，你可以通过修改 config/auth.php 文件中的选项 reminder.expire 来改变有效时间。

## 6、社会化登录认证
Laravel 中还可以使用Laravel Socialite通过 OAuth 提供者进行简单、方便的认证，也就是社会化登录，目前支持使用 Facebook、Twitter、LinkedIn、Google 和 Bitbucket 进行登录认证。

要使用社会化登录，需要在 composer.json 文件中添加依赖：

```
composer require laravel/socialite
```

### 6.1 配置
安装完社会化登录库后，在配置文件 config/app.php 中注册 Laravel\Socialite\SocialiteServiceProvider：

```
'providers' => [
    // 其它服务提供者...
    Laravel\Socialite\SocialiteServiceProvider::class,
],
```

还要在 app 配置文件中添加 Socialite 门面到 aliases 数组：

```
'Socialite' => Laravel\Socialite\Facades\Socialite::class,
```

你还需要为应用使用的 OAuth 服务添加认证信息，这些认证信息位于配置文件 config/services.php，而且键为 facebook, twitter,linkedin, google, github 或 bitbucket，这取决于应用需要的提供者。例如：

```
'github' => [
    'client_id' => 'your-github-app-id',
    'client_secret' => 'your-github-app-secret',
    'redirect' => 'http://your-callback-url',
],
```

### 6.2 基本使用
接下来，准备好认证用户！你需要两个路由：一个用于重定向用户到 OAuth 提供者，另一个用户获取认证后来自提供者的回调。我们使用 Socialite 门面访问 Socialite ：

```
<?php

namespace App\Http\Controllers;

use Socialite;
use Illuminate\Routing\Controller;

class AuthController extends Controller{
    /**
     * 将用户重定向到 GitHub 认证页面.
     *
     * @return Response
     */
    public function redirectToProvider()
    {
        return Socialite::driver('github')->redirect();
    }

    /**
     * 从 GitHub 获取用户信息.
     *
     * @return Response
     */
    public function handleProviderCallback()
    {
        $user = Socialite::driver('github')->user();

        // $user->token;
    }
}
```

redirect 方法将用户发送到 OAuth 提供者，user 方法读取请求信息并从提供者中获取用户信息，在重定向用户之前，你还可以在请求上使用 scope 方法设置”作用域”，该方法将会重写已存在的所有作用域：

```
return Socialite::driver('github')
            ->scopes(['scope1', 'scope2'])->redirect();
```

当然，你需要定义路由到控制器方法：

```
 Route::get('auth/github', 'Auth\AuthController@redirectToProvider');
 Route::get('auth/github/callback', 'Auth\AuthController@handleProviderCallback');
```

#### 6.2.1 获取用户信息
有了用户实例之后，可以获取用户的更多详情：

```
$user = Socialite::driver('github')->user();
// OAuth Two Providers
$token = $user->token;
// OAuth One Providers
$token = $user->token;
$tokenSecret = $user->tokenSecret;
// All Providers
$user->getId();
$user->getNickname();
$user->getName();
$user->getEmail();
$user->getAvatar();
```

## 7、添加自定义认证驱动
如果你没有使用传统的关系型数据库存储用户信息，你需要使用自己的认证驱动扩展 Laravel。我们使用 Auth门面上的 extend 方法来定义自定义的驱动，你需要在服务提供者调用 extend 方法：

```
<?php

namespace App\Providers;

use Auth;
use App\Extensions\RiakUserProvider;
use Illuminate\Support\ServiceProvider;

class AuthServiceProvider extends ServiceProvider{
    /**
     * Perform post-registration booting of services.
     *
     * @return void
     */
    public function boot()
    {
        Auth::extend('riak', function($app) {
            // 返回 Illuminate\Contracts\Auth\UserProvider 实例...
            return new RiakUserProvider($app['riak.connection']);
        });
    }

    /**
     * 在容器中注册绑定.
     *
     * @return void
     */
    public function register()
    {
        //
    }
}
```

通过 extend 方法注册驱动后，你可以在配置文件 config/auth.php 中切换到新的驱动。

### 7.1 UserProvider 契约
Illuminate\Contracts\Auth\UserProvider 实现只负责从持久化存储系统中获取 Illuminate\Contracts\Auth\Authenticatable 实现，例如 MySQL、Riak 等等。这两个接口允许 Laravel 认证机制继续起作用而不管用户数据如何存储或者何种类来展现。
让我们先看看 Illuminate\Contracts\Auth\UserProvider契约：

```
<?php

namespace Illuminate\Contracts\Auth;

interface UserProvider {

    public function retrieveById($identifier);
    public function retrieveByToken($identifier, $token);
    public function updateRememberToken(Authenticatable $user, $token);
    public function retrieveByCredentials(array $credentials);
    public function validateCredentials(Authenticatable $user, array $credentials);

}
```

retrieveById 方法通常获取一个代表用户的键，例如 MySQL 数据中的自增 ID。该方法获取并返回匹配该 ID 的 Authenticatabl 实现。
retrieveByToken 函数通过唯一标识和存储在 remember_token 字段中的“记住我”令牌获取用户。和上一个方法一样，该方法也返回 Authenticatabl 实现。

updateRememberToken 方法使用新的`$token` 更新`$user` 的 remember_token 字段，新令牌可以是新生成的令牌（在登录是选择“记住我”被成功赋值）或者 null（用户退出）。

retrieveByCredentials 方法在尝试登录系统时获取传递给 Auth::attempt 方法的认证信息数组。该方法接下来去底层持久化存储系统查询与认证信息匹配的用户，通常，该方法运行一个带“where”条件`（$credentials[‘username’]）`的查询。然后该方法返回 UserInterface 的实现。这个方法不做任何密码校验和认证。

validateCredentials 方法比较给定`$user` 和`$credentials` 来认证用户。例如，这个方法比较`$user->getAuthPassword()`字符串和经 Hash::make 处理的`$credentials['password']`。这个方法只验证用户认证信息并返回布尔值。

### 7.2 Authenticatable 契约
既然我们已经探索了 UserProvider 上的每一个方法，接下来让我们看看 Authenticatable。该提供者应该从 retrieveById 和 retrieveByCredentials 方法中返回接口实现：

```
<?php

namespace Illuminate\Contracts\Auth;

interface Authenticatable {

    public function getAuthIdentifier();
    public function getAuthPassword();
    public function getRememberToken();
    public function setRememberToken($value);
    public function getRememberTokenName();

}
```

这个接口很简单，getAuthIdentifier 方法返回用户“主键”，在 MySQL 后台中是 ID，getAuthPassword 返回经哈希处理的用户密码，这个接口允许认证系统处理任何用户类，不管是你使用的是 ORM 还是存储抽象层。默认情况下，Laravel 自带的 app 目录下的 User 类实现了这个接口，所以你可以将这个类作为实现例子。