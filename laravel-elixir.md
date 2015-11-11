# Laravel Elixir

# 1、简介
`Laravel Elixir` 提供了一套干净、平滑的 `API `用于为 `Laravel `应用定义基本的Gulp任务。`Elixir 支持一些通用的 `CSS `和 `JavaScript `预处理器，甚至是测试工具。使用方法链， `Elixir `允许你平滑的定义资源管道。例如：

```
elixir(function(mix) {
    mix.sass('app.scss')
       .coffee('app.coffee');
});
```

如果你曾经对如何开始 `Gulp `和前端资源编译感到困惑，那么你会爱上 `Laravel Elixir`。然而，并不是强制要求在开发期间使用它。你可以自由选择使用任何前端资源管道工具，或者压根不使用。
# 2、安装 & 设置
## 2.1 安装 Node
在开始 `Elixir `之前，必须首先确保 `Node.js `在机器上已经安装：

```
node -v
```

默认情况下， `Laravel Homestead` 包含你需要的一切；然而，如果你不使用 `Vagrant`，你也可以通过访问 `Node `的下载页面轻松的安装 Node。
## 2.2 Gulp
接下来，需要安装 `Gulp `作为全局 `NPM `包：

```
npm install --global gulp
```

## 2.3 Laravel Elixir
最后，在新安装的 `Laravel `根目录下，你会发现有一个 `package.json `文件。该文件和 `composer.json `一样，只不过是用来定义 `Node `依赖而非 `PHP`，你可以通过运行如下命令来安装需要的依赖：

```
npm install
```

如果你正在 `Windows `系统上开发，需要在运行 `npm install `命令时带上`--no-bin-links`：

```
npm install --no-bin-links
```

# 3、运行 Elixir
`Elixir` 基于 `Gulp`，所以要运行 `Elixir `命令你只需要在终端中运行 `gulp `命令即可。添加`--production` 标识到命令将会最小化 `CSS `和 `JavaScript `文件：

```
// Run all tasks...
gulp

// Run all tasks and minify all CSS and JavaScript...
gulp --production
```

## 3.1 监控前端资源改变
由于每次修改前端资源后都要运行 `gulp `很不方便，可以使用 `gulp watch `命令。该命令将会一直在终端运行并监控前端文件的改动。当改变发生时，新文件将会自动被编译：

```
gulp watch
```

# 4、处理 CSS
项目根目录下的 `gulpfile.js `文件包含了所有的 `Elixir `任务。`Elixir `任务可以使用方法链的方式链接起来用于定义前端资源如何被编译。
## 4.1 Less
要将 `Less` 编译成 `CSS`，可以使用 `less `方法。`less `方法假定你的 `Less `文件都放在 `resources/assets/less`。默认情况下，本例中该任务会将编译后的 `CSS `放到 `public/css/app.css`：

```
elixir(function(mix) {
    mix.less('app.less');
});
```

你还可以将多个 `Less `文件编译成单个 `CSS `文件。同样，该文件会被放到 `public/css/app.css`：

```
elixir(function(mix) {
    mix.less([
        'app.less',
        'controllers.less'
    ]);
});
```

如果你想要自定义编译后文件的输出位置，可以传递第二个参数到 `less `方法：

```
elixir(function(mix) {
    mix.less('app.less', 'public/stylesheets');
});

// Specifying a specific output filename...
elixir(function(mix) {
    mix.less('app.less', 'public/stylesheets/style.css');
});
```

## 4.2 Sass
`sass` 方法允许你将 `Sass `编译成 `CSS`。假定你的 `Sass `文件存放在 `resources/assets/sass`，你可以像这样使用该方法：

```
elixir(function(mix) {
    mix.sass('app.scss');
});
```

同样，和 `less `方法一样，你可以将多个脚本编译成单个 `CSS `文件，甚至自定义结果 `CSS `的输出路径：

```
elixir(function(mix) {
    mix.sass([
        'app.scss',
        'controllers.scss'
    ], 'public/assets/css');
});
```

## 4.3 原生 CSS
如果你只想要将多个原生 `CSS `样式文件合并到一个文件，可以使用 `styles `方法。传递给该方法的路径相对于 `resources/assets/css `目录，结果 `CSS `被存放在 `public/css/all.css`：

```
elixir(function(mix) {
    mix.styles([
        'normalize.css',
        'main.css'
    ]);
});
```

当然，你还可以通过传递第二个参数到 `styles `方法来输出结果文件到一个自定义路径：

```
elixir(function(mix) {
    mix.styles([
        'normalize.css',
        'main.css'
    ], 'public/assets/css');});
```

## 4.4 源地图
默认源地图被启用，所以，对于每一个你编译过的文件都可以在同一目录下找到一个对应的*.css.map 文件。这种匹配允许你在浏览器中调试时将编译过的样式选择器回溯到原来的 `Sass `或 `Less`。
如果你不想为 `CSS `生成源地图，可以使用一个简单配置选项关闭它们：

```
elixir.config.sourcemaps = false;

elixir(function(mix) {
    mix.sass('app.scss');
});
```

# 5、处理 JavaScript
`Elixir` 还提供了多个函数帮助你处理 `JavaScript `文件，例如编译 `ECMAScript 6`，`CoffeeScript`，`Browserify`，最小化以及简单连接原生 `JavaScript `文件。
## 5.1 CoffeeScript
`coffee` 方法用于将 `CoffeeScript` 编译成原生 `JavaScript`。该方法接收关联到 `resources/assets/coffee `目录的 `CoffeeScript `文件的一个字符串或数组并在 `public/js `目录下生成单个 `app.js `文件：

```
elixir(function(mix) {
    mix.coffee(['app.coffee', 'controllers.coffee']);
});
```

## 5.2 Browserify
`Elixir` 还提供了 `browserify `方法，从而让你可以在浏览器中引入模块并使用 `EcmaScript 6`。
该任务假定你的脚本都存放在 `resources/assets/js `而且将结果文件存放到 `public/js/bundle.js`：

```
elixir(function(mix) {
    mix.browserify('main.js');
});
```

除了处理 `Partialify `和 `Babelify`，还可以安装并添加更多：

```
npm install vueify --save-dev
```

```
elixir.config.js.browserify.transformers.push({
    name: 'vueify',
    options: {}
});

elixir(function(mix) {
    mix.browserify('main.js');
});
```

## 5.3 Babel
`babel` 方法可用于将`EcmaScript 6` 和 `7`编译成原生 `JavaScript`。该方法接收相对于 `resources/assets/js `目录的文件数组，并在 `public/js `目录下生成单个 `all.js`：

```
elixir(function(mix) {
    mix.babel([
        'order.js',
        'product.js'
    ]);});
```

要选择不同的输出路径，只需将目标路径作为第二个参数传递给该方法。处了 `Babel `编译之外，`babel` `和 mix.scripts()`的使用方法和功能差不多。
## 5.4 脚本
如果你有多个 `JavaScript `文件想要编译成单个文件，可以使用 scripts 方法。
scripts 方法假定所有路径相对于 `resources/assets/js `目录，而且所有结果 `JavaScript `默认存放在 `public/js/all.js`：

```
elixir(function(mix) {
    mix.scripts([
        'jquery.js',
        'app.js'
    ]);
});
```

如果你需要将多个脚本集合合并到不同的文件，需要多次调用 `scripts `方法。该方法的第二个参数决定每个合并的结果文件名：

```
elixir(function(mix) {
    mix.scripts(['app.js', 'controllers.js'], 'public/js/app.js')
       .scripts(['forum.js', 'threads.js'], 'public/js/forum.js');
});
```

如果你需要将多个脚本合并到给定目录，可以使用 `scriptsIn `方法。结果 `JavaScript `会被存放到 `public/js/all.js`：

```
elixir(function(mix) {
    mix.scriptsIn('public/js/some/directory');
});
```

# 6、版本号/缓存刷新
很多开发者会给编译的前端资源添加时间戳或者唯一令牌后缀以强制浏览器加载最新版本而不是代码的缓存副本。`Elixir `可以使用 `version `方法为你处理这种情况。
`version `方法接收相对于 `public `目录的文件名，附加唯一 `hash `到文件名，从而实现缓存刷新。例如，生成的文件名看上去是这样——all-16d570a7.css：

```
elixir(function(mix) {
    mix.version('css/all.css');
});
```

生成版本文件后，可以在视图中使用 `Elixir `全局的 `PHP `帮助函数 `elixir `方法来加载相应的带 `hash `值的前端资源，`elixir `函数会自动判断 `hash `文件名：

```
<link rel="stylesheet" href="{{ elixir('css/all.css') }}">
```

## 6.1 给多个文件加上版本号
你可以传递一个数组到 `version `方法来为多个文件添加版本号：

```
elixir(function(mix) {
    mix.version(['css/all.css', 'js/app.js']);
});
```

一旦文件被加上版本号，就可以使用帮助函数 `elixir `来生成指向该 `hash `文件的链接。记住，你只需要传递没有 `hash `值的文件名到 `elixir `方法。该帮助函数使用未加 `hash `值的文件名来判断文件当前的 `hash `版本：

```
<link rel="stylesheet" href="{{ elixir('css/all.css') }}">

<script src="{{ elixir('js/app.js') }}"></script>
```

# 7、调用存在的 Gulp 任务
如果你需要从 `Elixir `调用已存在的 `Gulp `任务，可以使用 `task `方法。例如，假定你有一个调用时只是简单说几句话的 `Gulp `任务：

```
gulp.task('speak', function() {
    var message = 'Tea...Earl Grey...Hot';
    gulp.src('').pipe(shell('say ' + message));
});
```

如果你想要从 `Elixir `中调用该任务，使用 `mix.task `方法并传递任务名作为该方法的唯一参数：

```
elixir(function(mix) {
    mix.task('speak');
});
```

## 7.1 自定义监控者
如果你需要注册一个监控器在每一次文件修改时都运行自定义任务，传递一个正则表达式作为 `task `方法的第二个参数：

```
elixir(function(mix) {
    mix.task('speak', 'app/**/*.php');});
```

# 8、编写 Elixir 扩展
如果你需要比 `Elixir `的 `task `方法所提供的更加灵活的功能，可以创建自定义的 `Elixir `扩展。`Elixir `扩展允许你传递参数到自定义任务，例如，你可以像这样编写一个扩展：

```
// File: elixir-extensions.js
var gulp = require('gulp');
var shell = require('gulp-shell');
var Elixir = require('laravel-elixir');

var Task = Elixir.Task;

Elixir.extend('speak', function(message) {
    new Task('speak', function() {
        return gulp.src('').pipe(shell('say ' + message));
    });

});

// mix.speak('Hello World');
```

就是这样简单！注意你的特定 `Gulp `逻辑应该放到闭包函数里作为第二个参数传递给 `Task `构造器。你可以将其放在 `Gulpfile `顶端，或者将其解析到自定义的任务文件。例如，如果你将扩展放在 `elixir-extensions.js`，可以在主 `Gulpfile `中像这样引入该文件：

```
// File: Gulpfile.js
var elixir = require('laravel-elixir');

require('./elixir-extensions')

elixir(function(mix) {
    mix.speak('Tea, Earl Grey, Hot');
});
```

## 8.1 自定义监控器
如果你想要自定义任务在运行 `gulp watch` 的时候被触发，可以注册一个监控器：

```
new Task('speak', function() {
    return gulp.src('').pipe(shell('say ' + message));
}).watch('./app/**');
```