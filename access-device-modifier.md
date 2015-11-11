# 调整器

## 1、简介
访问器和调整器允许你在获取模型属性或设置其值时格式化 Eloquent 属性。例如，你可能想要使用 Laravel 加密器对存储在数据库中的数据进行加密，并且在 Eloquent 模型中访问时自动进行解密。

除了自定义访问器和调整器，Eloquent 还可以自动转换日期字段为 Carbon 实例甚至将文本转换为 JSON。

## 2、访问器 & 调整器

### 2.1 定义访问器
要定义一个访问器，需要在模型中创建一个` getFooAttribute `方法，其中 `Foo `是你想要访问的字段名（使用驼峰式命名规则）。在本例中，我们将会为` first_name` 属性定义一个访问器，该访问器在获取 `first_name` 的值时被 Eloquent 自动调用：

```
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class User extends Model{
    /**
     * 获取用户的名字
     *
     * @param  string  $value
     * @return string
     */
    public function getFirstNameAttribute($value)
    {
        return ucfirst($value);
    }
}
```

正如你所看到的，该字段的原生值被传递给访问器，然后返回处理过的值。要访问该值只需要简单访问 `first_name` 即可：

```
$user = App\User::find(1);
$firstName = $user->first_name;
```

### 2.2 定义调整器
要定义一个调整器，需要在模型中定义 `setFooAttribute` 方法，其中 `Foo` 是你想要访问的字段（使用驼峰式命名规则）。接下来让我们为 `first_name` 属性定义一个调整器，当我们为模型上的 `first_name` 赋值时该调整器会被自动调用：

```
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class User extends Model{
    /**
     * 设置用户的名字
     *
     * @param  string  $value
     * @return string
     */
    public function setFirstNameAttribute($value)
    {
        $this->attributes['first_name'] = strtolower($value);
    }
}
```

该调整器获取要被设置的属性值，允许你操纵该值并设置 Eloquent 模型内部属性值为操作后的值。例如，如果你尝试设置 `Sally` 的 `first_name` 属性：

```
$user = App\User::find(1);
$user->first_name = 'Sally';
```

在本例中，`setFirstNameAttribute` 方法会被调用，传入参数为 `Sally`，调整器会对其调用 `strtolower` 函数并将处理后的值设置为内部属性的值。

## 3、日期调整器
默认情况下，Eloquent 将会转化 `created_at` 和 `updated_at` 列的值为 Carbon 实例，该类继承自 PHP 原生的 `Datetime `类，并提供了各种有用的方法。

你可以自定义哪些字段被自动调整修改，甚至可以通过重写模型中的`$dates` 属性完全禁止调整：

```
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class User extends Model{
    /**
     * 应该被调整为日期的属性
     *
     * @var array
     */
    protected $dates = ['created_at', 'updated_at', 'disabled_at'];
}
```

如果字段是日期格式时，你可以将其值设置为 UNIX 时间戳，日期字符串（`Y-m-d`），日期-时间字符串，`Datetime/Carbon `实例，日期的值将会自动以正确格式存储到数据库中：

```
$user = App\User::find(1);
$user->disabled_at = Carbon::now();
$user->save();
```

正如上面提到的，当获取被罗列在`$dates` 数组中的属性时，它们会被自动转化为 `Carbon `实例，允许你在属性上使用任何` Carbon `的方法：

```
$user = App\User::find(1);
return $user->disabled_at->getTimestamp();
```

如果你需要自定义时间戳格式，在模型中设置`$dateFormat` 属性，该属性决定日期属性将以何种格式存储在数据库中、以及序列化为数组或 JSON 时的格式：

```
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class Flight extends Model{
    /**
     * 模型日期的存储格式
     *
     * @var string
     */
    protected $dateFormat = 'U';
}
```

# 4、属性转换
模型中的`$casts` 属性提供了便利方法转换属性到通用数据类型。`$casts` 属性是数组格式，其键是要被转换的属性名称，其值时你想要转换的类型。目前支持的转换类型包括：`integer`, `real`, `float`, `double`, `string`, `boolean`, `object` 和 `array`。
例如，让我们转换 `is_admin` 属性，将其由 `integer` 类型转换为 `boolean` 类型：

```
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class User extends Model{
    /**
     * 应该被转化为原生类型的属性
     *
     * @var array
     */
    protected $casts = [
        'is_admin' => 'boolean',
    ];
}
```

现在，`is_admin `属性在被访问时总是被转换为 `boolean`，即使底层存储在数据库中的值是 `integer`：

```
$user = App\User::find(1);

if ($user->is_admin) {
    //
}
```

### 4.1 数组转换
`array `类型转换在处理被存储为序列化 JSON 的字段是特别有用，例如，如果数据库有一个 TEXT 字段类型包含了序列化 JSON，添加 `array` 类型转换到该属性将会在 Eloquent 模型中访问其值时自动将其反序列化为 PHP 数组：

```
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class User extends Model{
    /**
     * 应该被转化为原生类型的属性
     *
     * @var array
     */
    protected $casts = [
        'options' => 'array',
    ];
}
```

类型转换被定义后，就可以访问 `options` 属性，它将会自动从 JSON 反序列化为 PHP 数组，当你设置` options` 属性的值时，给定数组将会自动转化为 JSON 以供存储：

```
$user = App\User::find(1);
$options = $user->options;
$options['key'] = 'value';
$user->options = $options;
$user->save();
```