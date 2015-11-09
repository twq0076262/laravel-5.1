# 升级指南

## 更新到5.1.0
**预计更新时间：小于1小时**

### 更新 bootstrap/autoload.php
更新 bootstrap/autoload.php 中的变量 `$compilePath：`

```
$compiledPath = __DIR__.'/cache/compiled.php';
```

### 创建 bootstrap/cache 目录
在 bootstrap 目录中，创建 cache 目录，在该目录中创建一个 .gitignore 文件，编辑文件内容如下：

```
*!.gitignore
```

该目录应该是可写的，用来存储临时优化文件如 compiled.php，routes.php，config.php 以及 service.json

### 新增 BroadcastServiceProvider
在配置文件 config/app.php 中，添加 Illuminate\Broadcasting\BroadcastServiceProvider 到 providers 数组。

### 认证
如果你在使用 Laravel 自带的 AuthenticatesAndRegistersUsers 的 AuthController，则需要对新用户的验证和创建做一些代码改动：

首先，你不再需要传递 Guard 和 Register 实例到构造函数，你可以从控制器的构造器中完全移除这些以依赖。

然后，Laravel 5.0 中使用的 App\Services\Registrar 不再被需要，你可以直接简单拷贝粘贴其中的 validator 方法和 create 方法到 AuthController 中，这两个方法中的代码不需要做任何改动。不要忘记确认 Validator 和 User 在 AuthController 中是否已经被导入。

PasswordController 不再需要在构造函数中声明任何依赖，可以移除5.0中要求的两个依赖。

### 验证
如果你重写了 Controller 类中的 formatValidationErrors 方法，需要将类型提示由 Illuminate\Validation\Validator 改为 Illuminate\Contracts\Validation\Validator。

### Eloquent
**create 方法**

Eloquent 的 create 方法现在可以不传入任何参数进行调用，如果你在模型中要重写 create 方法，将 `$attributes` 参数的默认值改为数组：

```
public static function create(array $attributes = []){
    // Your custom implementation
}
```

**find 方法**

如果你要在自己的模型中重写 find 方法并在其中调用 parent::find()，应该改由调用 Eloquent 查询构建器的 find 方法：

```
public static function find($id, $columns = ['*']){
    $model = static::query()->find($id, $columns);

    // ...

    return $model;
}
```

**lists 方法**

lists 方法现在返回一个 Collection 实例而不是包含 Eloquent 查询结果的数组，如果你想将 Collection 转化为数组，使用 all 方法：

```
User::lists('id')->all();
```

注意：Query Builder 的 lists 返回的仍然是数组。

**日期格式化**

以前，模型中的 Eloquent 日期字段存储格式可以通过重写 getDateFormat 方法来修改，现在依然可以这么做；但是为了更加方便可以在模型中简单通过指定 `$dateFormat` 属性来替代重写方法。

在序列化模型到数组或 JSON 时日期格式也被应用到，当从 Laravel 5.0 迁移到5.1时，这将会改变 JSON 序列化的日期字段的格式。想要在序列化模型中设置指定的日期格式，你可以在模型中重写 serializeDate(DateTime `$date)`方法，这样就可以在不改变字段存储格式的情况下对格式化序列化的 Eloquent 日期字段有着更加细粒度的控制。

## Collection 类
### sortBy 方法

sortBy 方法现在返回一个新的 collection 实例而不是改变已有的 collection：

```
$collection = $collection->sortBy('name');
```

### groupBy 方法

groupBy 方法现在为每个父级 Collection 中的 item 返回 Collection 实例，如果你想要将这些 items 转化为数组，可以通过 map 方法实现：

```
$collection->groupBy('type')->map(function($item){
    return $item->all();
});
```

### lists 方法

lists 方法现在返回一个 Collection 实例而不是数组，如果你想要将 Collection 转化数组，使用 all 方法：

```
$collection->lists('id')->all();
```

## 命令&处理器
app/Commands 目录现在被重命名为 app/Jobs，但是并不需要将你的命令移动到新位置，你可以继续使用 make:command 和 handler:command Artisan 命令生成自己的类。

同样的，app/Handlers 目录被合并到 app/Listeners 目录下，你也不必将已经存在的命令和事件处理器进行移动和重命名，你可以继续使用 handler:event 命令生成事件处理器。

通过提供对 Laravel 5.0 目录结构的向后兼容，你可以无缝升级应用到 Laravel 5.1 然后慢慢升级你的事件和命令到新的位置——在一个对你或你的团队合适的时间。

## Blade
createMatcher，createOpenMatcher 和 createPlainMatcher 方法已经从 Blade 编译器中移除，可以使用新的 directive 方法来为5.1版的 Blade 创建自定义的指令。查阅扩展 Blade 文档了解详情。

## 测试
在 tests/TestCase.php 文件中新增 protected 属性 `$baseUrl：`

```
protected $baseUrl = 'http://localhost';
```

## 翻译文件
用于为 vendor 包发布语言文件的默认目录做了移动，所有 vendor 包语言文件从 resources/lang/packages/{locale}/{namespace} 移动到了 resources/lang/vendor/{namespace}/{locale} 目录。例如，Acme/Anvil 包的 acme/anvil::foo 英语语言文件将会从 resources/lang/packages/en/acme/anvil/foo.php 移动到 resources/lang/vendor/acme/anvil/en/foo.php。

## Amazon Web Services SDK
如果你正在使用 AWS SQS 队列驱动或者 AWS SES 电子邮件驱动，需要升级 AWS PHP SDK 到3.0版本。

如果你正在使用 Amazon S3 文件系统驱动，需要通过 Composer 升级相应的文件系统包：

- Amazon S3: league/flysystem-aws-s3-v3 ~1.0

## 废弃
以下 Laravel 特性已经被废弃并且会在2015年12月份的 Laravel 5.2 中被完全移除：

- 中间件中的路由过滤器
- Illuminate\Contracts\Routing\Middleware，中间件中不再需要任何 contract，Illuminate\Contracts\Routing\TerminableMiddleware 被废弃，在中间件中定义一个 terminate 方法替代实现该接口。
- Illuminate\Contracts\Queue\ShouldBeQueued 被废弃，使用 Illuminate\Contracts\Queue\ShouldQueue
- Iron.io “推入队列” 被废弃， 使用 Iron.io 队列和队列监听器.
- Illuminate\Foundation\Bus\DispatchesCommands trait 被废弃并被重命名为 Illuminate\Foundation\Bus\DispatchesJobs.
- Illuminate\Container\BindingResolutionException 被移动到 Illuminate\Contracts\Container\BindingResolutionException.
- 服务容器的 bindShared 方法被废弃，使用 singleton 方法。
- Eloquent 和 query builder 的 pluck 方法被废弃并重命名为 value.
- Collection 的 fetch 方法被废弃，使用 pluck 方法.
- array_fetch 帮助函数被废弃， 使用 array_pluck

