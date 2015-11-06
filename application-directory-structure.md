# 应用目录结构

# 1、简介
Laravel 应用默认的目录结构试图为不管是大型应用还是小型应用提供一个好的起点，当然，你可以自己按照喜好重新组织应用目录结构，Laravel 对类在何处被加载没有任何限制——只要 Composer 可以自动载入它们即可。
# 2、根目录
新安装的 Laravel 应用包含许多文件夹：
app 目录包含了应用的核心代码；
bootstrap 目录包含了少许文件用于框架的启动和自动载入配置，还有一个 cache 文件夹用于包含框架生成的启动文件以提高性能；
config 目录包含了应用所有的配置文件；
database 目录包含了数据迁移及填充文件，如果你喜欢的话还可以将其作为 SQLite 数据库存放目录；
public 目录包含了前端控制器和资源文件（图片、js、css 等）；
resources 目录包含了视图文件及原生资源文件（LESS、SASS、CoffeeScript），以及本地化文件；
storage 目录包含了编译过的 Blade 模板、基于文件的 session、文件缓存，以及其它由框架生成的文件，该文件夹被隔离成 app、framework 和 logs 目录，app 目录用于存放应用要使用的文件，framework 目录用于存放框架生成的文件和缓存，最后，logs 目录包含应用的日志文件；
tests 目录包含自动化测试，其中已经提供了一个开箱即用的 PHPUnit 示例；
vendor 目录包含 Composer 依赖；
# 3、App 目录
应用的核心代码位于 app 目录下，默认情况下，该目录位于命名空间 App 下，  并且被 Composer 通过 PSR-4 自动载入标准自动加载。你可以通过 Artisan 命令 app:name 来修改该命名空间。
app 目录下包含多个子目录，如 Console、Http、Providers 等。Console 和 Http 目录提供了进入应用核心的 API，HTTP 协议和 CLI 是和应用进行交互的两种机制，但实际上并不包含应用逻辑。换句话说，它们只是两个向应用发布命令的方式。Console 目录包含了所有的 Artisan 命令，Http 目录包含了控制器、过滤器和请求等。
Jobs 目录是放置队列任务的地方，应用中的任务可以被队列化，也可以在当前请求生命周期内同步执行。
Events 目录是放置事件类的地方，事件可以用于通知应用其它部分给定的动作已经发生，并提供灵活的解耦的处理。
Listeners 目录包含事件的处理器类，处理器接收一个事件并提供对该事件发生后的响应逻辑，比如，UserRegistered 事件可以被 SendWelcomeEmail 监听器处理。
Exceptions 目录包含应用的异常处理器，同时还是处理应用抛出的任何异常的好地方。
注意：app 目录中的很多类都可以通过 Artisan 命令生成，要查看所有有效的命令，可以在终端中运行 php artisan list make 命令。
# 4、设置应用的命令空间
上面已经讨论过，应用默认的命名空间是 App；当然你可以修改该命名空间以匹配应用的名字，修改命名空间可以通过 app:name 命令来实现。比如，如果你的应用取名叫”SocialNet“，那么可以运行如下命令：

```
php artisan app:name SocialNet
```

当然，你也可以继续使用 App 命名空间不做修改。