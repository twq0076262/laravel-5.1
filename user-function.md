# 带用户功能的任务管理系统

本进阶指南提供了对 Laravel 框架更深入的介绍，包括数据库迁移、Eloquent ORM、路由、认证、授权、依赖注入、验证、视图以及 Blade 模板。如果你对 Laravel 框架或其他 PHP 框架已经有了基本的认识，本章节将是你新的起点，如果你完全还是新手，请从新手入门指南开始。

本节的示例仍然是构建一个任务系统，但是在上一节基础上，本任务系统将允许用户注册登录，同样完整的代码已经放到 GitHub 上：https://github.com/laravel/quickstart-intermediate。

# 1、安装
当然，首先你需要安装一个新的 Laravel 应用。你可以使用 Homestead 虚拟机或者本地 PHP 开发环境来运行应用。开发环境准备完毕后，可以使用 Composer 来安装应用：

```
composer create-project laravel/laravel quickstart --prefer-dist
```

你可以继续往下阅读，也可以选择去 GitHub 下载项目源码并在本地运行：

```
git clone https://github.com/laravel/quickstart-intermediate quickstart
cd quickstart
composer install
php artisan migrate
```

关于构建本地开发环境的详细文档可查看 Homestead 和安装文档。

# 2、准备好数据库
## 数据库迁移

首先，我们使用迁移来定于处理所有任务的数据库。Laravel 的数据库迁移使用平滑、优雅的 PHP 代码来提供一个简单的方式定义和修改数据表结构。团队成员们无需在本地数据库手动添加或删除列，只需要简单运行你提交到源码控制系统中的迁移即可。

### users 表

由于我们允许用户注册，所以需要一张用来存储用户的表。幸运的是 Laravel 已经自带了这个迁移用于创建基本的 users 表，我们不需要手动生成。该迁移文件默认位于 database/migrations 目录下。

### tasks 表

接下来，让我们来创建用于处理所有任务的数据表 tasks。我们使用 Artisan 命令 make:migration 来为 tasks 生成一个新的数据库迁移：

```
php artisan make:migration create_tasks_table --create=tasks
```

生成的新迁移文件位于 database/migrations 目录下。你可能已经注意到了，make:migration 命令已经在迁移文件中添加了自增 ID 和时间戳。我们将编辑该文件添加更多的字段到任务表 tasks：

```
<?php

use Illuminate\Database\Schema\Blueprint;
use Illuminate\Database\Migrations\Migration;

class CreateTasksTable extends Migration{
    /**
     * Run the migrations.
     *
     * @return void
     */
    public function up()
    {
        Schema::create('tasks', function (Blueprint $table) {
            $table->increments('id');
            $table->integer('user_id')->index();
            $table->string('name');
            $table->timestamps();
        });
    }

    /**
     * Reverse the migrations.
     *
     * @return void
     */
    public function down()
    { 
        Schema::drop('tasks');
    }
}
```

其中，user_id 用于建立 tasks 表与 users 表之间的关联。

要运行迁移，可以使用 migrate 命令。如果你使用的是 Homestead，需要在虚拟机中运行该命令，因为你的主机不能直接访问 Homestead 上的数据库：

```
php artisan migrate
```

该命令将会创建迁移中定义的尚未创建的所有数据表。如果你使用 MySQL 客户端（如 Navicat For MySQL）查看数据表，你将会看到新的 users 表和 tasks 表。下一步，我们将要定义 Eloquent ORM模型。

## Eloquent 模型

Eloquent 是 Laravel 默认的 ORM，Eloquent 使用“模型”这一概念使得从数据库存取数据变得轻松。通常，每个 Eloquent 模型都对应一张数据表。

### User 模型

首先，我们一个与 users 表相对应的模型 User。Laravel 已经自带了这一模型 app/User，所以我们不需要重复创建了。

### Task 模型

接下来，我们来定义与 tasks 表相对应的模型 Task。同样，我们使用 Artisan 命令来生成模型类，在本例中，我们使用 make:model 命令：

```
php artisan make:model Task
```

该模型位于应用的 app 目录下，默认情况下，该模型类是空的。我们不需要明确告诉 Eloquent 模型对应哪张表，Laravel 底层会有一个映射规则，这一点在之前 Eloquent 文档已有说明，按照规则，这里 Task 模型默认对应 tasks 表。

接下来，让我们在 Task 模型类中加一些代码。首先，我们声明模型上的 name 属性支持“批量赋值”（关于批量赋值说明可查看这篇文章）：

```
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class Task extends Model{
    /**
     * The attributes that are mass assignable.
     *
     * @var array
     */
    protected $fillable = ['name'];
}
```

我们将在后续添加路由到应用中学习更多如何使用 Eloquent 模型。当然，你也可以先去查看完整的 Eloquent 文档了解更多。

## Eloquent 关联关系

现在，模型已经定义好了，我们需要将它们关联起来。例如，一个 User 实例对应多个 Task 实例，而一个 Task 实例从属于某个 User。定义关联关系后将允许我们更方便的获取关联模型：

```
$user = App\User::find(1);

foreach ($user->tasks as $task) {
    echo $task->name;
}
```

### tasks 关联关系

首先，我们在 User 模型中定义 tasks 关联关系。Eloquent 关联关系被定义成模型的方法，并且支持多种不同的关联关系类型（查看完整的 Eloquent 关联关系文档了解更多）。在本例中，我们将会在 User 模型中定义 tasks 方法并在其中调用 Eloquent 提供的 hasMany 方法：

```
<?php

namespace App;
// Namespace Imports...
class User extends Model implements AuthenticatableContract,
AuthorizableContract,CanResetPasswordContract
{
    use Authenticatable, Authorizable, CanResetPassword;

    // Other Eloquent Properties...

    /**
     * Get all of the tasks for the user.
     */
    public function tasks()
    {
        return $this->hasMany(Task::class);
    }
}
```

### user 关联关系

接下来，我们会在 Task 模型中定义 user 关联关系。同样，我们将其定义为模型的方法。在本例中，我们使用 Eloquent 提供的 belongsTo 方法来定义该关联关系：

```
<?php

namespace App;

use App\User;
use Illuminate\Database\Eloquent\Model;

class Task extends Model{
    /**
     * The attributes that are mass assignable.
     *
     * @var array
     */
    protected $fillable = ['name'];

    /**
     * Get the user that owns the task.
     */
    public function user()
    {
        return $this->belongsTo(User::class);
    }
}
```

好极了！现在我们已经定义好了关联关系，接下来可以正式开始创建控制器了！

# 3、路由
在新手入门指南创建的任务管理系统中，我们在 routes.php 中使用闭包定义所有的业务逻辑。而实际上，大部分应用都会使用控制器来组织路由。

## 显示视图

我们还保留一个路由使用闭包：/路由，该路由是用于展示给游客的引导页，我们将在该路由中渲染欢迎页面。

在 Laravel 中，所有的 HTML 模板都位于 resources/views 目录，并且我们使用 view 函数从路由中返回其中一个模板：

```
Route::get('/', function () {
    return view('welcome');
});
```

当然，我们需要创建这个视图，稍后就会。

## 登录认证

此外，我们还要让用户注册并登录到应用。通常，在 web 应用中构建整个登录认证层是一件相当冗长乏味的工作，然而，由于它是一个如此通用的需求，Laravel 试图将这一过程变得简单而轻松。

首先，注意到新安装的 Laravel 应用中已经包含了 app/Http/Controllers/AuthController 这个控制器，该控制器中使用了一个特殊的 AuthenticatesAndRegistersUsers trait，而这个 trait 中包含了用户注册登录的所必须的相关逻辑。

### 认证路由

那么接下来我们该怎么做呢？我们仍然需要创建注册和登录模板并定义指向认证控制器 AuthController 的路由。首先，添加我们需要的路由到 app/Http/routes.php 文件：

```
// Authentication Routes...
Route::get('auth/login', 'Auth\AuthController@getLogin');
Route::post('auth/login', 'Auth\AuthController@postLogin');
Route::get('auth/logout', 'Auth\AuthController@getLogout');
// Registration Routes...
Route::get('auth/register', 'Auth\AuthController@getRegister');
Route::post('auth/register', 'Auth\AuthController@postRegister');
```

### 认证视图

完成认证还需要我们在 resources/views/auth 目录下创建 login.blade.php 和 register.blade.php，当然，这些个页面的设计和样式并不重要，只是需要包含一些基本的字段即可。

register.blade.php 文件需要一个包含 name、email、password 和 password_confirmation 字段的表单，该表单请求方式为 POST，请求页面路由为/auth/register。

login.blade.php 文件应该需要一个包含 email 和 password 字段的表单，该表单请求方式为 POST，请求页面路由为/auth/login。

注：如果你想要查看这些视图的完整示例，可以去下载相应的 GitHub 项目：https://github.com/laravel/quickstart-intermediate
## 任务控制器

由于我们需要获取和保存任务，所以还需要使用 Artisan 命令创建一个 TaskController，生成的控制器位于 app/Http/Controllers 目录：

```
php artisan make:controller TaskController --plain
```

现在这个控制器已经生成了，让我们去 app/Http/routes.php 中定义一些指向该控制器的路由吧：

```
Route::get('/tasks', 'TaskController@index');
Route::post('/task', 'TaskController@store');
Route::delete('/task/{task}', 'TaskController@destroy');
```

### 设置所有任务路由需要登录才能访问

对本应用而言，我们想要所有任务需要登录用户才能访问，换句话说，用户必须登录到系统才能创建新任务。所以，我们需要限制访问任务路由的用户为登录用户。Laravel 使用中间件来处理这种限制。

如果要限制登录用户才能访问该控制器的所有动作，可以在控制器的构造函数中添加对 middleware 方法的调用。所有有效的路由中间件都定义在 app/Http/Kernel.php 文件中。在本例中，我们想要定义一个 auth 中间件到 TaskController 上的所有动作：

```
<?php

namespace App\Http\Controllers;

use App\Http\Requests;
use Illuminate\Http\Request;
use App\Http\Controllers\Controller;

class TaskController extends Controller{
    /**
     * Create a new controller instance.
     *
     * @return void
     */
    public function __construct()
    {
        $this->middleware('auth');
    }
}
```

# 4、创建布局&视图
本应用仍然只有一个视图，该视图包含了用于添加新任务的表单和显示已存在任务的列表。为了让你更直观的查看该视图，我们将已完成的应用截图如下：

![](images/basic-overview.png)

## 定义布局

几乎所有的 web 应用都会在不同界面共享同一布局，例如，本应用顶部的导航条将会在每个页面显示。Laravel 使用 Blade 让不同页面共享这些公共特性变得简单。

正如我们之前讨论的，所有 Laravel 视图都存放在 resources/views 中，因此，我们在 resources/views/layouts/app.blade.php 中定义一个新的布局视图，.blade.php 扩展表明框架使用 Blade 模板引擎来渲染视图，当然，你可以使用原生的 PHP 模板，然而，Blade 提供了的标签语法可以帮助我们编写更加清爽、简短的模板。

编辑 app.blade.php 内容如下：

```
// resources/views/layouts/app.blade.php
<!DOCTYPE html><html lang="en">
    <head>
        <title>Laravel Quickstart - Advanced</title>

        <!-- CSS And JavaScript -->
    </head>

    <body>
        <div class="container">
            <nav class="navbar navbar-default">
                <!-- Navbar Contents -->
            </nav>
        </div>

        @yield('content')
    </body>
</html>
```

注意布局中的@yield('content')部分，这是一个 Blade 指令，用于指定继承布局的子页面在这里可以注入自己的内容。接下来，我们来定义使用该布局的子视图来提供主体内容。

## 定义子视图

好了，我们已经创建了应用的布局视图，下面我们需要定义一个包含创建新任务的表单和已存在任务列表的视图，该视图文件存放在 resources/views/tasks.blade.php，对应 TaskController 中的 index 方法。

我们将跳过 Bootstrap CSS 的样板文件而只专注在我们所关注的事情上，不要忘了，你可以从 GitHub 下载本应用的所有资源：

```
// resources/views/tasks.blade.php

@extends('layouts.app')

@section('content')

    <!-- Bootstrap Boilerplate... -->

    <div class="panel-body">
        <!-- Display Validation Errors -->
        @include('common.errors')

        <!-- New Task Form -->
        <form action="/task" method="POST" class="form-horizontal">
            {{ csrf_field() }}

            <!-- Task Name -->
            <div class="form-group">
                <label for="task" class="col-sm-3 control-label">Task</label>

                <div class="col-sm-6">
                    <input type="text" name="name" id="task-name" class="form-control">
                </div>
            </div>

            <!-- Add Task Button -->
            <div class="form-group">
                <div class="col-sm-offset-3 col-sm-6">
                    <button type="submit" class="btn btn-default">
                        <i class="fa fa-plus"></i> Add Task
                    </button>
                </div>
            </div>
        </form>
    </div>

    <!-- TODO: Current Tasks -->
@endsection
```

### 一些需要解释的地方

在继续往下之前，让我们简单谈谈这个模板。首先，我们使用@extends 指令告诉 Blade 我们要使用定义在 resources/views/layouts/app.blade.php 的布局，所有@section('content')和@endsection 之间的内容将会被注入到 app.blade.php 布局的@yield('contents')指令位置。

现在，我们已经为应用定义了基本的布局和视图，然后我们回到 TaskController 的 index 方法：

```
/**
 * Display a list of all of the user's task.
 *
 * @param Request $request
 * @return Response
 */
public function index(Request $request){
    return view('tasks.index');
}
```

接下来，让我们继续添加代码到 POST /task 路由的控制器方法来处理表单输入并添加新任务到数据库。

注：@include('common.errors')指令将会加载 resources/views/common/errors.blade.php 模板中的内容，我们还没有定义这个模板，但很快就会了！
# 6、添加任务
## 验证表单输入

现在我们已经在视图中定义了表单，接下来需要编写代码到 TaskController@store 方法来处理表单请求并创建一个新任务。

对这个表单而言，我们将 name 字段设置为必填项，而且长度不能超过255个字符。如果表单验证失败，将会跳转到/tasks 页面，并且将错误信息存放到一次性 session 中：

```
/**
 * Create a new task.
 *
 * @param Request $request
 * @return Response
 */
public function store(Request $request){
    $this->validate($request, [
        'name' => 'required|max:255',
    ]);

    // Create The Task...
}
```

如果你已经看过新手入门教程，那么你可能会注意到这里的验证代码与之前大不相同，这是因为我们现在在控制器中，可以方便地调用 ValidatesRequests trait（包含在 Laravel 控制器基类中）提供的 validate 方法。

我们甚至不需要手动判读是否验证失败然后重定向。如果验证失败，用户会自动被重定向到来源页面，而且错误信息也会被存放到一次性 Session 中。简直太棒了，有木有！

### $errors 变量

我们在视图中使用了@include('common.errors')指令来渲染表单验证错误信息，common.errors 允许我们在所有页面以统一格式显示错误信息。我们定义 common.errors 内容如下：

```
// resources/views/common/errors.blade.php

@if (count($errors) > 0)
    <!-- Form Error List -->
    <div class="alert alert-danger">
        <strong>Whoops! Something went wrong!</strong>

        <br><br>

        <ul>
        @foreach ($errors->all() as $error)
            <li>{{ $error }}</li>
        @endforeach
        </ul>
    </div>
@endif
```

注：$errors 变量在每个 Laravel 视图中都可以访问，如果没有错误信息的话它就是一个空的 ViewErrorBag 实例。
## 创建任务

现在输入验证已经做好了，接下来正式开始创建一个新任务。一旦新任务创建成功，页面会跳转到/ tasks。要创建任务，可以借助 Eloquent 模型的关联关系。

大部分 Laravel 的关联关系提供了 save 方法，该方法接收一个关联模型实例并且会在保存到数据库之前自动设置外键值到关联模型上。在本例中，save 方法会自动将当前用户登录认证用户的 ID 赋给给给定任务的 user_id 属性。我们通过$request->user()获取当前登录用户实例：

```
/**
 * Create a new task.
 *
 * @param Request $request
 * @return Response
 */
public function store(Request $request){
    $this->validate($request, [
        'name' => 'required|max:255',
    ]);

    $request->user()->tasks()->create([
        'name' => $request->name,
    ]);

    return redirect('/tasks');
}
```

很好，到了这里，我们已经可以成功创建任务，接下来，我们继续添加代码到视图来显示所有任务列表。

# 7、显示已存在的任务
首先，我们需要编辑 TaskController@index 传递所有已存在任务到视图。view 函数接收一个数组作为第二个参数，我们可以将数据通过该数组传递到视图中：

```
/**
 * Display a list of all of the user's task.
 *
 * @param Request $request
 * @return Response
 */
public function index(Request $request){
    $tasks = Task::where('user_id', $request->user()->id)->get();

    return view('tasks.index', [
        'tasks' => $tasks,
    ]);
}
```

这里，我们还要讲讲 Laravel 的依赖注入，这里我们将 TaskRepository 注入到 TaskController，以方便对 Task 模型所有数据的访问和使用。

## 依赖注入

Laravel 的服务容器是整个框架中最重要的特性，在看完快速入门教程后，建议去研习下服务容器的文档。

### 创建 Repository

正如我们之前提到的，我们想要定义一个 TaskRepository 来处理所有对 Task 模型的数据访问，随着应用的增长当你需要在应用中共享一些 Eloquent 查询时这就变得特别有用。

因此，我们创建一个 app/Repositories 目录并在其中创建一个 TaskRepository 类。记住，Laravel 项目的 app 文件夹下的所有目录都使用 PSR-4 自动加载标准被自动加载，所以你可以在其中随心所欲地创建需要的目录：

```
<?php

namespace App\Repositories;

use App\User;
use App\Task;

class TaskRepository{
    /**
     * Get all of the tasks for a given user.
     *
     * @param User $user
     * @return Collection
     */
    public function forUser(User $user)
    {
        return Task::where('user_id', $user->id)
            ->orderBy('created_at', 'asc')
            ->get();
    }
}
```

### 注入 Repository

Repository 创建好了之后，可以简单通过在 TaskController 的构造函数中以类型提示的方式注入该 Repository，然后就可以在 index 方法中使用 —— 由于 Laravel 使用容器来解析所有控制器，所以我们的依赖会被自动注入到控制器实例：

```
<?php

namespace App\Http\Controllers;

use App\Task;use App\Http\Requests;
use Illuminate\Http\Request;
use App\Http\Controllers\Controller;
use App\Repositories\TaskRepository;

class TaskController extends Controller{
    /**
     * The task repository instance.
     * 
     * @var TaskRepository
     */  
    protected $tasks;

    /**
     * Create a new controller instance.
     *
     * @param TaskRepository $tasks 
     * @return void
     */
    public function __construct(TaskRepository $tasks)
    {
        $this->middleware('auth');
        $this->tasks = $tasks;
    }

    /**
     * Display a list of all of the user's task.
     *
     * @param Request $request
     * @return Response
     */
    public function index(Request $request)
    {
        return view('tasks.index', [
            'tasks' => $this->tasks->forUser($request->user()),
        ]);
    }
}
```

## 显示任务

数据被传递到视图后，我们可以在 tasks/index.blade.php 中以表格形式显示所有任务。Blade 中使用@foreach 处理循环数据：

```
@extends('layouts.app')

@section('content')
     <!-- Create Task Form... -->

    <!-- Current Tasks -->
    @if (count($tasks) > 0)
        <div class="panel panel-default">
            <div class="panel-heading">
                Current Tasks
            </div>

            <div class="panel-body">
                <table class="table table-striped task-table">

                <!-- Table Headings -->
                <thead>
                    <th>Task</th>
                    <th>&nbsp;</th>
                </thead>

                <!-- Table Body -->
                <tbody>
                @foreach ($tasks as $task)
                    <tr>
                        <!-- Task Name -->
                        <td class="table-text">
                            <div>{{ $task->name }}</div>
                        </td>

                        <td>
                            <!-- TODO: Delete Button -->
                        </td>
                    </tr>
                @endforeach
                </tbody>
                </table>
            </div>
        </div>
    @endif
@endsection
```

至此，本应用基本完成。但是，当任务完成时我们还没有途径删除该任务，接下来我们就来处理这件事。

# 7、删除任务
## 添加删除按钮

我们在 tasks/index.blade.php 视图中留了一个“TODO”注释用于放置删除按钮。当删除按钮被点击时，DELETE /task 请求被发送到应用后台并触发 TaskController@destroy 方法：

```
<tr>
    <!-- Task Name -->
    <td class="table-text">
        <div>{{ $task->name }}</div>
    </td>

    <!-- Delete Button -->
    <td>
        <form action="/task/{{ $task->id }}" method="POST">
            {{ csrf_field() }}
            {{ method_field('DELETE') }}

            <button>Delete Task</button>
        </form>
    </td>
</tr>
```

### 关于方法伪造

尽管我们使用的路由是 Route::delete，但我们在删除按钮表单中使用的请求方法为 POST，HTML 表单只支持 GET 和 POST 两种请求方式，因此我们需要使用某种方式来伪造 DELETE 请求。

我们可以在表单中通过输出 method_field('DELETE')来伪造 DELETE 请求，该函数生成一个隐藏的表单输入框，然后 Laravel 识别出该输入并使用其值覆盖实际的 HTTP 请求方法。生成的输入框如下：

```
<input type="hidden" name="_method" value="DELETE">
```

## 路由模型绑定

现在，我们准备在 TaskController 中定义 destroy 方法，但是，在此之前，让我们回顾下路由中对删除任务的定义：

```
Route::delete('/task/{task}', 'TaskController@destroy');
```

在没有添加任何额外代码的情况下，Laravel 会自动注入给定任务 ID 到 TaskController@destroy，就像这样：

```
/**
 * Destroy the given task.
 *
 * @param Request $request
 * @param string $taskId
 * @return Response
 */
public function destroy(Request $request, $taskId){
    //
}
```

然而，在这个方法中首当其冲需要处理的就是通过给定 ID 从数据库中获取对应的 Task 实例，因此，如果 Laravel 能够从一开始注入与给定 ID 匹配的 Task 实例岂不是很好？下面就让我们来实现这个！

在 app/Providers/RouteServiceProvider.php 文件的 boot 方法中，我们添加如下这行代码：

```
$router->model('task', 'App\Task');
```

这一小行代码将会告知 Laravel一旦在路由声明中找到{task}，就会获取与给定 ID 匹配的 Task 模型。现在我们可以像这样定义 destroy 方法：

```
/**
 * Destroy the given task.
 *
 * @param Request $request
 * @param Task $task
 * @return Response
 */
public function destroy(Request $request, Task $task){
    //
}
```

## 权限&授权

现在，我们已经将 Task 实例注入到 destroy 方法；然而，我们并不能保证当前登录认证用户是给定任务的实际拥有人。例如，一些恶意请求可能尝试通过传递随机任务 ID 到/tasks/{task}链接删除另一个用户的任务。因此，我们需要使用 Laravel 的授权功能来确保当前登录用户拥有对注入到路由中的 Task 实例进行删除的权限。

### 创建 Policy

Laravel 使用“策略”来将授权逻辑组织到单个类中，通常，每个策略都对应一个模型，因此，让我们使用 Artisan 命令创建一个 TaskPolicy，生成的文件位于 app/Policies/TaskPolicy.php：

```
php artisan make:policy TaskPolicy
```

接下来，让我们添加 destroy 方法到策略中，该方法会获取一个 User 实例和一个 Task 实例。该方法简单检查用户 ID 和任务的 user_id 值是否相等。实际上，所有的策略方法都会返回 true 或 false：

```
<?php

namespace App\Policies;

use App\User;
use App\Task;
use Illuminate\Auth\Access\HandlesAuthorization;

class TaskPolicy{
    use HandlesAuthorization;

    /**
     * Determine if the given user can delete the given task.
     *
     * @param User $user
     * @param Task $task
     * @return bool
     */
    public function destroy(User $user, Task $task)
    {
        return $user->id === $task->user_id;
    }
}
```

最后，我们需要关联 Task 模型和 TaskPolicy，这可以通过在 app/Providers/AuthServiceProvider.php 文件的 policies 属性中添加注册来实现，注册后会告知 Laravel 无论何时我们尝试授权动作到 Task 实例时该使用哪个策略类进行判断：

```
/**
 * The policy mappings for the application.
 *
 * @var array
 */
protected $policies = [
    'App\Task' => 'App\Policies\TaskPolicy',
];
```

### 授权动作

现在我们编写好了策略，让我们在 destroy 方法中使用它。所有的 Laravel 控制器中都可以调用 authorize 方法，该方法由 AuthorizesRequest trait 提供：

```
/**
 * Destroy the given task.
 *
 * @param Request $request
 * @param Task $task
 * @return Response
 */
public function destroy(Request $request, Task $task){
    $this->authorize('destroy', $task);
    // Delete The Task...
}
```

我们可以检查下该方法调用：传递给 authorize 方法的第一个参数是我们想要调用的策略方法名，第二个参数是当前操作的模型实例。由于我们在之前告诉过 Laravel，Task 模型对应的策略类是 TaskPolicy，所以框架知道触发哪个策略类上的 destroy 方法。当前用户实例会被自动传递到策略方法，所以我们不需要手动传递。

如果授权成功，代码会继续执行。如果授权失败，会抛出一个403异常并显示一个错误页面给用户。

注：除此之外，Laravel 还提供了其它授权服务实现方式，可以查看授权文档了解更多。
## 删除任务

最后，让我们添加业务逻辑到路由中执行删除操作，我们可以使用 Eloquent 提供的 delete 方法从数据库中删除给定的模型实例。记录被删除后，跳转到/tasks 页面：

```
/**
 * Destroy the given task.
 *
 * @param Request $request
 * @param Task $task
 * @return Response
 */
public function destroy(Request $request, Task $task){
    $this->authorize('destroy', $task);
    $task->delete();
    return redirect('/tasks');
}
```