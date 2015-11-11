# 包开发

## 1、简介
包是添加功能到 Laravel 的主要方式。包可以提供任何功能，小到处理日期如 Carbon，大到整个 BDD 测试框架如 Behat。

当然，有很多不同类型的包。有些包是独立的，意味着可以在任何框架中使用，而不仅是 Laravel。比如 Carbon 和 Behat 都是独立的包。所有这些包都可以通过在 `composer.json` 文件中请求以便被 Laravel 使用。

另一方面，其它包只能特定和 Laravel 一起使用，这些包可能有路由，控制器、视图和配置用于加强 Laravel 应用的功能，本章主要讨论只能在 Laravel 中使用的包。

## 2、服务提供者
服务提供者是包和 Laravel 之间的连接点。服务提供者负责绑定对象到 Laravel 的服务容器并告知 Laravel 从哪里加载包资源如视图、配置和本地化文件。

服务提供者继承自 `Illuminate\Support\ServiceProvider` 类并包含两个方法：`register` 和 `boot`。`ServiceProvider` 基类位于 Composer 包 `illuminate/support`。

要了解更多关于服务提供者的内容，查看其文档。

## 3、路由
要定义包的路由，只需要在包服务提供者中的 `boot` 方法中引入路由文件。在路由文件中，可以使用 `Route` 门面注册路由，和 Laravel 应用中注册路由一样：

```
/**
 * Perform post-registration booting of services.
 *
 * @return void
 */
public function boot(){
    if (! $this->app->routesAreCached()) {
        require __DIR__.'/../../routes.php';
    }
}
```

## 4、资源

### 4.1 视图
要在 Laravel 中注册包视图，需要告诉 Laravel 视图在哪，可以使用服务提供者的 `loadViewsFrom` 方法来实现。`loadViewsFrom` 方法接收两个参数：视图模板的路径和包名称。例如，如果你的包名称是“courier”，添加如下代码到服务提供者的 `boot` 方法：

```
/**
 * Perform post-registration booting of services.
 *
 * @return void
 */
public function boot(){
    $this->loadViewsFrom(__DIR__.'/path/to/views', 'courier');
}
```

包视图通过使用类似的 `package::view` 语法来引用。所以，你可以通过如下方式加载` courier `包上的 `admin` 视图：

```
Route::get('admin', function () {
    return view('courier::admin');
});
```

#### 4.1.1 覆盖包视图
当你使用 `loadViewsFrom` 方法的时候，Laravel 实际上为视图注册了两个存放位置：一个是 `resources/views/vendor` 目录，另一个是你指定的目录。所以，以 `courier` 为例：当请求一个包视图时，Laravel 首先检查开发者是否在 `resources/views/vendor/courier` 提供了自定义版本的视图，如果该视图不存在，Laravel 才会搜索你调用 `loadViewsFrom `方法时指定的目录。这种机制使得终端用户可以轻松地自定义/覆盖包视图。

#### 4.1.2 发布视图
如果你想要视图能够发布到应用的 `resources/views/vendor` 目录，可以使用服务提供者的` publishes `方法。该方法接收包视图路径及其相应的发布路径数组作为参数：

```
/**
 * Perform post-registration booting of services.
 *
 * @return void
 */
public function boot(){
    $this->loadViewsFrom(__DIR__.'/path/to/views', 'courier');

    $this->publishes([
        __DIR__.'/path/to/views' => base_path('resources/views/vendor/courier'),
    ]);
}
```

现在，当包用户执行 Laravel 的 Artisan 命令 `vendor:publish `时，你的视图包将会被拷贝到指定路径。

### 4.2 翻译
如果你的包包含翻译文件，你可以使用 `loadTranslationsFrom` 方法告诉 Laravel 如何加载它们，例如，如果你的包命名为“courier”，你应该添加如下代码到服务提供者的 `boot` 方法：

```
/**
 * Perform post-registration booting of services.
 *
 * @return void
 */
public function boot(){
    $this->loadTranslationsFrom(__DIR__.'/path/to/translations', 'courier');
}
```

包翻译使用形如 `package::file.line` 的语法进行引用。所以，你可以使用如下方式从` messages` 文件中加载 `courier` 包的 `welcome` 行：

```
echo trans('courier::messages.welcome');
```

### 4.3 配置
通常，你想要发布包配置文件到应用根目录下的 `config` 目录，这将允许包用户轻松覆盖默认配置选项，要发布一个配置文件，只需在服务提供者的 `boot` 方法中使用 `publishes `方法即可：

```
/**
 * Perform post-registration booting of services.
 *
 * @return void
 */
public function boot(){
    $this->publishes([
        __DIR__.'/path/to/config/courier.php' => config_path('courier.php'),
    ])
}
```

现在，当包用户执行 Laravel 的 Artisan 命令 `vendor:publish` 时，你的文件将会被拷贝到指定位置，当然，配置被发布后，可以通过和其它配置选项一样的方式进行访问：

```
$value = config('courier.option');
```

#### 4.3.1 默认包配置
你还可以选择将自己的包配置文件合并到应用的拷贝版本，这允许用户只引入他们在应用配置文件中实际想要覆盖的配置选项。要合并两个配置，在服务提供者的 `register` 方法中使用 `mergeConfigFrom` 方法即可：

```
/**
 * Register bindings in the container.
 *
 * @return void
 */
public function register(){
    $this->mergeConfigFrom(
        __DIR__.'/path/to/config/courier.php', 'courier'
    );
}
```

## 5、公共前端资源
你的包可能包含 JavaScript、CSS 和图片，要发布这些前端资源到应用根目录下的 `public` 目录，使用服务提供者的 `publishes` 方法。在本例中，我们添加一个前端资源组标签 `public`，用于发布相关的前端资源组：

```
/**
 * Perform post-registration booting of services.
 *
 * @return void
 */
public function boot(){
    $this->publishes([
        __DIR__.'/path/to/assets' => public_path('vendor/courier'),
    ], 'public');
}
```

现在，当包用户执行 `vendor:publish` 命令时，前端资源将会被拷贝到指定位置，由于你需要在每次包更新时重写前端资源，可以使用`--force` 标识：

```
php artisan vendor:publish --tag=public --force
```

如果你想要确保前端资源已经更新到最新版本，可添加该命令到` composer.json `文件的` post-update-cmd `列表。

## 6、发布文件组
你可能想要发分开布包前端资源组和资源，例如，你可能想要用户发布包配置的同时不发布包前端资源，可以通过在调用时给它们打上“标签”来实现分离。下面我们在包服务提供者的 `boot `方法中定义两个公共组：

```
/**
 * Perform post-registration booting of services.
 *
 * @return void
 */
public function boot(){
    $this->publishes([
        __DIR__.'/../config/package.php' => config_path('package.php')
    ], 'config');

    $this->publishes([
        __DIR__.'/../database/migrations/' => database_path('migrations')
    ], 'migrations');
}
```

现在用户可以在使用 Artisan 命令 `vendor:publish `时通过引用标签名来分开发布这两个组：

```
php artisan vendor:publish --provider="Vendor\Providers\PackageServiceProvider" --tag="config"
```