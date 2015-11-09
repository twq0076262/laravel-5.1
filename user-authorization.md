# 用户授权

## 1、简介
除了提供“开箱即用”的认证服务之外，Laravel 还提供了一个简单的方式来管理授权逻辑以便控制对资源的访问权限。在 Laravel 中，有很多种方法和帮助函数来协助你管理授权逻辑，本文档将会一一覆盖这些方法。
注意：授权在 Laravel 5.1.11 版本中引入，在将该特性集成到应用之前请参考升级指南。

## 2、定义权限（Abilities）
判断用户是否有权限执行给定动作的最简单方式就是使用 Illuminate\Auth\Access\Gate类来定义一个“权限”。我们在 AuthServiceProvider 中定义所有权限，例如，我们来定义一个接收当前 User 和 Post模型的 update-post 权限，在该权限中，我们判断用户 id 是否和文章的 user_id 匹配：

```
<?php

namespace App\Providers;

use Illuminate\Contracts\Auth\Access\Gate as GateContract;
use Illuminate\Foundation\Support\Providers\AuthServiceProvider as ServiceProvider;

class AuthServiceProvider extends ServiceProvider{
    /**
     * 注册应用所有的认证/授权服务.
     *
     * @param  \Illuminate\Contracts\Auth\Access\Gate  $gate
     * @return void
     */
    public function boot(GateContract $gate)
    {
        parent::registerPolicies($gate);

        $gate->define('update-post', function ($user, $post) {
            return $user->id === $post->user_id;
        });
    }
}
```

注意我们并没有检查给定`$user` 是否为 NULL，当用户未经过登录认证或者用户没有通过 forUser 方法指定，Gate 会自动为所有权限返回 false。

**基于类的权限**
除了注册授权回调闭包之外，还可以通过传递包含权限类名和类方法的方式来注册权限方法，当需要的时候，该类会通过服务容器进行解析：

```
$gate->define('update-post', 'PostPolicy@update');
```

## 3、检查权限（Abilities）

### 3.1 通过 Gate 门面
权限定义好之后，可以使用多种方式来“检查”。首先，可以使用 Gate 门面的 check, allows, 或者 denies 方法。所有这些方法都接收权限名和传递给该权限回调的参数作为参数。你不需要传递当前用户到这些方法，因为 Gate 会自动附加当前用户到传递给回调的参数，因此，当检查我们之前定义的 update-post 权限时，我们只需要传递一个 Post 实例到 denies 方法：

```
<?php

namespace App\Http\Controllers;

use Gate;
use App\User;
use App\Post;
use App\Http\Controllers\Controller;

class PostController extends Controller{
    /**
     * 更新给定文章
     *
     * @param  int  $id
     * @return Response
     */
    public function update($id)
    {
        $post = Post::findOrFail($id);

        if (Gate::denies('update-post', $post)) {
            abort(403);
        }

        // 更新文章...
    }
}
```

当然，allows 方法和 denies 方法是相对的，如果动作被授权会返回 true ，check 方法是 allows 方法的别名。

**为指定用户检查权限**
如果你想要使用 Gate 门面判断非当前用户是否有权限，可以使用 forUser 方法：

```
if (Gate::forUser($user)->allows('update-post', $post)) {
    //
}
```

**传递多个参数**
当然，权限回调还可以接收多个参数：

```
Gate::define('delete-comment', function ($user, $post, $comment) {
    //
});
```

如果权限需要多个参数，简单传递参数数组到 Gate 方法：

```
if (Gate::allows('delete-comment', [$post, $comment])) {
    //
}
```

### 3.2 通过 User 模型
还可以通过 User 模型实例来检查权限。默认情况下，Laravel 的 App\User 模型使用一个 Authorizabletrait 来提供两种方法：can 和 cannot。这两个方法的功能和 Gate 门面上的 allows 和 denies 方法类似。因此，使用我们前面的例子，可以修改代码如下：

```
<?php

namespace App\Http\Controllers;

use App\Post;
use Illuminate\Http\Request;
use App\Http\Controllers\Controller;

class PostController extends Controller{
    /**
     * 更新给定文章
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  int  $id
     * @return Response
     */
    public function update(Request $request, $id)
    {
        $post = Post::findOrFail($id);

        if ($request->user()->cannot('update-post', $post)) {
            abort(403);
        }

        // 更新文章...
    }
}
```

当然，can 方法和 cannot 方法相反：

```
if ($request->user()->can('update-post', $post)) {
    // 更新文章...
}
```

### 3.3 在Blade模板引擎中检查
为了方便，Laravel 提供了 Blade 指令@can 来快速检查当前用户是否有指定权限。例如：

```
<a href="/post/{{ $post->id }}">View Post</a>

@can('update-post', $post)
    <a href="/post/{{ $post->id }}/edit">Edit Post</a>
@endcan
```

你还可以将 @can 指令和@else 指令联合起来使用：

```
@can('update-post', $post)
    <!-- The Current User Can Update The Post -->
@else
    <!-- The Current User Can't Update The Post -->
@endcan
```

### 3.4 在表单请求中检查
你还可以选择在表单请求的 authorize 方法中使用 Gate 定义的权限。例如：

```
/**
 * 判断请求用户是否经过授权
 *
 * @return bool
 */
public function authorize(){
    $postId = $this->route('post');
    return Gate::allows('update', Post::findOrFail($postId));
}
```

## 4、策略类（Policies）

## 4.1 创建策略类
由于在 AuthServiceProvider 中定义所有的授权逻辑将会变得越来越臃肿笨重，尤其是在大型应用中，所以 Laravel 允许你将授权逻辑分割到多个“策略”类中，策略类是原生的 PHP 类，基于授权资源对授权逻辑进行分组。

首先，让我们生成一个策略类来管理对 Post 模型的授权，你可以使用 Artisan 命令 make:policy 来生成该策略类。生成的策略类位于 app/Policies 目录：

```
php artisan make:policy PostPolicy
```

**注册策略类**
策略类生成后我们需要将其注册到 Gate 类。AuthServiceProvider 包含了一个 policies 属性来映射实体及管理该实体的策略类。因此，我们指定 Post 模型的策略类是 PostPolicy：

```
<?php

namespace App\Providers;

use App\Post;
use App\Policies\PostPolicy;
use Illuminate\Contracts\Auth\Access\Gate as GateContract;
use Illuminate\Foundation\Support\Providers\AuthServiceProvider as ServiceProvider;

class AuthServiceProvider extends ServiceProvider{
    /**
     * 应用的策略映射
     *
     * @var array
     */
    protected $policies = [
        Post::class => PostPolicy::class,
    ];
}
```

### 4.2 编写策略
策略类生成和注册后，我们可以为授权的每个权限添加方法。例如，我们在 PostPolicy 中定义一个 update 方法，该方法判断给定 User 是否可以更新某个 Post：

```
<?php

namespace App\Policies;

use App\User;
use App\Post;

class PostPolicy{
    /**
     * 判断给定文章是否可以被给定用户更新
     *
     * @param  \App\User  $user
     * @param  \App\Post  $post
     * @return bool
     */
    public function update(User $user, Post $post)
    {
        return $user->id === $post->user_id;
    }
}
```

你可以继续在策略类中为授权的权限定义更多需要的方法，例如，你可以定义 show, destroy, 或者 addComment 方法来认证多个 Post 动作。
注意：所有策略类都通过服务容器进行解析，这意味着你可以在策略类的构造函数中类型提示任何依赖，它们将会自动被注入。

### 4.3 检查策略
策略类方法的调用方式和基于授权回调的闭包一样，你可以使用 Gate 门面，User 模型，@can 指令或者帮助函数 policy。

**通过 Gate 门面**
Gate 将会自动通过检测传递过来的类参数来判断使用哪一个策略类，因此，如果传递一个 Post 实例给 denies 方法，相应的，Gate 会使用 PostPolicy 来进行动作授权：

```
<?php

namespace App\Http\Controllers;

use Gate;
use App\User;
use App\Post;
use App\Http\Controllers\Controller;

class PostController extends Controller{
    /**
     * 更新给定文章
     *
     * @param  int  $id
     * @return Response
     */
    public function update($id)
    {
        $post = Post::findOrFail($id);

        if (Gate::denies('update', $post)) {
            abort(403);
        }

        // 更新文章...
    }
}
```

**通过 User 模型**
User 模型的 can 和 cannot 方法将会自动使用给定参数中有效的策略类。这些方法提供了便利的方式来为应用接收到的任意 User 实例进行授权：

```
if ($user->can('update', $post)) {
    //
}

if ($user->cannot('update', $post)) {
    //
}
```

**Blade 模板中的使用**
类似的，Blade 指令@can 将会使用参数中有效的策略类：

```
@can('update', $post)
    <!-- The Current User Can Update The Post -->
@endcan
```

**通过帮助函数 policy**
全局的帮助函数 policy 用于为给定类实例接收策略类。例如，我们可以传递一个 Post 实例给帮助函数 policy 来获取相应的 PostPolicy 类的实例：

```
if (policy($post)->update($user, $post)) {
    //
}
```

## 5、控制器授权
默认情况下，Laravel 自带的控制器基类 App\Http\Controllers\Controller 使用了 AuthorizesRequeststrait，该 trait 提供了可用于快速授权给定动作的 authorize 方法，如果授权不通过，则抛出 HttpException 异常。

该 authorize 方法和其他多种授权方法使用方法一致，例如 Gate::allows 和`$user->can()`。因此，我们可以这样使用 authorize 方法快速授权更新 Post 的请求：

```
<?php

namespace App\Http\Controllers;

use App\Post;
use App\Http\Controllers\Controller;

class PostController extends Controller{
    /**
     * 更新给定文章
     *
     * @param  int  $id
     * @return Response
     */
    public function update($id)
    {
        $post = Post::findOrFail($id);

        $this->authorize('update', $post);

        // 更新文章...
    }
}
```

如果授权成功，控制器继续正常执行；然而，如果 authorize 方法判断该动作授权失败，将会抛出 HttpException 异常并生成带 403 Not Authorized 状态码的 HTTP 响应。正如你所看到的，authorize 方法是一个授权动作、抛出异常的便捷方法。
AuthorizesRequeststrait 还提供了 authorizeForUser 方法用于授权非当前用户：

```
$this->authorizeForUser($user, 'update', $post);
```

**自动判断策略类方法**
通常，一个策略类方法对应一个控制器上的方法，例如，在上面的 update 方法中，控制器方法和策略类方法共享同一个方法名：update。
正是因为这个原因，Laravel 允许你简单传递实例参数到 authorize 方法，被授权的权限将会自动基于调用的方法名进行判断。在本例中，由于 authorize 在控制器的 update 方法中被调用，那么对应的，PostPolicy 上 update 方法将会被调用：

```
/**
 * 更新给定文章
 *
 * @param  int  $id
 * @return Response
 */
public function update($id){
    $post = Post::findOrFail($id);

    $this->authorize($post);

    // 更新文章...
}
```