# 新手入门指南-简单任务管理系统

引言：Laravel 官方终于推出5.1版本快速入门指南了，学院君在 reddit 上看到大家的讨论后才得知这一消息，立即着手进行了翻译，希望对 Laravel 学习者有所帮助。

## 1、简介
快速入门指南会对 Laravel 框架做一个基本介绍，包括数据库迁移、Eloquent ORM、路由、验证、视图以及 Blade 模板等等。如果你是个 Laravel 新手甚至之前对 PHP 框架也很陌生，那么这里将会成为你的良好起点。如果你已经使用过 Laravel 获取其它 PHP 框架，可以考虑跳转到进阶指南（翻译中）。

为了演示 Laravel 特性的基本使用，我们将将会构建一个简单的、用于追踪所有要完成任务的任务列表（To-Do List），本教程完整的代码已经公开在 Github 上：https://github.com/laravel/quickstart-basic。

## 2、安装
当然，开始之前你首先要做的是安装一个新的 Laravel 应用。你可以使用 Homestead 虚拟机或者本地 PHP 开发环境来运行应用。设置好开发环境后，可以使用如下 Composer 命令安装应用：

```
composer create-project laravel/laravel quickstart --prefer-dist
```

当然你还可以通过克隆 GitHub 仓库到本地来安装：

```
git clone https://github.com/laravel/quickstart-basic quickstart
cd quickstart
composer install
php artisan migrate
```

如果你还不了解如何构建本地开发环境，可参考 Homestead 和安装文档。

## 3、准备好数据库

### 数据库迁移

首先，让我们使用迁移来定义数据表用于处理所有任务。Laravel 的数据库迁移特性提供了一个简单的方式来对数据表结构进行定义和修改：不需要让团队的每个成员添加列到本地数据库，只需要简单运行你提交到源码控制中的迁移即可实现数据表创建及修改。

那么，让我们来创建这个处理所有任务的数据表吧。Artisan 命令可以用来生成多种类从而节省重复的劳动，在本例中，我们使用 `make:migration` 命令生成 `tasks` 对应的数据表迁移：

```
php artisan make:migration create_tasks_table --create=tasks
```

该命令生成的迁移文件位于项目根目录下的 `database/migrations` 目录，可能你已经注意到了，`make:migration` 命令已经在迁移文件中为我们添加了自增 ID 和时间戳，接下来我们要编辑该文件添加更多的列到数据表 `tasks`：

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

要运行迁移，可以使用 Artisan 命令 `migrate`。如果你使用的是 Homestead，应该在虚拟机中运行该命令：

```
php artisan migrate
```

该命令会为我们创建迁移文件中定义的所有数据表，如果你使用数据库客户端软件查看数据库，可以看到已经创建了一个新的 `tasks` 表，其中包含了我们在迁移中定义的列。接下来，我们准备为这个数据表定义一个 Eloquent ORM 模型。

### Eloquent 模型

Laravel 使用的默认 ORM 是 Eloquent，Eloquent 使用模型让数据存取变得简单和轻松，通常，每一个 Eloquent 模型都有一个与之对应的数据表。

所以我们要定义一个与刚刚创建的 `tasks` 表对应的 `Task` 模型，同样我们使用 Artisan 命令来生成这个模型：

```
 php artisan make:model Task
```

该模型类位于 `app` 目录下，默认情况下，模型类是空的，我们不需要告诉该 Eloquent 模型对应哪张数据表，这一点我们在 Eloquent 文档中提及过，这里默认对应的数据表是 `tasks`，下面是这个空的模型类：

```
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class Task extends Model{
    //
}
```

了解更多关于 Eloquent 模型类的细节，可查看完整的 Eloquent 文档。

## 4、路由

### 创建路由

下面我们需要为应用定义一些路由，路由的作用是在用户访问指定页面时将页面URL匹配到被执行的控制器或匿名函数。默认情况下，所有的 Laravel 路由都定义在 `app/Http/routes.php`。

在本应用中，我们需要至少三个路由：显示所有任务的路由，添加新任务的路由，以及删除已存在任务的路由。接下来，让我们在 `app/Http/routes.php` 文件中创建这三个路由：

```
<?php

use App\Task;
use Illuminate\Http\Request;

/**
 * Display All Tasks
 */
Route::get('/', function () {
   //
});

/**
 * Add A New Task
 */
Route::post('/task', function (Request $request) {
    //
});

/**
 * Delete An Existing Task
 */
Route::delete('/task/{id}', function ($id) {
    //
});
```

### 显示视图

接下来，我们来填充/路由，在这个路由中，我们要渲染一个 HTML 模板，该模板包含添加新任务的表单，以及显示任务列表。

在 Laravel 中，所有的 HTML 模板都存放在 `resources/views` 目录下，我们可以使用 `view` 函数从路由中返回其中一个模板：

```
Route::get('/', function () {
    return view('tasks');
});
```

当然，接下来我们需要去创建这个视图。

## 5、创建布局&视图

本应用为了简单处理只包含一个视图，其中包含了添加新任务的表单和所有任务的列表。为了让大家有一个直观的视觉效果，我们贴出该视图的截图，可以看到我们在视图中使用了基本的 Bootstrap CSS 样式：

![](images/basic-overview.png)


### 定义布局

几乎所有的 web 应用都是在不同页面中共享同一个布局，例如，本应用在视图顶部有一个导航条，该导航条在每个页面都会出现。Laravel 通过在每个页面中使用 Blade 布局让共享这些公共特性变得简单。

正如我们之前讨论的，所有 Laravel 视图都存放在 `resources/views` 中，因此，我们在 `resources/views/layouts/app.blade.php` 中定义一个新的布局视图，`.blade.php` 扩展表明框架使用 Blade 模板引擎来渲染视图，当然，你可以使用原生的 PHP 模板，然而，Blade 提供了的标签语法可以帮助我们编写更加清爽、简短的模板。

编辑 `app.blade.php` 内容如下：

```
// resources/views/layouts/app.blade.php
<!DOCTYPE html><html lang="en">
    <head>
        <title>Laravel Quickstart - Basic</title>

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

注意布局中的 `@yield('content')`部分，这是一个 Blade 指令，用于指定继承布局的子页面在这里可以注入自己的内容。接下来，我们来定义使用该布局的子视图来提供主体内容。

### 定义子视图

好了，我们已经创建了应用的布局视图，下面我们需要定义一个包含创建新任务的表单和已存在任务列表的视图，该视图文件存放在 `resources/views/tasks.blade.php`。

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

在继续往下之前，让我们简单谈谈这个模板。首先，我们使用 `@extends` 指令告诉 Blade 我们要使用定义在 `resources/views/layouts/app.blade.php` 的布局，所有 `@section('content')`和`@endsection` 之间的内容将会被注入到 `app.blade.php` 布局的 `@yield('contents')`指令位置。

现在，我们已经为应用定义了基本的布局和视图，接下来，我们准备添加代码到 `POST /task` 路由来处理添加新任务到数据库。

注：`@include('common.errors')`指令将会加载 `resources/views/common/errors.blade.php` 模板中的内容，我们还没有定义这个模板，但很快就会了！

## 6、添加任务

### 验证表单输入

现在我们已经在视图中定义了表单，接下来需要编写代码处理表单请求，我们需要验证表单输入，然后才能创建一个新任务。

对这个表单而言，我们将 `name` 字段设置为必填项，而且长度不能超过255个字符。如果表单验证失败，将会跳转到前一个页面，并且将错误信息存放到一次性 session 中：

```
Route::post('/task', function (Request $request) {
    $validator = Validator::make($request->all(), [
        'name' => 'required|max:255',
    ]);

    if ($validator->fails()) {
        return redirect('/')
            ->withInput()
            ->withErrors($validator);
    }

    // Create The Task...
});
```

### `$errors` 变量

让我们停下来讨论下上述代码中的`->withErrors($validator)`部分，`->withErrors($validator)`会将验证错误信息存放到一次性 session 中，以便在视图中可以通过 `$errors` 变量访问。

我们在视图中使用了 `@include('common.errors')`指令来渲染表单验证错误信息，`common.errors` 允许我们在所有页面以统一格式显示错误信息。我们定义 `common.errors` 内容如下：

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


注：`$errors` 变量在每个 Laravel 视图中都可以访问，如果没有错误信息的话它就是一个空的 `ViewErrorBag` 实例。

### 创建任务

现在输入验证已经做好了，接下来正式开始创建一个新任务。一旦新任务创建成功，页面会跳转到`/`。要创建任务，可以使用 Eloquent 模型提供的 `use` 方法：

```
Route::post('/task', function (Request $request) {
    $validator = Validator::make($request->all(), [
        'name' => 'required|max:255',
    ]);

    if ($validator->fails()) {
        return redirect('/')
            ->withInput()
            ->withErrors($validator);
    }

    $task = new Task;
    $task->name = $request->name;
    $task->save();

    return redirect('/');
});
```

好了，到了这里，我们已经可以成功创建任务，接下来，我们继续添加代码到视图来显示所有任务列表。

### 显示已存在的任务

首先，我们需要编辑/路由传递所有已存在任务到视图。`view` 函数接收一个数组作为第二个参数，我们可以将数据通过该数组传递到视图中：

```
Route::get('/', function () {
    $tasks = Task::orderBy('created_at', 'asc')->get();

    return view('tasks', [
        'tasks' => $tasks
    ]);
});
```

数据被传递到视图后，我们可以在 `tasks.blade.php` 中以表格形式显示所有任务。Blade 中使用`@foreach` 处理循环数据：

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

## 7、删除任务

### 添加删除按钮

我们在 `tasks.blade.php` 视图中留了一个“TODO”注释用于放置删除按钮。当删除按钮被点击时， `DELETE /task` 请求被发送到应用后台：

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

#### 关于方法伪造

尽管我们使用的路由是 `Route::delete`，但我们在删除按钮表单中使用的请求方法为 POST，HTML 表单只支持 `GET` 和 `POST` 两种请求方式，因此我们需要使用某种方式来伪造 DELETE 请求。

我们可以在表单中通过输出 `method_field('DELETE')`来伪造 `DELETE` 请求，该函数生成一个隐藏的表单输入框，然后 Laravel 识别出该输入并使用其值覆盖实际的 HTTP 请求方法。生成的输入框如下：

```
<input type="hidden" name="_method" value="DELETE">
```

### 删除任务

最后，让我们添加业务逻辑到路由中执行删除操作，我们可以使用 Eloquent 提供的 `findOrFail` 方法从数据库通过 ID 获取模型实例，如果不存在则抛出404异常。获取到模型后，我们使用模型的 `delete` 方法删除该模型在数据库中对应的记录。记录被删除后，跳转到`/`页面：

```
Route::delete('/task/{id}', function ($id) {
    Task::findOrFail($id)->delete();
    return redirect('/');
});
```