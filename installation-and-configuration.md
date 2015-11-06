# 安装及配置

# 1、安装
## 1.1 服务器要求

Laravel 框架有少量的系统要求，当然，Laravel Homestead 虚拟机满足所有这些要求：

- PHP 版本 >= 5.5.9
- PHP 扩展：OpenSSL
- PHP 扩展：PDO
- PHP 扩展：Mbstring
- PHP 扩展：Tokenizer
## 1.2 安装 Laravel

Laravel 使用 Composer 管理依赖，因此，使用 Laravel 之前，确保机器上已经安装 Composer。

### 1.2.1 通过 Laravel 安装器

首先，通过 Composer 安装 Laravel 安装器：

```
composer global require "laravel/installer=~1.1"
```

确保 ~/.composer/vendor/bin 在系统路径 PATH 中，否则不能调用 larave l命令。

安装完成后，通过简单的 laravel new 命令将会在当前目录下创建一个新的 Laravel 应用，例如，laravel new blog 将会创建一个名为 blog 的 Laravel 安装目录，该目录中已经包含了所有 Laravel 依赖。该安装方法比通过 Composer 安装要快很多：

```
laravel new blog
```

### 1.2.2 通过 Composer

你还可以在终端中通过 Composer 的 create-project 目录来安装 Laravel：

```
composer create-project laravel/laravel --prefer-dist
```

该命令会在当前目录中创建一个名为 laravel 的 Laravel 安装，如果想要指定安装目录名，可通过如下命令：

```
composer create-project laravel/laravel blog --prefer-dist
```

该命令会在当前目录中创建一个名为 blog 的 Laravel 安装。

扩展阅读 —— 实例教程：[在 Windows 中安装 Laravel 5.1.X](http://laravelacademy.org/post/306.html)
# 2、配置
## 2.1 基本配置

Laravel 框架的所有配置文件都存放在 config 目录中，每一个选项都是文档化（有良好注释）的，所以随便浏览所有配置文件去熟悉这些配置选项。

### 2.1.1 目录权限

安装完 Laravel 后，需要配置一些权限。storage 和 bootstrap/cache 目录应该是可写的，如果你在使用 Homestead 虚拟机，这些权限已经被设置好了。

### 2.1.2 应用 Key

接下来要做的事情就是将应用 key 设置为一个随机字符串，如果你是通过 Composer 或者 Laravel 安装器安装的话，该 key 的值已经通过 key:generate 命令生成好了。通常，该字符串应该是32位长，该 key 被配置在 .env 环境文件中（APP_KEY），如果你还没有将 .env.example 文件重命名为 .env，现在立即这样做。如果应用 key 没有被设置，用户 sessions 和其它加密数据将会有安全隐患！

### 2.1.3 更多配置

Laravel 几乎不再需要其它任何配置就可以使用了，你可以自由地开始开发了！但是，你最好再看看 config/app.php 文件和它的文档，其中包含了一些基于你的应用可能需要进行改变的配置，比如 timezone 和 locale。

你可能还想要配置 Laravel 的一些其它组件，比如：

- 缓存
- 数据库
- Session
Laravel 安装完成后，你还应该配置自己的本地环境，如数据库驱动、邮箱服务器、缓存驱动等。

### 2.1.4 美化 URL

- Apache
框架中自带的 public/.htaccess 文件支持 URL 中隐藏 index.php，如过你的 Laravel 应用使用 Apache 作为服务器，需要先确保 Apache 启用了 mod_rewrite 模块以支持 .htaccess 解析。

如果 Laravel 自带的 .htaccess 文件不起作用，试试将其中内容做如下替换：

```
Options +FollowSymLinks
RewriteEngine On

RewriteCond %{REQUEST_FILENAME} !-d
RewriteCond %{REQUEST_FILENAME} !-f
RewriteRule ^ index.php [L]
```

- Nginx
在 Nginx 中，使用如下站点配置指令就可以支持 URL 美化：

```
location / {
    try_files $uri $uri/ /index.php?$query_string;
}
```

当然，使用 Homestead 的话，以上配置已经为你配置好以支持 URL 美化。

## 2.2 环境配置

基于应用运行环境拥有不同配置值能够给我们开发带来极大的方便，比如，我们想在本地和线上环境配置不同的缓存驱动，在 Laravel 中这很容易实现。

Laravel 中使用了 Vance Lucas 开发的 PHP 库 DotEnv 来实现这一目的，在新安装的 Laravel 中，根目录下有一个 .env.example 文件，如果 Laravel 是通过 Composer 安装的，那么该文件已经被重命名为 .env，否则的话你要自己手动重命名该文件。

在每次应用接受请求时，.env 中列出的所有变量都会被载入到 PHP 超全局变量 $_ENV 中，然后你就可以在应用中通过帮助函数 env 来获取这些变量值。实际上，如果你去查看 Laravel 的配置文件，就会发现很多选项已经在使用这些帮助函数了。

你可以尽情的按你所需对本地服务器上的环境变量进行修改，线上环境也是一样。但不要把 .env 文件提交到源码控制（svn 或 git 等）中，因为每个使用你的应用的不同开发者或服务器可能要求不同的环境配置。

如果你是在一个团队中进行开发，你可能需要将 .env.example 文件随你的应用一起提交到源码控制中，通过将一些配置值以占位符的方式放置在 .env.example 文件中，其他开发者可以很清楚明了的知道运行你的应用需要配置哪些环境变量。

### 2.2.1 访问当前应用环境

当前应用环境由 .env 文件中的 APP_ENV 变量决定，你可以通过 App 门面的 environment 方法来访问其值：

```
$environment = App::environment();
```

你也可以向 environment 方法中传递参数来判断当前环境是否匹配给定值，如果需要的话你甚至可以传递多个值：

```
if (App::environment('local')) {
    // The environment is local
}

if (App::environment('local', 'staging')) {
    // The environment is either local OR staging...
}
```

应用实例也可以通过帮助函数 app 来访问：

```
$environment = app()->environment();
```

## 2.3 配置缓存

为了给应用加速，你可以使用 Artisan 命令 config:cache 将所有配置文件合并到单个文件里，这将会将所有配置选项合并到单个文件从而可以被框架快速加载。

你应该将 config:cache 作为日常部署的一部分。

## 2.4 访问配置值

你可以使用全局的帮助函数 config 来访问配置值，配置值可以通过”.”来分隔配置文件和配置选项，如果配置选项不存在的话则会返回默认值：

```
$value = config('app.timezone');
```

如果要在运行时设置配置值，传递一个数组到 config 帮助函数：

```
config(['app.timezone' => 'America/Chicago']);
```

## 2.5 命名你的应用

安装完成 Laravel 之后，你可能想要命名你的应用，默认情况下，app 目录处于命名空间 App 之下，然后 Composer 使用 PSR-4 自动载入标准来自动载入该目录，你可以使用 Artisan 命令 app:name来改变该命名空间以匹配你的应用名称。

比如，如果你的应用名称是“Horsefly”，你可以在安装根目录下运行如下命令：

```
php artisan app:name Horsefly
```

来重命名应用的命名空间，当然你也可以继续使用 App 作为命名空间不变。

# 3、维护模式
当你的站点处于维护模式时，所有对站点的请求都会返回同一个自定义视图。当你在对站点进行升级或者维护时，这使得“关闭”站点变得轻而易举，对维护模式的判断代码位于默认的中间件栈中，如果应用处于维护模式，则状态码为503的 HttpException 将会被抛出。

想要开启维护模式，只需执行 Artisan 命令 down 即可：

```
php artisan down
```

关闭维护模式，对应的 Artisan 命令是 up：

```
php artisan up
```

## 3.1 维护模式响应模板

默认的维护模式响应模板位于 resources/views/errors/503.blade.php

## 3.2 维护模式 & 队列

当你的站点处于维护模式中时，所有的队列任务都不会执行；当应用退出维护模式这些任务才会被继续正常处理。