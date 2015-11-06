# 邮件

# 1、简介
Laravel 基于目前流行的 SwiftMailer 库提供了一套干净清爽的邮件 API。Laravel 为 SMTP、Mailgun、Mandrill、Amazon SES、PHP 的 mail 函数，以及 sendmail 提供了驱动，从而允许你快速通过本地或云服务发送邮件。
## 1.1 邮件驱动预备知识
基于驱动的 API 如 Mailgun 和 Mandrill 通常比 SMTP 服务器更简单、更快。所有的 API 驱动要求应用已经安装 Guzzle HTTP 库。你可以通过添加如下行到 composer.json 文件来安装 Guzzle 到项目：

```
"guzzlehttp/guzzle": "~5.3|~6.0"
```

### 1.1.1 Mailgun 驱动
要使用 Mailgun 驱动，首先安装 Guzzle，然后在配置文件 config/mail.php 中设置 driver 选项为 mailgun。接下来，验证配置文件 config/services.php 包含如下选项：

```
'mailgun' => [
    'domain' => 'your-mailgun-domain',
    'secret' => 'your-mailgun-key',],
```

### 1.1.2 Mandrill 驱动
要使用 Mandrill 驱动，首先安装 Guzzle，然后在配置文件 config/mail.php 中设置 driver 选项值为 mandrill。接下来，验证配置文件 config/services.php 包含如下选项：

```
'mandrill' => [
    'secret' => 'your-mandrill-key',],
```

### 1.1.3 SES 驱动
要使用 Amazon SES 驱动，安装 Amazon AWS 的 PHP SDK，你可以通过添加如下行到 composer.json 文件的 require 部分来安装该库：

```
"aws/aws-sdk-php": "~3.0"
```

接下来，设置配置文件 config/mail.php 中的 driver 选项为 ses。然后，验证配置文件 config/services.php 包含如下选项：

```
'ses' => [
    'key' => 'your-ses-key',
    'secret' => 'your-ses-secret',
    'region' => 'ses-region',  // e.g. us-east-1
],
```

# 2、发送邮件
Laravel 允许你在视图中存储邮件信息，例如，要组织你的电子邮件，可以在 resources/views 目录下创建 emails 目录。
要发送一条信息，使用 Mail门面上的 send 方法。send 方法接收三个参数。第一个参数是包含邮件信息的视图名称；第二个参数是你想要传递到该视图的数组数据；第三个参数是接收消息实例的闭包回调——允许你自定义收件人、主题以及邮件其他方面的信息：

```
<?php

namespace App\Http\Controllers;

use Mail;
use App\User;
use Illuminate\Http\Request;
use App\Http\Controllers\Controller;

class UserController extends Controller{
    /**
     * 发送邮件给用户
     *
     * @param  Request  $request
     * @param  int  $id
     * @return Response
     */
    public function sendEmailReminder(Request $request, $id)
    {
        $user = User::findOrFail($id);

        Mail::send('emails.reminder', ['user' => $user], function ($m) use ($user) {
            $m->to($user->email, $user->name)->subject('Your Reminder!');
        });
    }
}
```

由于我们在上例中传递一个包含 user 键的数组，我们可以在邮件中使用如下方式显示用户名：

```
<?php echo $user->name; ?>
```

注意：$message 变量总是被传递到邮件视图，并允许嵌入附件，因此，你应该在视图负载中避免传入消息变量。
**构造消息**
正如前面所讨论的，传递给 send 方法的第三个参数是一个允许你指定邮件消息本身多个选项的闭包。使用这个闭包可以指定消息的其他属性，例如抄送、群发，等等：

```
Mail::send('emails.welcome', $data, function ($message) {
    $message->from('us@example.com', 'Laravel');
    $message->to('foo@example.com')->cc('bar@example.com');
});
```

下面试$message 消息构建器实例上的可用方法：

```
$message->from($address, $name = null);
$message->sender($address, $name = null);
$message->to($address, $name = null);
$message->cc($address, $name = null);
$message->bcc($address, $name = null);
$message->replyTo($address, $name = null);
$message->subject($subject);
$message->priority($level);
$message->attach($pathToFile, array $options = []);
// 从$data 字符串追加文件...
$message->attachData($data, $name, array $options = []);
// 获取底层SwiftMailer消息实例...
$message->getSwiftMessage();
```

注意：传递给 Mail::send 闭包的消息实例继承自 SwiftMailer 消息类，该实例允许你调用该类上的任何方法来构建自己的电子邮件消息。
**纯文本邮件**
默认情况下，传递给 send 方法的视图假定包含 HTML，然而，通过传递数组作为第一个参数到 send 方法，你可以指定发送除 HTML 视图之外的纯文本视图：

```
Mail::send(['html.view', 'text.view'], $data, $callback);
```

或者，如果你只需要发送纯文本邮件，可以指定在数组中使用 text 键：

```
Mail::send(['text' => 'view'], $data, $callback);
```

**原生字符串邮件**
如果你想要直接发送原生字符串邮件你可以使用 raw 方法：

```
Mail::raw('Text to e-mail', function ($message) {
    //
});
```

## 2.1 附件
要添加附件到邮件，使用传递给闭包的$message 对象上的 attach 方法。该方法接收文件的绝对路径作为第一个参数：

```
Mail::send('emails.welcome', $data, function ($message) {
    //
    $message->attach($pathToFile);
});
```

当添加文件到消息时，你还可以通过传递数组作为第二个参数到 attach 方法来指定文件显示名和 MIME 类型：

```
$message->attach($pathToFile, ['as' => $display, 'mime' => $mime]);
```

## 2.2 内联附件
### 2.2.1 在邮件视图中嵌入一张图片
嵌套内联图片到邮件中通常是很笨重的，然而，Laravel 提供了一个便捷的方式附加图片到邮件并获取相应的 CID，要嵌入内联图片，在邮件视图中使用$message 变量上的 embed 方法。记住，Laravel 自动在所有邮件视图中传入$message 变量使其有效：

```
<body>
    Here is an image:
    <img src="<?php echo $message->embed($pathToFile); ?>">
</body>
```

### 2.2.2 在邮件视图中嵌入原生数据
如果你想要在邮件消息中嵌入原生数据字符串，可以使用$message 变量上的 embedData 方法：

```
<body>
    Here is an image from raw data:
    <img src="<?php echo $message->embedData($data, $name); ?>">
</body>
```

## 2.3 邮件队列
### 2.3.1 邮件消息队列
发送邮件消息可能会大幅度延长应用的响应时间，许多开发者选择将邮件发送放到队列中再后台执行，Laravel 中可以使用内置的统一队列 API来实现。要将邮件消息放到队列中，使用 Mail 门面上的 queue 方法：

```
Mail::queue('emails.welcome', $data, function ($message) {
    //
});
```

该方法自动将邮件任务推送到队列中以便在后台发送。当然，你需要在使用该特性前配置队列。
### 2.3.2 延迟消息队列
如果你想要延迟已经放到队列中邮件的发送，可以使用 later 方法。只需要传递你想要延迟发送的秒数作为第一个参数到该方法即可：

```
Mail::later(5, 'emails.welcome', $data, function ($message) {
    //
});
```

### 2.3.3 推入指定队列
如果你想要将邮件消息推送到指定队列，可以使用 queueOn 和 laterOn 方法：

```
Mail::queueOn('queue-name', 'emails.welcome', $data, function ($message) {
    //
});

Mail::laterOn('queue-name', 5, 'emails.welcome', $data, function ($message) {
    //
});
```

# 3、邮件&本地开发
开发发送邮件的应用时，你可能不想要真的发送邮件到有效的电子邮件地址，而只是想要做下测试。Laravel 提供了几种方式“禁止”邮件的实际发送。
## 3.1 日志驱动
一种解决方案是在本地开发时使用 log 邮件驱动。该驱动将所有邮件信息写到日志文件中以备查看，想要了解更多关于每个环境的应用配置信息，查看配置文档。
## 3.2 通用配置
Laravel 提供的另一种解决方案是为框架发送的所有邮件设置通用收件人，这样的话，所有应用生成的邮件将会被发送到指定地址，而不是实际发送邮件指定的地址。这可以通过在配置文件 config/mail.php 中设置 to 选项来实现：

```
'to' => [
    'address' => 'dev@domain.com',
    'name' => 'Dev Example'
],
```

## 3.3 Mailtrap
最后，你可以使用 Mailtrap 服务和 smtp 驱动发送邮件信息到“虚拟”邮箱，这种方法允许你在 Mailtrap 的消息查看器中查看最终的邮件。