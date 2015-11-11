# 文件系统/云存储

## 1、简介
基于 `Frank de Jonge `的 `PHP `包 `Flysystem，Laravel `提供了强大的文件系统抽象。`Laravel `文件系统集成提供了使用驱动处理本地文件系统的简单使用，这些驱动包括 `Amazon S3`，以及 `Rackspace `云存储。更好的是在这些存储选项间切换非常简单，因为对每个系统而言，API 是一样的。

## 2、配置
文件系统配置文件位于 `config/filesystems.php`。在该文件中可以配置所有”硬盘“，每个硬盘描述了特定的存储驱动和存储位置。为每种支持的驱动的示例配置包含在该配置文件中，所以，简单编辑该配置来反映你的存储参数和认证信息。

当然，你想配置磁盘多少就配置多少，多个磁盘也可以共用同一个驱动。

### 2.1 本地驱动
使用 `local `驱动的时候，注意所有文件操作相对于定义在配置文件中的 `root `目录，默认情况下，该值设置为 `storage/app `目录，因此，下面的方法将会存储文件到 `storage/app/file.txt`：

```
Storage::disk('local')->put('file.txt', 'Contents');
```

### 2.2 其它驱动预备知识
在使用 `Amazon S3` 或 `Rackspace `驱动之前，需要通过 `Composer `安装相应的包：

- 	Amazon S3: league/flysystem-aws-s3-v3 ~1.0 
- 	Rackspace: league/flysystem-rackspace ~1.0

## 3、基本使用

### 3.1 获取硬盘实例
`Storage`门面用于和你配置的所有磁盘进行交互，例如，你可以使用该门面上的 `put `方法来存储头像到默认磁盘，如果你调用 `Storage `门面上的方法却先调用 `disk `方法，该方法调用自动传递到默认磁盘：

```
<?php

namespace App\Http\Controllers;

use Storage;
use Illuminate\Http\Request;
use App\Http\Controllers\Controller;

class UserController extends Controller{
    /**
     * 更新指定用户头像
     *
     * @param  Request  $request
     * @param  int  $id
     * @return Response
     */
    public function updateAvatar(Request $request, $id)
    {
        $user = User::findOrFail($id);

        Storage::put(
            'avatars/'.$user->id,
            file_get_contents($request->file('avatar')->getRealPath())
        );
    }
}
```

使用多个磁盘时，可以使用 `Storage `门面上的 `disk `方法访问特定磁盘。当然，可以继续使用方法链执行该磁盘上的方法：

```
$disk = Storage::disk('s3');
$contents = Storage::disk('local')->get('file.jpg')
```

### 3.2 获取文件
get 方法用于获取给定文件的内容，该方法将会返回该文件的原生字符串内容：

```
$contents = Storage::get('file.jpg');
```

`exists` 方法用于判断给定文件是否存在于磁盘上：

```
$exists = Storage::disk('s3')->exists('file.jpg');
```

#### 3.2.1 文件元信息
size 方法以字节方式返回文件大小：

```
$size = Storage::size('file1.jpg');
```

`lastModified` 方法以 `UNIX `时间戳格式返回文件最后一次修改时间：

```
$time = Storage::lastModified('file1.jpg');
```

### 3.3 存储文件
`put` 方法用于存储文件到磁盘。可以传递一个 `PHP `资源到 `put `方法，该方法将会使用 `Flysystem` 底层的流支持。在处理大文件的时候推荐使用文件流：

```
Storage::put('file.jpg', $contents);
Storage::put('file.jpg', $resource);
```

`copy` 方法将磁盘中已存在的文件从一个地方拷贝到另一个地方：

```
Storage::copy('old/file1.jpg', 'new/file1.jpg');
```

`move` 方法将磁盘中已存在的文件从一定地方移到到另一个地方：

```
Storage::move('old/file1.jpg', 'new/file1.jpg');
```

#### 3.3.1 添加内容到文件开头/结尾
`prepend` 和 `append` 方法允许你轻松插入内容到文件开头/结尾：

```
Storage::prepend('file.log', 'Prepended Text');
Storage::append('file.log', 'Appended Text');
```

### 3.4 删除文件
`delete` 方法接收单个文件名或多个文件数组并将其从磁盘移除：

```
Storage::delete('file.jpg');
Storage::delete(['file1.jpg', 'file2.jpg']);
```

### 3.5 目录

#### 3.5.1 获取一个目录下的所有文件
`files` 方法返回给定目录下的所有文件数组，如果你想要获取给定目录下包含子目录的所有文件列表，可以使用 `allFiles `方法：

```
$files = Storage::files($directory);
$files = Storage::allFiles($directory);
```

#### 3.5.2 获取一个目录下的所有子目录
`directories `方法返回给定目录下所有目录数组，此外，可以使用 `allDirectories `方法获取嵌套的所有子目录数组：

```
$directories = Storage::directories($directory);
// 递归...
$directories = Storage::allDirectories($directory);
```

#### 3.5.3 创建目录
`makeDirectory` 方法将会创建给定目录，包含子目录（递归）：

```
Storage::makeDirectory($directory);
```

#### 3.5.4 删除目录
最后，`deleteDirectory `方法用于移除目录，包括该目录下的所有文件：

```
Storage::deleteDirectory($directory);
```

## 4、自定义文件系统
`Laravel `的 `Flysystem `集成支持自定义驱动，为了设置自定义的文件系统你需要创建一个服务提供者如 `DropboxServiceProvider`。在该提供者的 `boot `方法中，你可以使用 `Storage `门面的 `extend `方法定义自定义驱动：

```
<?php

namespace App\Providers;

use Storage;
use League\Flysystem\Filesystem;
use Dropbox\Client as DropboxClient;
use Illuminate\Support\ServiceProvider;
use League\Flysystem\Dropbox\DropboxAdapter;

class DropboxServiceProvider extends ServiceProvider{
    /**
     * Perform post-registration booting of services.
     *
     * @return void
     */
    public function boot()
    {
        Storage::extend('dropbox', function($app, $config) {
            $client = new DropboxClient(
                $config['accessToken'], $config['clientIdentifier']
            );

            return new Filesystem(new DropboxAdapter($client));
        });
    }

    /**
     * Register bindings in the container.
     *
     * @return void
     */
    public function register()
    {
        //
    }
}
```

`extend `方法的第一个参数是驱动名称，第二个参数是获取`$app` 和`$config` 变量的闭包。该解析器闭包必须返回一个 `League\Flysystem\Filesystem `实例。`$config `变量包含了定义在配置文件 `config/filesystems.php `中为特定磁盘定义的选项。

创建好注册扩展的服务提供者后，就可以使用配置文件 `config/filesystem.php `中的 `dropbox `驱动了。