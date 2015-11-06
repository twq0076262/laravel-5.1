# 哈希

# 1、简介
Laravel Hash 门面为存储用户密码提供了安全的 Bcrypt 哈希算法。如果你正在使用 Laravel 应用自带的 AuthController 控制器，将会自动为注册和认证使用该 Bcrypt。
Bcrypt 是散列密码的绝佳选择，因为其”工作因子“是可调整的，这意味着随着硬件功能的提升，生成哈希所花费的时间也会增加。
# 2、基本使用
可以调用 Hash 门面上的 make 方法散列存储密码：

```
<?php

namespace App\Http\Controllers;

use Hash;
use App\User;
use Illuminate\Http\Request;
use App\Http\Controllers\Controller;

class UserController extends Controller{
    /**
     * 更新用户密码
     *
     * @param  Request  $request
     * @param  int  $id
     * @return Response
     */
    public function updatePassword(Request $request, $id)
    {
        $user = User::findOrFail($id);

        // 验证新密码长度...

        $user->fill([
            'password' => Hash::make($request->newPassword)
        ])->save();
    }
}
```

此外，还可以使用全局的帮助函数 bcrypt：

```
bcrypt('plain-text');
```

## 2.1 验证哈希密码
check 方法允许你验证给定原生字符串和给定哈希是否相等，然而，如果你在使用 Laravel 自带的 AuthController（详见用户认证一节），就不需要再直接使用该方法，因为自带的认证控制器自动调用了该方法：

```
if (Hash::check('plain-text', $hashedPassword)) {
    // 密码匹配...
}
```

## 2.2 检查密码是否需要被重新哈希
needsRehash 方法允许你判断哈希计算器使用的工作因子在上次密码被哈希后是否发生改变：

```
if (Hash::needsRehash($hashed)) {
    $hashed = Hash::make('plain-text');
}
```