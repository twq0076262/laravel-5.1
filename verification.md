#  验证

## 1、简介
Laravel 提供了多种方法来验证应用输入数据。默认情况下，Laravel 的控制器基类使用 ValidatesRequests trait，该 trait 提供了便利的方法通过各种功能强大的验证规则来验证输入的 HTTP请求。

## 2、快速入门
要学习 Laravel 强大的验证特性，让我们先看一个完整的验证表单并返回错误信息给用户的例子。

### 2.1 定义路由
首先，我们假定在 app/Http/routes.php 文件中包含如下路由：

```
// 显示创建博客文章表单...
Route::get('post/create', 'PostController@create');
// 存储新的博客文章...
Route::post('post', 'PostController@store');
```

当然，GET 路由为用户显示了一个创建新的博客文章的表单，POST 路由将新的博客文章存储到数据库。

### 2.2 创建控制器
接下来，让我们看一个处理这些路由的简单控制器示例。我们先将 store 方法留空：

```
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use App\Http\Controllers\Controller;

class PostController extends Controller{
    /**
     * 显示创建新的博客文章的表单
     *
     * @return Response
     */
    public function create()
    {
        return view('post.create');
    }

    /**
     * 存储新的博客文章
     *
     * @param  Request  $request
     * @return Response
     */
    public function store(Request $request)
    {
        // 验证并存储博客文章...
    }
}
```

### 2.3 编写验证逻辑
现在我们准备用验证新博客文章输入的逻辑填充 store 方法。如果你检查应用的控制器基类（App\Http\Controllers\Controller），你会发现该类使用了 ValidatesRequests trait，这个 trait 在所有控制器中提供了一个便利的 validate 方法。

validate 方法接收一个 HTTP 请求输入数据和验证规则，如果验证规则通过，代码将会继续往下执行；然而，如果验证失败，将会抛出一个异常，相应的错误响应也会自动发送给用户。在一个传统的 HTTP 请求案例中，将会生成一个重定向响应，如果是 AJAX 请求则会返回一个 JSON 响应。
要更好的理解 validate 方法，让我们回到 store 方法：

```
/**
 * 存储博客文章
 *
 * @param  Request  $request
 * @return Response
 */
public function store(Request $request){
    $this->validate($request, [
        'title' => 'required|unique:posts|max:255',
        'body' => 'required',
    ]);

    // 验证通过，存储到数据库...
}
```

正如你所看到的，我们只是传递输入的 HTTP 请求和期望的验证规则到 validate 方法，在强调一次，如果验证失败，相应的响应会自动生成。如果验证通过，控制器将会继续正常执行。

#### 2.3.1 嵌套属性注意事项
如果 HTTP 请求中包含“嵌套”参数，可以使用“.”在验证规则中指定它们：

```
$this->validate($request, [
    'title' => 'required|unique:posts|max:255',
    'author.name' => 'required',
    'author.description' => 'required',
]);
```

### 2.4 显示验证错误信息
那么，如果请求输入参数没有通过给定验证规则怎么办？正如前面所提到的，Laravel 将会自动将用户重定向回上一个位置。此外，所有验证错误信息会自动一次性存放到 session。

注意我们并没有在 GET 路由中明确绑定错误信息到视图。这是因为 Laravel 总是从 session 数据中检查错误信息，而且如果有的话会自动将其绑定到视图。所以，值得注意的是每次请求的所有视图中总是存在一个`$errors` 变量，从而允许你在视图中方便而又安全地使用。`$errors` 变量是的一个 Illuminate\Support\MessageBag实例。想要了解更多关于该对象的信息，查看其文档。

所以，在我们的例子中，验证失败的话用户将会被重定向到控制器的 create 方法，从而允许我们在视图中显示错误信息：

```
<!-- /resources/views/post/create.blade.php -->

<h1>Create Post</h1>

@if (count($errors) > 0)
    <div class="alert alert-danger">
        <ul>
            @foreach ($errors->all() as $error)
                <li>{{ $error }}</li>
            @endforeach
        </ul>
    </div>
@endif

<!-- Create Post Form -->
```

### 2.5 AJAX 请求&验证
在这个例子中，我们使用传统的表单来发送数据到应用。然而，很多应用使用 AJAX 请求。在 AJAX 请求中使用 validate 方法时，Laravel 不会生成重定向响应。取而代之的，Laravel 生成一个包含验证错误信息的 JSON 响应。该 JSON 响应会带上一个 HTTP 状态码 422。

## 3、其它验证方法

### 3.1 手动创建验证器
如果你不想使用 ValidatesRequests trait 的 validate 方法，可以使用Validator门面手动创建一个验证器实例，该门面上的 make 方法用于生成一个新的验证器实例：

```
<?php

namespace App\Http\Controllers;

use Validator;
use Illuminate\Http\Request;
use App\Http\Controllers\Controller;

class PostController extends Controller{
    /**
     * 存储新的博客文章
     *
     * @param  Request  $request
     * @return Response
     */
    public function store(Request $request)
    {
        $validator = Validator::make($request->all(), [
            'title' => 'required|unique:posts|max:255',
            'body' => 'required',
        ]);

        if ($validator->fails()) {
            return redirect('post/create')
                        ->withErrors($validator)
                        ->withInput();
        }

        // 存储博客文章...
    }
}
```

传递给 make 方法的第一个参数是需要验证的数据，第二个参数是要应用到数据上的验证规则。

检查请求是够通过验证后，可以使用 withErrors 方法将错误数据一次性存放到 session，使用该方法时，`$errors` 变量重定向后自动在视图间共享，从而允许你轻松将其显示给用户，withErrors 方法接收一个验证器、或者一个 MessageBag，又或者一个 PHP 数组。

#### 3.1.1 命名错误包
如果你在单个页面上有多个表单，可能需要命名 MessageBag，从而允许你为指定表单获取错误信息。只需要传递名称作为第二个参数给 withErrors 即可：

```
return redirect('register')
            ->withErrors($validator, 'login');
```

然后你就可以从`$errors` 变量中访问命名的 MessageBag 实例：

```
{{ $errors->login->first('email') }}
```

#### 3.1.2 验证钩子之后
验证器允许你在验证完成后添加回调，这种机制允许你轻松执行更多验证，甚至添加更多错误信息到消息集合。使用验证器实例上的 after 方法即可：

```
$validator = Validator::make(...);

$validator->after(function($validator) {
    if ($this->somethingElseIsInvalid()) {
        $validator->errors()->add('field', 'Something is wrong with this field!');
    }
});

if ($validator->fails()) {
    //
}
```

### 3.2 表单请求验证
对于更复杂的验证场景，你可能想要创建一个“表单请求”。表单请求是包含验证逻辑的自定义请求类，要创建表单验证类，可以使用 Artisan 命令 make:request：

```
php artisan make:request StoreBlogPostRequest
```

生成的类位于 app/Http/Requests 目录下，接下来我们添加少许验证规则到 rules 方法：

```
/**
 * 获取应用到请求的验证规则
 *
 * @return array
 */
public function rules(){
    return [
        'title' => 'required|unique:posts|max:255',
        'body' => 'required',
    ];
}
```

那么，验证规则如何生效呢？你所要做的就是在控制器方法中类型提示该请求。表单输入请求会在控制器方法被调用之前被验证，这就是说你不需要将控制器和验证逻辑杂糅在一起：

```
/**
 * 存储输入的博客文章
 *
 * @param  StoreBlogPostRequest  $request
 * @return Response
 */
public function store(StoreBlogPostRequest $request){
    // The incoming request is valid...
}
```

如果验证失败，重定向响应会被生成并将用户退回上一个位置，错误信息也会被一次性存储到 session 以便在视图中显示。如果是 AJAX 请求，带 422 状态码的 HTTP 响应将会返回给用户，该响应数据中还包含了 JSON 格式的验证错误信息。

#### 3.2.1 认证表单请求
表单请求类还包含了一个 authorize 方法，你可以检查认证用户是否有资格更新指定资源。例如，如果用户尝试更新一个博客评论，那么他是否是评论的所有者呢？举个例子：

```
/**
 * 判断请求用户是否经过认证
 *
 * @return bool
 */
public function authorize(){
    $commentId = $this->route('comment');
    return Comment::where('id', $commentId)
                  ->where('user_id', Auth::id())->exists();
}
```


注意上面这个例子中对 route 方法的调用。该方法赋予用户访问被调用路由 URI 参数的权限，比如下面这个例子中的{comment}参数：

```
Route::post('comment/{comment}');
```

如果 authorize 方法返回 false，一个包含 403 状态码的 HTTP 响应会自动返回而且控制器方法将不会被执行。
如果你计划在应用的其他部分包含认证逻辑，只需在 authorize 方法中简单返回 true 即可：

```
/**
 * 判断请求用户是否经过认证
 *
 * @return bool
 */
public function authorize(){
    return true;
}
```

#### 3.2.2 自定义一次性错误格式
如果你想要自定义验证失败时一次性存储到 session 中验证错误信息的格式，重写请求基类（App\Http\Requests\Request）中的 formatErrors 方法即可。不要忘记在文件顶部导入 Illuminate\Contracts\Validation\Validator 类：

```
/**
 * {@inheritdoc}
 */
protected function formatErrors(Validator $validator){
    return $validator->errors()->all();
}
```

## 4、处理错误信息
调用 Validator 实例上的 errors 方法之后，将会获取一个 Illuminate\Support\MessageBag 实例，该实例中包含了多种处理错误信息的便利方法。

**获取某字段的第一条错误信息**

要获取指定字段的第一条错误信息，可以使用 first 方法：

```
$messages = $validator->errors();
echo $messages->first('email');
```

**获取指定字段的所有错误信息**

如果你想要简单获取指定字段的所有错误信息数组，使用 get 方法：

```
foreach ($messages->get('email') as $message) {
    //
}
```

**获取所有字段的所有错误信息**

要获取所有字段的所有错误信息，可以使用 all 方法：

```
foreach ($messages->all() as $message) {
    //
}
```

**判断消息中是否存在某字段的错误信息**

```
if ($messages->has('email')) {
    //
}
```

**获取指定格式的错误信息**

```
echo $messages->first('email', '<p>:message</p>');
```

**获取指定格式的所有错误信息**

```
foreach ($messages->all('<li>:message</li>') as $message) {
    //
}
```

### 4.1 自定义错误信息
如果需要的话，你可以使用自定义错误信息替代默认的，有多种方法来指定自定义信息。首先，你可以传递自定义信息作为第三方参数给 Validator::make 方法：

```
$messages = [
    'required' => 'The :attribute field is required.',
];

$validator = Validator::make($input, $rules, $messages);
```

在本例中，:attribute 占位符将会被验证时实际的字段名替换，你还可以在验证消息中使用其他占位符，例如：

```
$messages = [
    'same'    => 'The :attribute and :other must match.',
    'size'    => 'The :attribute must be exactly :size.',
    'between' => 'The :attribute must be between :min - :max.',
    'in'      => 'The :attribute must be one of the following types: :values',
];
```

#### 4.1.1 为给定属性指定自定义信息
有时候你可能只想为特定字段指定自定义错误信息，可以通过”.”来实现，首先指定属性名，然后是规则：

```
$messages = [
    'email.required' => 'We need to know your e-mail address!',
];
```

#### 4.1.2 在语言文件中指定自定义消息
在很多案例中，你可能想要在语言文件中指定属性特定自定义消息而不是将它们直接传递给 Validator。要实现这个，添加消息到 resources/lang/xx/validation.php 语言文件的 custom 数组：

```
'custom' => [
    'email' => [
        'required' => 'We need to know your e-mail address!',
    ],
],
```

## 5、有效验证规则
下面是有效规则及其函数列表：

- 	Accepted 
-	Active URL 
-	After (Date) 
-	Alpha 
-	Alpha Dash 
-	Alpha Numeric 
-	Array 
-	Before (Date) 
-	Between 
-	Boolean 
-	Confirmed 
-	Date 
-	Date Format 
-	Different 
-	Digits 
-	Digits Between 
-	E-Mail 
-	Exists (Database) 
-	Image (File) 
-	In 
-	Integer 
-	IP Address 
-	Max 
-	MIME Types (File) 
-	Min 
-	Not In 
-	Numeric 
-	Regular Expression 
-	Required 
-	Required If 
-	Required With 
-	Required With All 
-	Required Without 
-	Required Without All 
-	Same 
-	Size 
-	String 
-	Timezone 
-	Unique (Database) 
-	URL

### accepted
在验证中该字段的值必须是 yes、on、1 或 true，这在“同意服务协议”时很有用。

### active_url
该字段必须是一个基于 PHP 函数 checkdnsrr 的有效 URL

### after:date
该字段必须是给定日期后的一个值，日期将会通过 PHP 函数 strtotime 传递：

```
'start_date' => 'required|date|after:tomorrow'
```

你可以指定另外一个比较字段而不是使用 strtotime 验证传递的日期字符串：

```
'finish_date' => 'required|date|after:start_date'
```

### alpha
该字段必须是字母

### alpha_dash
该字段可以包含字母和数字，以及破折号和下划线

### alpha_num
该字段必须是字母或数字

### array
该字段必须是 PHP 数组

### before:date
验证字段必须是指定日期之前的一个数值，该日期将会传递给 PHP strtotime 函数。

### between:min,max
验证字段尺寸在给定的最小值和最大值之间，字符串、数值和文件都可以使用该规则

### boolean
验证字段必须可以被转化为 boolean，接收 true, false, 1,0, "1", 和 "0"等输入。

### confirmed
验证字段必须有一个匹配字段 foo_confirmation，例如，如果验证字段是 password，必须输入一个与之匹配的 password_confirmation 字段

### date
验证字段必须是一个基于 PHP strtotime 函数的有效日期

### date_format:format
验证字段必须匹配指定格式，该格式将使用 PHP 函数 date_parse_from_format 进行验证。你应该在验证字段时使用 date 或 date_format

### different:field
验证字段必须是一个和指定字段不同的值

### digits:value
验证字段必须是数字且长度为 value 指定的值

### digits_between:min,max
验证字段数值长度必须介于最小值和最大值之间

### email
验证字段必须是格式化的电子邮件地址

### exists:table.column
验证字段必须存在于指定数据表

- **基本使用：**

```
'state' => 'exists:states'
```

- **指定自定义列名：**

```
'state' => 'exists:states,abbreviation'
```

还可以添加更多查询条件到 where 查询子句：

```
'email' => 'exists:staff,email,account_id,1'
```

- **传递 NULL 作为 where 子句的值将会判断数据库值是否为 NULL：**

```
'email' => 'exists:staff,email,deleted_at,NULL'
```

- **image**验证文件必须是图片（jpeg、png、bmp、gif 或者 svg）

- **in:foo,bar…**验证字段值必须在给定的列表中

- **integer**验证字段必须是整型

- **ip**验证字段必须是 IP 地址

- **max:value**验证字段必须小于等于最大值，和字符串、数值、文件字段的 size 规则一起使用

- **mimes:foo,bar,…**验证文件的 MIMIE 类型必须是该规则列出的扩展类型中的一个
MIMIE 规则的基本使用：

```
'photo' => 'mimes:jpeg,bmp,png'
```

- **min:value**
验证字段的最小值，和字符串、数值、文件字段的 size 规则一起使用

- **not_in:foo,bar,…**
验证字段值不在给定列表中

- **numeric**
验证字段必须是数值

- **regex:pattern**
验证字段必须匹配给定正则表达式
注意：使用 regex 模式时，规则必须放在数组中，而不能使用管道分隔符，尤其是正则表达式中使用管道符号时。

- **required**
验证字段时必须的

- **required_if:anotherfield,value,…**
验证字段在另一个字段等于指定值 value 时是必须的

- **required_with:foo,bar,…**
验证字段只有在任一其它指定字段存在的话才是必须的

- **required_with_all:foo,bar,…**
验证字段只有在所有指定字段存在的情况下才是必须的

- **required_without:foo,bar,…**
验证字段只有当任一指定字段不存在的情况下才是必须的

- **required_without_all:foo,bar,…**
验证字段只有当所有指定字段不存在的情况下才是必须的

- **same:field**
给定字段和验证字段必须匹配

- **size:value**
验证字段必须有和给定值相 value 匹配的尺寸，对字符串而言，value 是相应的字符数目；对数值而言，value 是给定整型值；对文件而言，value 是相应的文件字节数

- **string**
验证字段必须是字符串

- **timezone**
验证字符必须是基于 PHP 函数 timezone_identifiers_list 的有效时区标识

- **unique:table,column,except,idColumn**
验证字段在给定数据表上必须是唯一的，如果不指定 column 选项，字段名将作为默认 column。
指定自定义列名：

```
'email' => 'unique:users,email_address'
```

- **自定义数据库连接**
有时候，你可能需要自定义验证器生成的数据库连接，正如上面所看到的，设置 unique:users 作为验证规则将会使用默认数据库连接来查询数据库。要覆盖默认连接，在数据表名后使用”.“指定连接：

```
'email' => 'unique:connection.users,email_address'
```

- **强制一个唯一规则来忽略给定 ID：**
有时候，你可能希望在唯一检查时忽略给定 ID，例如，考虑一个包含用户名、邮箱地址和位置的”更新属性“界面，当然，你将会验证邮箱地址是唯一的，然而，如果用户只改变用户名字段而并没有改变邮箱字段，你不想要因为用户已经拥有该邮箱地址而抛出验证错误，你只想要在用户提供的邮箱已经被别人使用的情况下才抛出验证错误，要告诉唯一规则忽略用户 ID，可以传递 ID 作为第三个参数：

```
'email' => 'unique:users,email_address,'.$user->id
```

- **添加额外的 where 子句：**
还可以指定更多条件给 where 子句：

```
'email' => 'unique:users,email_address,NULL,id,account_id,1'
```

- **url**
验证字段必须是基于 PHP 函数 filter_var 过滤的的有效 URL

## 6、添加条件规则
在某些场景下，你可能想要只有某个字段存在的情况下运行验证检查，要快速完成这个，添加 sometimes 规则到规则列表：

```
$v = Validator::make($data, [
    'email' => 'sometimes|required|email',
]);
```

在上例中，email 字段只有存在于`$data` 数组时才会被验证。

**复杂条件验证**
有时候你可能想要基于更复杂的条件逻辑添加验证规则。例如，你可能想要只有在另一个字段值大于 100 时才要求一个给定字段是必须的，或者，你可能需要只有当另一个字段存在时两个字段才都有给定值。添加这个验证规则并不是一件头疼的事。首先，创建一个永远不会改变的静态规则到 Validator 实例：

```
$v = Validator::make($data, [
    'email' => 'required|email',
    'games' => 'required|numeric',
]);
```

让我们假定我们的 web 应用服务于游戏收集者。如果一个游戏收集者注册了我们的应用并拥有超过 100 个游戏，我们想要他们解释为什么他们会有这么多游戏，例如，也许他们在运营一个游戏二手店，又或者他们只是喜欢收集。要添加这种条件，我们可以使用 Validator 实例上的 sometimes 方法：

```
$v->sometimes('reason', 'required|max:500', function($input) {
    return $input->games >= 100;
});
```

传递给 sometimes 方法的第一个参数是我们需要有条件验证的名称字段，第二个参数是我们想要添加的规则，如果作为第三个参数的闭包返回 true，规则被添加。该方法让构建复杂条件验证变得简单，你甚至可以一次为多个字段添加条件验证：

```
$v->sometimes(['reason', 'cost'], 'required', function($input) {
    return $input->games >= 100;
});
```

注意：传递给闭包的`$input` 参数是 Illuminate\Support\Fluent 的一个实例，可用于访问输入和文件。

# 7、自定义验证规则
Laravel 提供了多种有用的验证规则；然而，你可能还是想要指定一些自己的验证规则。注册验证规则的一种方法是使用 Validator门面的 extend 方法。让我们在服务提供者中使用这种方法来注册一个自定义的验证规则：

```
<?php

namespace App\Providers;

use Validator;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider{
    /**
     * 启动应用服务
     *
     * @return void
     */
    public function boot()
    {
        Validator::extend('foo', function($attribute, $value, $parameters) {
            return $value == 'foo';
        });
    }

    /**
     * 注册服务提供者
     *
     * @return void
     */
    public function register()
    {
        //
    }
}
```

自定义验证器闭包接收三个参数：要验证的属性名称，属性值和传递给规则的参数数组。
你还可以传递类和方法到 extend 方法而不是闭包：

```
Validator::extend('foo', 'FooValidator@validate');
```

**定义错误信息**
你还需要为自定义规则定义错误信息。你可以使用内联自定义消息数组或者在验证语言文件中添加条目来实现这一目的。消息应该被放到数组的第一维，而不是在只用于存放属性指定错误信息的 custom 数组内：

```
"foo" => "Your input was invalid!",
"accepted" => "The :attribute must be accepted.",
// 验证错误信息其它部分...
```

当创建一个自定义验证规则时，你可能有时候需要为错误信息定义自定义占位符，可以通过创建自定义验证器然后调用 Validator 门面上的 replacer 方法来实现。可以在服务提供者的 boot 方法中编写代码：

```
/**
 * 启动应用服务
 *
 * @return void
 */
public function boot(){
    Validator::extend(...);
    Validator::replacer('foo', function($message, $attribute, $rule, $parameters) {
        return str_replace(...);
    });
}
```