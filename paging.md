# 分页

# 1、简介
在其他框架中，分页是件非常痛苦的事，Laravel 则使其变得轻而易举。Laravel 能够基于当前页智能生成一定范围的链接，且生成的 HTML 兼容 Bootstrap CSS 框架。
# 2、基本使用
## 2.1 基于查询构建器分页
有多种方式实现分页，最简单的方式就是使用查询构建器或 Eloquent 模型的 paginate 方法。该方法基于当前用户查看页自动设置合适的偏移（offset）和限制（limit）。默认情况下，当前页通过 HTTP 请求查询字符串参数?page 的值判断。当然，该值由 Laravel 自动检测，然后自动插入分页器生成的链接中。
让我们先来看看如何在查询上调用 paginate 方法。在本例中，传递给 paginate 的唯一参数就是你每页想要显示的数目，这里我们指定每页显示 15 个：

```
<?php

namespace App\Http\Controllers;

use DB;
use App\Http\Controllers\Controller;

class UserController extends Controller{
    /**
     * 显示应用中的所有用户
     *
     * @return Response
     */
    public function index()
    {
        $users = DB::table('users')->paginate(15);
        return view('user.index', ['users' => $users]);
    }
}
```

注意：目前，使用 groupBy 的分页操作不能被 Laravel 有效执行，如果你需要在分页结果中使用 groupBy，推荐你手动查询数据库然后创建分页器。
### 2.1.1 “简单分页”
如果你只需要在分页视图中简单的显示“下一个”和“上一个”链接，可以使用 simplePaginate 方法来执行该查询。在渲染包含大数据集的视图且不需要显示每个页码时非常有用：

```
$users = DB::table('users')->simplePaginate(15);
```

## 2.2 基于Eloquent模型分页
你还可以对 Eloquent 查询结果进行分页，在本例中，我们对 User 模型进行分页，每页显示 15 条记录。正如你所看到的，该语法和基于查询构建器的分页差不多：

```
$users = App\User::paginate(15);
```

当然，你可以在设置其它约束调价之后调用 paginate，比如 where 子句：

```
$users = User::where('votes', '>', 100)->paginate(15);
```

你也可以使用 simplePaginate 方法：

```
$users = User::where('votes', '>', 100)->simplePaginate(15);
```

## 2.3 手动创建分页器
有时候你可能想要通过传递数组数据来手动创建分页实例，你可以基于自己的需求通过创建 Illuminate\Pagination\Paginator 或 Illuminate\Pagination\LengthAwarePaginator 实例来实现。
Paginator 类不需要知道结果集中数据项的总数；然而，正因如此，该类也没有提供获取最后一页索引的方法。
LengthAwarePaginator 接收参数和 Paginator 几乎一样，只是，它要求传入结果集的总数。
换句话说，Paginator 对应 simplePaginate 方法，而 LengthAwarePaginator 对应 paginate 方法。
当手动创建分页器实例的时候，应该手动对传递到分页器的结果集进行“切片”，如果你不确定怎么做，查看 PHP 函数array_slice。
# 3、在视图中显示分页结果
当你调用查询构建器或 Eloquent 查询上的 paginate 或 simplePaginate 方法时，你将会获取一个分页器实例。当调用 paginate 方法时，你将获取 Illuminate\Pagination\LengthAwarePaginator，而调用方法 simplePaginate 时，将会获取 Illuminate\Pagination\Paginator 实例。这些对象提供相关方法描述这些结果集，除了这些帮助函数外，分页器实例本身就是迭代器，可以像数组一样对其进行循环调用。
所以，获取到结果后，可以按如下方式使用Blade显示这些结果并渲染页面链接：

```
<div class="container">
    @foreach ($users as $user)
        {{ $user->name }}
    @endforeach
</div>

{!! $users->render() !!}
```

render 方法将会将结果集中的其它页面链接渲染出来。每个链接已经包含了?page 查询字符串变量。记住，render 方法生成的 HTML 兼容Bootstrap CSS 框架。
注意：我们从 Blade 模板调用 render 方法时，确保使用{!! !!}语法以便 HTML 链接不被过滤。
## 3.1 自定义分页器 URI
setPath 方法允许你生成分页链接时自定义分页器使用的 URI，例如，如果你想要分页器生成形如 http://example.com/custom/url?page=N 的链接，应该传递 custom/url 到 setPath 方法：

```
Route::get('users', function () {
    $users = App\User::paginate(15);
    $users->setPath('custom/url');
    //
});
```

## 3.2 添加参数到分页链接
你可以使用 appends 方法添加查询参数到分页链接查询字符串。例如，要添加&sort=votes 到每个分页链接，应该像如下方式调用 appends：

```
{!! $users->appends(['sort' => 'votes'])->render() !!}
```

如果你想要添加”哈希片段”到分页链接，可以使用 fragment 方法。例如，要添加#foo 到每个分页链接的末尾，像这样调用 fragment 方法：

```
{!! $users->fragment('foo')->render() !!}
```

## 3.3 更多帮助方法
你还可以通过如下分页器实例上的方法访问更多分页信息：
- 	$results->count() 
- 	$results->currentPage() 
- 	$results->hasMorePages() 
- 	$results->lastPage() (使用 simplePaginate 时无效) 
- 	$results->nextPageUrl() 
- 	$results->perPage() 
- 	$results->total() (使用 simplePaginate 时无效) 
- 	$results->url($page)
# 4、将结果转化为JSON
Laravel 分页器结果类实现了 Illuminate\Contracts\Support\JsonableInterface契约并实现 toJson 方法，所以将分页结果转化为 JSON 非常简单。
你还可以简单通过从路由或控制器动作返回分页器实例将转其化为 JSON：

```
Route::get('users', function () {
    return App\User::paginate();
});
```

从分页器转化来的 JSON 包含了元信息如 total, current_page,last_page 等等，实际的结果对象数据可以通过该 JSON 数组中的 data 键访问。下面是一个通过从路由返回的分页器实例创建的 JSON 例子：

```
{
   "total": 50,
   "per_page": 15,
   "current_page": 1,
   "last_page": 4,
   "next_page_url": "http://laravel.app?page=2",
   "prev_page_url": null,
   "from": 1,
   "to": 15,
   "data":[
        {
            // Result Object
        },
        {
            // Result Object
        }
   ]
}
```