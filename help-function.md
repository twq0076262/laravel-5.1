# 帮助函数

## 1、简介
`Laravel` 自带了一系列 `PHP `帮助函数，很多被框架自身使用，然而，如果你觉得方便的话也可以在应用中随心所欲的使用它们。

## 2、 数组函数

### array_add()
`array_add` 函数添加给定键值对到数组，如果给定键不存在的话：

```
$array = array_add(['name' => 'Desk'], 'price', 100);
// ['name' => 'Desk', 'price' => 100]
```

### array_divide()
`array_divide` 函数返回两个数组，一个包含原数组的所有键，另外一个包含原数组的所有值：

```
list($keys, $values) = array_divide(['name' => 'Desk']);
// $keys: ['name']
// $values: ['Desk']
```

### array_dot()
`array_dot` 函数使用”.“号将将多维数组转化为一维数组：

```
$array = array_dot(['foo' => ['bar' => 'baz']]);
// ['foo.bar' => 'baz'];
```

### array_except()
`array_except` 方法从数组中移除给定键值对：

```
$array = ['name' => 'Desk', 'price' => 100];

$array = array_except($array, ['price']);
// ['name' => 'Desk']
```

### array_first()
`array_first` 方法返回通过测试数组的第一个元素：

```
$array = [100, 200, 300];

$value = array_first($array, function ($key, $value) {
    return $value >= 150;});
// 200
```

默认值可以作为第三个参数传递给该方法，如果没有值通过测试的话返回默认值：

```
$value = array_first($array, $callback, $default);
```

### array_flatten()
`array_flatten` 方法将多维数组转化为一维数组：

```
$array = ['name' => 'Joe', 'languages' => ['PHP', 'Ruby']];

$array = array_flatten($array);
// ['Joe', 'PHP', 'Ruby'];
```

### array_forget()
`array_forget` 方法使用”.“号从嵌套数组中移除给定键值对：

```
$array = ['products' => ['desk' => ['price' => 100]]];

array_forget($array, 'products.desk');
// ['products' => []]
```

### array_get()
`array_get` 方法使用”.“号从嵌套数组中获取值：

```
$array = ['products' => ['desk' => ['price' => 100]]];

$value = array_get($array, 'products.desk');
// ['price' => 100]
```

`array_get` 函数还接收一个默认值，如果指定键不存在的话则返回该默认值：

```
$value = array_get($array, 'names.john', 'default');
```

### array_only()
`array_only` 方法只从给定数组中返回指定键值对：

```
$array = ['name' => 'Desk', 'price' => 100, 'orders' => 10];

$array = array_only($array, ['name', 'price']);
// ['name' => 'Desk', 'price' => 100]
```

### array_pluck()
`array_pluck` 方法从数组中返回给定键对应的键值对列表：

```
$array = [
    ['developer' => ['name' => 'Taylor']],
    ['developer' => ['name' => 'Abigail']]];

$array = array_pluck($array, 'developer.name');
// ['Taylor', 'Abigail'];
```

### array_pull()
`array_pull` 方法从数组中返回并移除键值对：

```
$array = ['name' => 'Desk', 'price' => 100];

$name = array_pull($array, 'name');
// $name: Desk

// $array: ['price' => 100]
```

### array_set()
`array_set` 方法在嵌套数组中使用”.“号设置值：

```
$array = ['products' => ['desk' => ['price' => 100]]];

array_set($array, 'products.desk.price', 200);
// ['products' => ['desk' => ['price' => 200]]]
```

### array_sort()
`array_sort` 方法通过给定闭包的结果对数组进行排序：

```
$array = [
    ['name' => 'Desk'],
    ['name' => 'Chair'],
];

$array = array_values(array_sort($array, function ($value) {
    return $value['name'];
}));

/*
    [
        ['name' => 'Chair'],
        ['name' => 'Desk'],
    ]
*/
```

### array_sort_recursive()
`array_sort_recursive`函数使用 `sort `函数对数组进行递归排序：

```
$array = [
    [
        'Roman',
        'Taylor',
        'Li',
    ],
    [
        'PHP',
        'Ruby',
        'JavaScript',
    ],
];

$array = array_sort_recursive($array);

/*
    [
        [
            'Li',
            'Roman',
            'Taylor',
        ],
        [
            'JavaScript',
            'PHP',
            'Ruby',
        ]
    ];
*/
```

### array_where()
`array_where` 函数使用给定闭包对数组进行排序：

```
$array = [100, '200', 300, '400', 500];

$array = array_where($array, function ($key, $value) {
    return is_string($value);
});
// [1 => 200, 3 => 400]
```

### head()
`head` 函数只是简单返回给定数组的第一个元素：

```
$array = [100, 200, 300];

$first = head($array);
// 100
```

### last()
`last` 函数返回给定数组的最后一个元素：

```
$array = [100, 200, 300];

$last = last($array);
// 300
```

## 3、路径函数
### app_path()
`app_path` 函数返回 `app` 目录的绝对路径：

```
$path = app_path();
```

你还可以使用 `app_path `函数为相对于 `app `目录的给定文件生成绝对路径：

```
$path = app_path('Http/Controllers/Controller.php');
```

### base_path()
`base_path` 函数返回项目根目录的绝对路径：

```
$path = base_path();
```

你还可以使用 `base_path `函数为相对于应用目录的给定文件生成绝对路径：

```
$path = base_path('vendor/bin');
```

### config_path()
`config_path` 函数返回应用配置目录的绝对路径：

```
$path = config_path();
```

### database_path()
`database_path` 函数返回应用数据库目录的绝对路径：

```
$path = database_path();
```

### public_path()
`public_path`函数返回 `public `目录的绝对路径：

```
$path = public_path();
```

### storage_path()
`storage_path` 函数返回 `storage` 目录的绝对路径：

```
$path = storage_path();
```

还可以使用 `storage_path` 函数生成相对于 `storage` 目录的给定文件的绝对路径：

```
$path = storage_path('app/file.txt');
```

## 4、字符串函数
### camel_case()
`camel_case` 函数将给定字符串转化为按驼峰式命名规则的字符串：

```
$camel = camel_case('foo_bar');
// fooBar
```

### class_basename()
`class_basename` 返回给定类移除命名空间后的类名：

```
$class = class_basename('Foo\Bar\Baz');
// Baz
```

### e()
`e` 函数在给定字符串上运行 `htmlentities`：

```
echo e('<html>foo</html>');
// &lt;html&gt;foo&lt;/html&gt;
```

### ends_with()
`ends_with` 函数判断给定字符串是否以给定值结尾：

```
$value = ends_with('This is my name', 'name');
// true
```

### snake_case()
`snake_case` 函数将给定字符串转化为下划线分隔的字符串：

```
$snake = snake_case('fooBar');
// foo_bar
```

### str_limit()
`str_limit` 函数限制输出字符串的数目，该方法接收一个字符串作为第一个参数以及该字符串最大输出字符数作为第二个参数：

```
$value = str_limit('The PHP framework for web artisans.', 7);
// The PHP...
```

### starts_with()
`starts_with` 函数判断给定字符串是否以给定值开头：

```
$value = starts_with('This is my name', 'This');
// true
```

### str_contains()
`str_contains` 函数判断给定字符串是否包含给定值：

```
$value = str_contains('This is my name', 'my');
// true
```

### str_finish()
`str_finish` 函数添加字符到字符串结尾：

```
$string = str_finish('this/string', '/');
// this/string/
```

### str_is()
`str_is` 函数判断给定字符串是否与给定模式匹配，星号可用于表示通配符：

```
$value = str_is('foo*', 'foobar');
// true
$value = str_is('baz*', 'foobar');
// false
```

### str_plural()
`str_plural` 函数将字符串转化为复数形式，该函数当前只支持英文：

```
$plural = str_plural('car');
// cars
$plural = str_plural('child');
// children
```

### str_random()
`str_random` 函数通过指定长度生成随机字符串：

```
$string = str_random(40);
```

### str_singular()
`str_singular` 函数将字符串转化为单数形式，该函数目前只支持英文：

```
$singular = str_singular('cars');
// car
```

### str_slug()
`str_slug` 函数将给定字符串生成URL友好的格式：

```
$title = str_slug("Laravel 5 Framework", "-");
// laravel-5-framework
```

### studly_case()
`studly_case` 函数将给定字符串转化为单词开头字母大写的格式：

```
$value = studly_case('foo_bar');
// FooBar
```

### trans()
`trans` 函数使用本地文件翻译给定语言行：

```
echo trans('validation.required'):
```

### trans_choice()
`trans_choice` 函数翻译带拐点的给定语言行：

```
$value = trans_choice('foo.bar', $count);
```

## 5、URL 函数

### action()
`action` 函数为给定控制器动作生成 `URL`，你不需要传递完整的命名空间到该控制器，传递相对于命名空间 `App\Http\Controllers` 的类名即可：

```
$url = action('HomeController@getIndex');
```

如果该方法接收路由参数，你可以将其作为第二个参数传递进来：

```
$url = action('UserController@profile', ['id' => 1]);
```

### asset()
使用当前请求的 `scheme（HTTP 或 HTTPS）`为前端资源生成一个 URL：

```
$url = asset('img/photo.jpg');
```

### secure_asset()
使用 `HTTPS `为前端资源生成一个 URL：

```
echo secure_asset('foo/bar.zip', $title, $attributes = []);
```

### route()
`route` 函数为给定命名路由生成一个 `URL`：

```
$url = route('routeName');
```

如果该路由接收参数，你可以将其作为第二个参数传递进来：

```
$url = route('routeName', ['id' => 1]);
```

### url()
`url` 函数为给定路径生成绝对路径：

```
echo url('user/profile');

echo url('user/profile', [1]);
```

## 6、其它函数
### auth()
`auth` 函数返回一个认证器实例，为方便起见你可以用其取代 `Auth` 门面：

```
$user = auth()->user();
```

### back()
`back` 函数生成重定向响应到用户前一个位置：

```
return back();
```

### bcrypt()
`bcrypt` 函数使用 `Bcrypt `对给定值进行哈希，你可以用其替代 `Hash `门面：

```
$password = bcrypt('my-secret-password');
```

### config()
`config` 函数获取配置变量的值，配置值可以通过使用”.”号访问，包含文件名以及你想要访问的选项。如果配置选项不存在的话默认值将会被指定并返回：

```
$value = config('app.timezone');$value = config('app.timezone', $default);
```

帮助函数 `config `还可以用于在运行时通过传递键值对数组设置配置变量值：

```
config(['app.debug' => true]);
```

### csrf_field()
`csrf_field` 函数生成一个包含 `CSRF` 令牌值的 `HTML` 隐藏域，例如，使用`Blade` 语法：

```
{!! csrf_field() !!}
```

### csrf_token()
`csrf_token` 函数获取当前 `CSRF` 令牌的值：

```
$token = csrf_token();
```

## dd()
`dd` 函数输出给定变量值并终止脚本执行：

```
dd($value);
```

### elixir()
`elixir` 函数获取带版本号的Elixir文件路径：

```
elixir($file);
```

### env()
`env` 函数获取环境变量值或返回默认值：

```
$env = env('APP_ENV');
// Return a default value if the variable doesn't exist...
$env = env('APP_ENV', 'production');
```

### event()
`event` 函数分发给定事件到对应监听器：

```
event(new UserRegistered($user));
```

### factory()
`factory` 函数为给定类、名称和数量创建模型工厂构建器，可用于测试或数据填充：

```
$user = factory('App\User')->make();
```

### method_field()
`method_field` 函数生成包含 `HTTP` 请求方法的 `HTML` 隐藏域，例如：

```
<form method="POST">
    {!! method_field('delete') !!}</form>
```

### old()
`old` 函数获取一次性存放在 `session` 中的值：

```
$value = old('value');
```

### redirect()
`redirect` 函数返回重定向器实例进行重定向：

```
return redirect('/home');
```

### response()
`response` 函数创建一个响应实例或者获取响应工厂实例：

```
return response('Hello World', 200, $headers);return response()->json(['foo' => 'bar'], 200, $headers)
```

### value()
`value` 函数返回给定的值，然而，如果你传递一个闭包到该函数，该闭包将会被执行并返回执行结果：

```
$value = value(function() { return 'bar'; });
```

### view()
`view` 函数获取一个视图实例：

```
return view('auth.login');
```

### with()
`with` 函数返回给定的值，该函数在方法链中特别有用，别的地方就没什么用了：

```
$value = with(new Foo)->work();
```