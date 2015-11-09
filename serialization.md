# 序列化

## 1、简介
当构建 JSON API 时，经常需要转化模型和关联关系为数组或 JSON。Eloquent 包含便捷方法实现这些转换，以及控制哪些属性被包含到序列化中。

## 2、基本使用

### 2.1 转化模型为数组
要转化模型及其加载的关联关系为数组，可以使用 toArray 方法。这个方法是递归的，所以所有属性及其关联对象属性（包括关联的关联）都会被转化为数组：

```
$user = App\User::with('roles')->first();
return $user->toArray();
```

还可以转化集合为数组：

```
$users = App\User::all();
return $users->toArray();
```

### 2.2 转化模型为 JSON
要转化模型为 JSON，可以使用 toJson 方法，和 toArray 一样，toJson 方法也是递归的，所有属性及其关联属性都会被转化为 JSON：

```
$user = App\User::find(1);
return $user->toJson();
```

你还可以转化模型或集合为字符串，这将会自动调用 toJson 方法：

```
$user = App\User::find(1);
return (string) $user;
```

由于模型和集合在转化为字符串的时候会被转化为 JSON，你可以从应用的路由或控制器中直接返回 Eloquent 对象：

```
Route::get('users', function () {
    return App\User::all();
});
```

## 3、在 JSON 中隐藏属性显示
有时候你希望在模型数组或 JSON 显示中限制某些属性，比如密码，要实现这个，在定义模型的时候添加一个`$hidden` 属性：

```
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class User extends Model{
    /**
     * 在数组中隐藏的属性
     *
     * @var array
     */
    protected $hidden = ['password'];
}
```

注意：如果要隐藏关联关系，使用关联关系的方法名，而不是动态属性名。
此外，可以使用 visible 属性定义属性显示的白名单：

```
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class User extends Model{
    /**
     * 在数组中显示的属性
     *
     * @var array
     */
    protected $visible = ['first_name', 'last_name'];
}
```

## 4、追加值到 JSON
有时候，需要添加数据库中没有相应的字段到数组中，要实现这个，首先要定义一个访问器：

```
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class User extends Model{
    /**
     * 为用户获取管理员标识
     *
     * @return bool
     */
    public function getIsAdminAttribute()
    {
        return $this->attributes['admin'] == 'yes';
    }
}
```

定义好访问器后，添加字段名到模型的 appends 属性：

```
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class User extends Model{
    /**
     * 追加到模型数组表单的访问器
     *
     * @var array
     */
    protected $appends = ['is_admin'];
}
```

字段被添加到 appends 列表之后，将会被包含到模型数组和 JSON 表单中，appends 数组中的字段还会遵循模型中的 visible 和 hidden 设置配置。