# Laravel Cashier（订购&支付&发票）

# 1、简介
Laravel Cashier 为 Stripe 的订购单据服务提供了一个优雅的、平滑的接口。它处理了几乎所有你恐惧编写的样板化的订购单据代码。除了基本的订购管理外，Cashier 还支持处理优惠券、交换订购、订购“数量”、取消宽限期，甚至生成PDF发票。
## 1.1 配置
### 1.1.1 Composer
首先，添加 Cashier 包到 composer.json 文件并运行 composer update 命令：

```
"laravel/cashier": "~5.0" (For Stripe SDK ~2.0, and Stripe APIs on 2015-02-18 version and later)
"laravel/cashier": "~4.0" (For Stripe APIs on 2015-02-18 version and later)
"laravel/cashier": "~3.0" (For Stripe APIs up to and including 2015-02-16 version)
```

### 1.1.2 服务提供者
接下来，在 app 配置文件中注册服务提供者Laravel\Cashier\CashierServiceProvider。
### 1.1.3 迁移
实现 Cashier 之前，我们需要添加额外的字段到数据库。别担心，你可以使用 Artisan 命令 cashier:table 来创建迁移以添加必要的字段。例如，要添加该字段到用户表运行如下命令：

```
php artisan cashier:table users
```

创建好迁移后，只需简单运行 migrate 命令。
### 1.1.4 设置模型
接下来，添加 Billable trait 和相应的日期调整器到模型定义：

```
use Laravel\Cashier\Billable;
use Laravel\Cashier\Contracts\Billable as BillableContract;

class User extends Model implements BillableContract{
    use Billable;
    protected $dates = ['trial_ends_at', 'subscription_ends_at'];
}
```

添加字段到模型的$dates 属性使 Eloquent 返回 Carbon/Datetime 实例格式字段而不是原生字符串。
### 1.1.5 Stripe 键
最后，在配置文件 services.php 中设置 Stripe 键：

```
'stripe' => [
    'model'  => 'User',
    'secret' => env('STRIPE_API_SECRET'),
],
```

# 2、订购
## 2.1 创建订购
要创建一个订购，首先获取一个账单模型的实例，通常是 App\User 的实例。获取到该模型实例之后，你可以使用 subscription 方法来管理模型的订购：

```
$user = User::find(1);
$user->subscription('monthly')->create($creditCardToken);
```

create 方法将会自动创建 Stripe 订购，并使用 Stripe 客户 ID 和其他相关单据信息更新数据库。如果你在 Stripe 中的计划有一个试用配置，那么用户记录中的试用结束时间也会自动设置。
如果你想要实现试用期，但是完全在 Laravel 应用中管理试用而不是将其定义在 Stripe 中，你必须手动设置试用结束时间：

```
$user->trial_ends_at = Carbon::now()->addDays(14);
$user->save();
```

### 2.1.1 额外的用户详情
如果你想要指定额外的客户详情，你可以将其作为第二个参数传递给 create 方法：

```
$user->subscription('monthly')->create($creditCardToken, [
    'email' => $email, 
    'description' => 'Our First Customer'
]);
```

要了解更多 Stripe 支持的字段，可以查看 Stripe 关于客户创建的文档。
### 2.1.2 优惠券
如果你想要在创建订购的时候使用优惠券，可以使用 withCoupon 方法：

```
$user->subscription('monthly')
     ->withCoupon('code')
     ->create($creditCardToken);
```

## 2.2 检查订购状态
用户订购你的应用后，你可以使用各种便利的方法来简单检查订购状态。首先，如果用户有一个有效的订购，则 subscribed 方法返回 true，即使订购现在出于试用期：

```
if ($user->subscribed()) {
    //
}
```

subscribed 方法还可以用于路由中间件，基于用户订购状态允许你对路由和控制器的访问进行过滤：

```
public function handle($request, Closure $next){
    if ($request->user() && ! $request->user()->subscribed()) {
        // This user is not a paying customer...
        return redirect('billing');
    }

    return $next($request);
}
```

如果你想要判断一个用户是否还在试用期，可以使用 onTrial 方法，该方法在为还处于试用期的用户显示警告信息很有用：

```
if ($user->onTrial()) {
    //
}
```

onPlan 方法可用于判断用户是否基于 Stripe ID 订购了给定的计划：

```
if ($user->onPlan('monthly')) {
    //
}
```

### 2.2.1 取消的订购状态
要判断用户是否曾经是有效的订购者，但现在取消了订购，可以使用 cancelled 方法：

```
if ($user->cancelled()) {
    //
}
```

你还可以判断用户是否曾经取消过订购，但现在仍然在“宽限期”直到订购完全失效。例如，如果一个用户在 3 月 5 号取消了一个实际有效期到 3 月 10 号的订购，该用户处于“宽限期”直到 3 月 10 号。注意 subscribed 方法在此期间仍然返回 true。

```
if ($user->onGracePeriod()) {
    //
}
```

everSubscribed 方法可以用于判断用户是否订购过应用的计划：

```
if ($user->everSubscribed()) {
    //
}
```

## 2.3 改变计划
用户订购应用后，偶尔想要改变到新的订购计划，要将用户切换到新的订购，使用 swap 方法。例如，我们可以轻松切换用户到 premium 订购：

```
$user = App\User::find(1);
$user->subscription('premium')->swap();
```

如果用户在试用，试用期将会被维护。还有，如果订购存在数量，数量也可以被维护。当交换订购计划，还可以使用 prorate 方法来表明费用是按比例的，此外，你可以使用 swapAndInvoice 方法立即为用户计划改变开发票：

```
$user->subscription('premium')
            ->prorate()
            ->swapAndInvoice();
```

## 2.4 订购数量
有时候订购也会被数量影响，例如，你的应用每个账户每月需要付费$10，要简单增加或减少订购数量，使用 increment 和 decrement 方法：

```
$user = User::find(1);

$user->subscription()->increment();
// 当前订购数量+5...
$user->subscription()->increment(5);

$user->subscription()->decrement();
// 当前订购数量-5...
$user->subscription()->decrement(5);
```

你也可以使用 updateQuantity 方法指定数量：

```
$user->subscription()->updateQuantity(10);
```

想要了解更多订购数量信息，查阅相关Stripe 文档。
## 2.5 订购税金
在 Cashier 中，提供 tax_percent 值发送给 Stripe 很简单。要指定用户支付订购的税率，实现账单模型的 getTaxPercent 方法，并返回一个在 0 到 100 之间的数值，不要超过两位小数：

```
public function getTaxPercent() {
    return 20;
}
```

这将使你可以在模型基础上使用税率，对跨越不同国家的用户很有用。
## 2.6 取消订购
要取消订购，可以调用用户订购上的 cancel 方法：

```
$user->subscription()->cancel();
```

当订购被取消时，Cashier 将会自动设置数据库中的 subscription_ends_at 字段。该字段用于了解 subscribed 方法什么时候开始返回 false。例如，如果客户 3 月 1 号份取消订购，但订购直到 3 月 5 号才会结束，那么 subscribed 方法继续返回 true 直到 3 月 5 号。
你可以使用 onGracePeriod 方法判断用户是否已经取消订购但仍然在“宽限期”：

```
if ($user->onGracePeriod()) {
    //
}
```

## 2.7 恢复订购
如果用户已经取消订购但你想恢复，使用 resume 方法：

```
$user->subscription('monthly')->resume($creditCardToken);
```

如果用户取消订购然后在订购失效前恢复，将不会立即付款，相反，他们的订购会重新激活，他们将会在正常的付款周期进行支付。
# 3、处理 Stripe
## 3.1 失败的订购
如果客户的信用卡失效怎么办？不用担心——Cashier 包含了 Webhook 控制器，该控制器可以轻易的为你取消客户订购。只需要设置下到该控制器的路由：

```
Route::post('stripe/webhook', 'Laravel\Cashier\WebhookController@handleWebhook');
```

就这样！失败的支付将会被该控制器捕获和处理。当 Stripe 判断订购失败（正常情况下尝试支付失败三次后）时该控制器将会取消客户的订购。不要忘了：你需要在 Stripe 控制面板设置中配置 webhook URI。
由于 Stripe webhooks 需要通过 Laravel 的CSRF 验证，确保将该 URI 置于 VerifyCsrfToken 中间件排除列表中：

```
protected $except = [
    'stripe/*',
];
```

## 3.2 其它 Webhooks
如果你有额外想要处理的 Stripe webhook 事件，只需简单继承 Webhook 控制器， 你的方法名应该和 Cashier 期望的约定一致，尤其是方法应该以“handle”开头并以驼峰命名法命名。例如，如果你想要处理 invoice.payment_succeeded webhook，你应该添加 handleInvoicePaymentSucceeded 方法到控制器：

```
<?php

namespace App\Http\Controller;

use Laravel\Cashier\WebhookController as BaseController;

class WebhookController extends BaseController{
    /**
     * 处理 stripe webhook.
     *
     * @param  array  $payload
     * @return Response
     */
    public function handleInvoicePaymentSucceeded($payload)
    {
        // 处理该事件
    }
}
```

# 4、一次性付款
如果你想要使用订购客户的信用卡一次性结清账单，可以使用单据模型实例上的 charge 方法，该方法接收付款金额（应用使用的货币的最小单位对应的金额数值）作为参数，例如，下面的例子要使用信用卡支付 100 美分，或 1 美元：

```
$user->charge(100);
```

charge 方法接收一个数组作为第二个参数，允许你传递任何你想要传递的底层 Stripe 账单创建参数：

```
$user->charge(100, [
    'source' => $token,
    'receipt_email' => $user->email,]);
```

如果支付失败 charge 方法将返回 false，这通常表明付款被拒绝：

```
if ( ! $user->charge(100)) {
    // The charge was denied...
}
```

如果支付成功，该方法将会返回一个完整的 Stripe 响应。
# 5、发票
你可以使用 invoices 方法轻松获取账单模型的发票数组：

```
$invoices = $user->invoices();
```

当列出客户发票时，你可以使用发票的帮助函数来显示相关的发票信息。例如，你可能想要在表格中列出每张发票，从而允许用户方便的下载它们：

```
<table>
    @foreach ($invoices as $invoice)
        <tr>
            <td>{{ $invoice->dateString() }}</td>
            <td>{{ $invoice->dollars() }}</td>
            <td><a href="/user/invoice/{{ $invoice->id }}">Download</a></td>
        </tr>
    @endforeach
</table>
```

## 5.1 生成 PDF 发票
在路由或控制器中，使用 downloadInvoice 方法生成发票的 PDF 下载，该方法将会自动生成合适的 HTTP 响应发送下载到浏览器：

```
Route::get('user/invoice/{invoice}', function ($invoiceId) {
    return Auth::user()->downloadInvoice($invoiceId, [
        'vendor'  => 'Your Company',
        'product' => 'Your Product',
    ]);
});
```