# Artisan 控制台

## 1、简介
`Artisan` 是 `Laravel` 自带的`命令`行接口名称，它为你在开发过程中提供了很多有用的命令。通过强大的 Symfony Console 组件驱动。想要查看所有可用的 Artisan 命令，可使用 `list` 命令：

```
php artisan list
```

每个命令都可以用 `help` 指令显示命令描述及命令参数和选项。想要查看帮助界面，只需要在命令前加上 `help` 就可以了：

```
php artisan help migrate
```

## 2、编写命令
除了 Artisan 提供的命令之外，还可以构建自己的命令。你可以将自定义命令存放在 app/Console/Commands 目录；当然，你可以自己选择存放位置，只要改命令可以基于 `composer.json` 被自动加载。
要创建一个新命令，你可以使用 Artisan 命令 `make:console`：

```
php artisan make:console SendEmails
```

上述命令将会生成一个类 `app/Console/Commands/SendEmails.php`，当创建命令时，`--command` 选项可用于分配终端命令名（在终端调用命令时用）：

```
php artisan make:console SendEmails --command=emails:send
```

### 2.1 命令结构
命令生成以后，需要填写该类的 `signature` 和 `description` 属性，这两个属性在调用 `list` 显示命令的时候会被用到。
`handle` 方法在命令执行时被调用，你可以将所有命令逻辑都放在这个方法里面，让我们先看一个命令例子。
我们可以在命令控制器的构造函数中注入任何依赖，Laravel服务提供者将会在构造函数中自动注入所有依赖类型提示。要增强代码的复用性，保持代码轻量级并让它们延迟到应用服务中完成任务是个不错的实践：

```
<?php

namespace App\Console\Commands;

use App\User;
use App\DripEmailer;
use Illuminate\Console\Command;
use Illuminate\Foundation\Inspiring;

class Inspire extends Command{
    /**
     * 控制台命令名称
     *
     * @var string
     */
    protected $signature = 'email:send {user}';

    /**
     * 控制台命令描述
     *
     * @var string
     */
    protected $description = 'Send drip e-mails to a user';

    /**
     * The drip e-mail service.
     *
     * @var DripEmailer
     */
    protected $drip;

    /**
     * 创建新的命令实例
     *
     * @param  DripEmailer  $drip
     * @return void
     */
    public function __construct(DripEmailer $drip)
    {
        parent::__construct();
        $this->drip = $drip;
    }

    /**
     * 执行控制台命令
     *
     * @return mixed
     */
    public function handle()
    {
        $this->drip->send(User::find($this->argument('user')));
    }
}
```

## 3、命令I/O 

### 3.1 定义输入异常
编写控制台命令的时候，通常通过参数和选项收集用户输入，Laravel 使这项操作变得很方便：在命令中使用 `signature` 属性来定义我们期望的用户输入。`signature` 属性通过一个优雅的、路由风格的语法允许你定义命令的名称、参数以及选项。所有用户提供的参数和选项都包含在大括号里：

```
/**
 * 控制台命令名称
 *
 * @var string
 */
protected $signature = 'email:send {user}';
```

在本例中，该命令定义了一个必须参数：`user`。你还可以让参数可选化并定义默认的可选参数值：

```
// 选项参数...
email:send {user?}
// 带默认值的选项参数...
email:send {user=foo}
```

选项，和参数一样，也是用户输入的一种格式，不同之处在于选项前面有两个短划线（–），我们可以这样定义选项：

```
/**
 * 控制台命令名称
 *
 * @var string
 */
protected $signature = 'email:send {user} {--queue}';
```

在本例中，--queue 开关在调用 `Artisan` 命令的时候被指定。如果`--queue` 开关被传递，其值时 `true`，否则其值是 `false`：

```
php artisan email:send 1 --queue
```

你还可以指定选项值被用户通过=来分配：

```
/**
 * 控制台命令名称
 *
 * @var string
 */
protected $signature = 'email:send {user} {--queue=}';
```

在本例中，用户可以通过这样的方式传值：

```
php artisan email:send 1 --queue=default
```

还可以给选项分配默认值：

```
email:send {user} {--queue=default}
```

#### 3.1.1 输入描述
你可以通过：分隔参数和描述来分配描述给输入参数和选项：

```
/**
 * 控制台命令名称
 *
 * @var string
 */
protected $signature = 'email:send
    {user : The ID of the user}
    {--queue= : Whether the job should be queued}';
```

### 3.2 获取输入
在命令被执行的时候，很明显，你需要访问命令获取的参数和选项的值。使用 `argument` 和 `option` 方法即可实现：
要获取参数的值，通过 `argument` 方法：

```
/**
 * 执行控制台命令
 *
 * @return mixed
 */
public function handle(){
    $userId = $this->argument('user');
}
```

如果你需要以数组形式获取所有参数值，使用不带参数的 `argument`：

```
$arguments = $this->argument();
```

选项值和参数值的获取一样简单，使用 `option` 方法，同 `argument` 一样如果要获取所有选项值，可以调用不带参数的 `option` 方法：

```
// 获取指定选项...
$queueName = $this->option('queue');
// 获取所有选项...
$options = $this->option();
```

如果参数或选项不存在，返回 null。

### 3.3 输入提示
除了显示输出之外，你可能还要在命令执行期间要用户提供输入。`ask` 方法将会使用给定问题提示用户，接收输入，然后返回用户输入到命令：

```
/**
 * 执行控制台命令
 *
 * @return mixed
 */
public function handle(){
    $name = $this->ask('What is your name?');
}
```

`secret` 方法和 `ask` 方法类似，但用户输入在终端对他们而言是不可见的，这个方法在问用户一些敏感信息如密码时很有用：

```
$password = $this->secret('What is the password?');
```

#### 3.3.1 让用户确认
如果你需要让用户确认信息，可以使用 `confirm` 方法，默认情况下，该方法返回 `false`，如果用户输入 `y`，则该方法返回 `true`：

```
if ($this->confirm('Do you wish to continue? [y|N]')) {
    //
}
```

#### 3.3.2 给用户提供选择
`anticipate` 方法可用于为可能的选项提供自动完成功能，用户仍然可以选择答案，而不管这些选择：

```
$name = $this->anticipate('What is your name?', ['Taylor', 'Dayle']);
```

如果你需要给用户预定义的选择，可以使用 `choice` 方法。用户选择答案的索引，但是返回给你的是答案的值。如果用户什么都没选的话你可以设置默认返回的值：

```
$name = $this->choice('What is your name?', ['Taylor', 'Dayle'], false);
```

### 3.4 编写输出
要将输出发送到控制台，使用 `info`, `comment`, `question` 和 `error`方法，每个方法都会使用相应的 `ANSI` 颜色以作标识。
要显示一条信息消息给用户，使用 `info` 方法。通常，在终端显示为绿色：

```
/**
 * 执行控制台命令
 *
 * @return mixed
 */
public function handle(){
    $this->info('Display this on the screen');
}
```

要显示一条错误消息，使用 `error` 方法。错误消息文本通常是红色：

```
$this->error('Something went wrong!');
```

#### 3.4.1 表格布局
`table` 方法使输出多行/列格式的数据变得简单，只需要将头和行传递给该方法，宽度和高度将基于给定数据自动计算：

```
$headers = ['Name', 'Email'];
$users = App\User::all(['name', 'email'])->toArray();
$this->table($headers, $users);
```

#### 3.4.2 进度条
对需要较长时间运行的任务，显示进度指示器很有用，使用该输出对象，我们可以开始、前进以及停止该进度条。在开始进度时你必须定义定义步数，然后每走一步进度条前进一格：

```
$users = App\User::all();

$this->output->progressStart(count($users));

foreach ($users as $user) {
    $this->performTask($user);
    $this->output->progressAdvance();
}

$this->output->progressFinish();
```

想要了解更多，查看 `Symfony` 进度条组件文档。
## 4、注册命令
命令编写完成后，需要注册到 `Artisan` 才可以使用，这可以在 `app/Console/Kernel.php` 文件中完成。

在该文件中，你会在 `commans` 属性中看到一个命令列表，要注册你的命令，只需将加到该列表中即可。当 `Artisan` 启动的时候，该属性中列出的命令将会被服务容器解析被注册到 `Artisan`：

```
protected $commands = [
    'App\Console\Commands\SendEmails'
];
```

## 5、通过代码调用命令
有时候你可能希望在 `CLI` 之外执行 `Artisan` 命令，比如，你可能希望在路由或控制器中触发 `Artisan` 命令，你可以使用 `Artisan` 门面上的 `call` 方法来完成这个。`call` 方法接收被执行的命令名称作为第一个参数，命令参数数组作为第二个参数，退出代码被返回：

```
Route::get('/foo', function () {
    $exitCode = Artisan::call('email:send', [
        'user' => 1, '--queue' => 'default'
    ]);
});
```

使用 `Artisan` 上的 `queue` 方法，你甚至可以将 `Artisan` 命令放到队列中，这样它们就可以通过后台的队列工作者来处理：

```
Route::get('/foo', function () {
    Artisan::queue('email:send', [
        'user' => 1, '--queue' => 'default'
    ]);
});
```

如果你需要指定不接收字符串的选项值，例如 `migrate:refresh` 命令上的`--force` 标识，可以传递布尔值 `true` 或 `false`：

```
$exitCode = Artisan::call('migrate:refresh', [
    '--force' => true,
]);
```

### 5.1 通过其他命令调用命令
有时候你希望从一个已存在的 `Artisan` 命令中调用其它命令。你可以通过使用 `call` 方法开实现这一目的。`call` 方法接收命令名称和数组形式的命令参数：

```
/**
 * 执行控制台命令
 *
 * @return mixed
 */
public function handle(){
    $this->call('email:send', [
        'user' => 1, '--queue' => 'default'
    ]);
}
```

如果你想要调用其它控制台命令并阻止其所有输出，可以使用 `callSilent` 方法。

```
callSilent 方法和 call 方法用法一致：
$this->callSilent('email:send', [
    'user' => 1, '--queue' => 'default'
]);
```