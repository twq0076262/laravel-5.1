# Envoy 任务运行器（SSH任务）

## 1、简介
Laravel Envoy 为定义运行在远程主机上的通用任务提供了一套干净、最简化的语法。使用 Blade 样式语法，你可以轻松为开发设置任务，Artisan 命令，以及更多，目前，Envoy 只支持 Mac 和 Linux 操作系统。

### 1.1 安装
首先，使用 Composer 的 `global` 命令安装 Envoy：

```
composer global require "laravel/envoy=~1.0"
```

确保`~/.composer/vendor/bin` 目录在系统路径 PATH 中否则在终端中由于找不到 `envoy` 而无法执行该命令。

#### 1.1.1 更新 Envoy
还可以使用 Composer 保持安装的 Envoy 是最新版本：

```
composer global update
```

## 2、编写任务
所有的 Envoy 任务都定义在项目根目录下的 `Envoy.blade.php `文件中，下面是一个让你开始的示例：

```
@servers(['web' => 'user@192.168.1.1'])

@task('foo', ['on' => 'web'])
    ls -la
@endtask
```

正如你所看到的，`@servers` 数组定义在文件顶部，从而允许你在任务声明中使用 `on `选项引用这些服务器，在` @task` 声明中，应该放置将要在服务器上运行的 Bash 代码。

**启动**
有时候，你需要在评估 Envoy 任务之前执行一些 PHP 代码，可以在 Envoy 文件中使用`@setup` 指令来声明变量和要执行的 PHP 代码：

```
@setup
    $now = new DateTime();
    $environment = isset($env) ? $env : "testing";
@endsetup
```

还可以使用`@include` 来引入外部 PHP 文件：

```
@include('vendor/autoload.php');
```

**确认任务**
如果你想要在服务器上运行给定任务之前弹出弹出提示进行确认，可以在任务声明中使用 `confirm `指令：

```
@task('deploy', ['on' => 'web', 'confirm' => true])
    cd site
    git pull origin {{ $branch }}
    php artisan migrate
@endtask
```

### 2.1 任务变量
如果需要的话，你可以使用命令行开关传递变量到 Envoy 文件，从而允许你自定义任务：

```
envoy run deploy --branch=master
```

你可以在任务中通过 Blade 的“echo”语法使用该选项：

```
@servers(['web' => '192.168.1.1'])

@task('deploy', ['on' => 'web'])
    cd site
    git pull origin {{ $branch }}
    php artisan migrate
@endtask
```

### 2.2 多个服务器
你可以轻松地在多主机上运行同一个任务，首先，添加额外服务器到`@servers` 声明，每个服务器应该被指配一个唯一的名字。定义好服务器后，在任务声明中简单列出所有服务器即可：

```
@servers(['web-1' => '192.168.1.1', 'web-2' => '192.168.1.2'])

@task('deploy', ['on' => ['web-1', 'web-2']])
    cd site
    git pull origin {{ $branch }}
    php artisan migrate
@endtask
```

默认情况下，该任务将会依次在每个服务器上执行，这意味着，该任务在第一台服务器上运行完成后才会开始在第二台服务器运行。

#### 2.2.1 平行运行
如果你想要在多个服务器上平行运行，添加` parallel` 选项到任务声明：

```
@servers(['web-1' => '192.168.1.1', 'web-2' => '192.168.1.2'])

@task('deploy', ['on' => ['web-1', 'web-2'], 'parallel' => true])
    cd site
    git pull origin {{ $branch }}
    php artisan migrate
@endtask
```

## 2.3 任务宏
宏允许你使用单个命令中定义多个依次运行的任务。例如，`deploy `宏会运行 git 和 composer 任务：

```
@servers(['web' => '192.168.1.1'])

@macro('deploy')
    git
    composer
@endmacro

@task('git')
    git pull origin master
@endtask

@task('composer')
    composer install
@endtask
```

宏被定义好了之后，你就可以通过如下单个命令运行它：

```
envoy run deploy
```

# 3、运行任务
要从 `Envoy.blade.php` 文件中运行一个任务，需要执行 Envoy 的` run` 命令，然后传递你要执行的任务的命令名或宏。Envoy 将会运行命令并从服务打印输出：

```
envoy run task
```

## 4、通知

### 4.1 HipChat
运行完一个任务后，可以使用 Envoy 的`@hipchat` 指令发送通知到团队的 HipChat 房间，该指令接收一个 API 令牌、房间名称、和用户名：

```
@servers(['web' => '192.168.1.1'])

@task('foo', ['on' => 'web'])
    ls -la
@endtask

@after
    @hipchat('token', 'room', 'Envoy')
@endafter
```

需要的话，你还可以传递自定义发送给 HipChat 房间的消息，所有在 Envoy 任务中有效的变量在构建消息时也有效：

```
@after
    @hipchat('token', 'room', 'Envoy', "{$task} ran in the {$env} environment.")
@endafter
```

### 4.2 Slack
除了 HipChat 之外，Envoy 还支持发送通知到Slack。`@slack` 指令接收一个 Slack 钩子 URL、频道名称、和你要发送给该频道的消息：

```
@after
    @slack('hook', 'channel', 'message')
@endafter
```

你可以通过创建集成到 Slack 网站的 Incoming WebHooks 来获取钩子 URL，该 hook 参数是由 Incoming Webhooks Slack 集成提供的完整 webhook URL，例如：

```
https://hooks.slack.com/services/ZZZZZZZZZ/YYYYYYYYY/XXXXXXXXXXXXXXX
```

你可以提供下面两种其中之一作为频道参数：

- 	发送消息到频道: `#channel `
- 	发送消息到用户: `@user`