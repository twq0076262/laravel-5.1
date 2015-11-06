# 加密

# 1、配置
在使用 Laravel 的加密器之前，应该在 config/app.php 配置文件中设置 key 选项为 32 位随机字符串。如果这个值没有被设置，所有 Laravel 加密过的值都是不安全的。
# 2、基本使用
## 2.1 加密
你可以使用 Crypt 门面对数据进行加密，所有加密值都使用 OpenSSL 和 AES-256-CBC 密码进行加密。此外，所有加密值都通过一个消息认证码（MAC）来检测对加密字符串的任何修改。
例如，我们可以使用 encrypt 方法加密 secret 属性并将其存储到 Eloquent 模型：

```
<?php

namespace App\Http\Controllers;

use Crypt;
use App\User;
use Illuminate\Http\Request;
use App\Http\Controllers\Controller;

class UserController extends Controller{
    /**
     * Store a secret message for the user.
     *
     * @param  Request  $request
     * @param  int  $id
     * @return Response
     */
    public function storeSecret(Request $request, $id)
    {
        $user = User::findOrFail($id);

        $user->fill([
            'secret' => Crypt::encrypt($request->secret)
        ])->save();
    }
}
```

## 2.2 解密
当然，你可以使用 Crypt 门面上的 decrypt 方法进行解密。如果该值不能被解密，例如 MAC 无效，将会抛出一个 Illuminate\Contracts\Encryption\DecryptException 异常：

```
use Illuminate\Contracts\Encryption\DecryptException;

try {
    $decrypted = Crypt::decrypt($encryptedValue);
} catch (DecryptException $e) {
    //
}
```