# 查询构建器

# 1、简介
数据库查询构建器提供了一个方便的、平滑的接口来创建和运行数据库查询。查询构建器可以用于执行应用中大部分数据库操作，并且能够在支持的所有数据库系统上工作。
注意：Laravel 查询构建器使用 PDO 参数绑定来避免 SQL 注入攻击，不再需要清除传递到绑定的字符串。
# 2、获取结果集
## 2.1 从一张表中取出所有行
在查询之前，使用 DB 门面的 table 方法，table 方法为给定表返回一个查询构建器，允许你在查询上链接更多约束条件并最终返回查询结果。在本例中，我们使用 get 方法获取表中所有记录：

```
<?php

namespace App\Http\Controllers;

use DB;
use App\Http\Controllers\Controller;

class UserController extends Controller{
    /**
     * 显示用户列表
     *
     * @return Response
     */
    public function index()
    {
        $users = DB::table('users')->get();

        return view('user.index', ['users' => $users]);
    }
}
```

和原生查询一样，get 方法返回结果集的数据组，其中每一个结果都是 PHP 对象的 StdClass 实例。你可以像访问对象的属性一样访问列的值：

```
foreach ($users as $user) {
    echo $user->name;
}
```

## 2.2 从一张表中获取一行/一列
如果你只是想要从数据表中获取一行数据，可以使用 first 方法，该方法将会返回单个 StdClass 对象：

```
$user = DB::table('users')->where('name', 'John')->first();
echo $user->name;
```

## 2.3 从一张表中获取组块结果集
如果你需要处理成千上百条数据库记录，可以考虑使用 chunk 方法，该方法一次获取结果集的一小块，然后填充每一小块数据到要处理的闭包，该方法在编写处理大量数据库记录的Artisan 命令的时候非常有用。比如，我们可以将处理全部 users 表数据处理成一次处理 100 记录的小组块：

```
DB::table('users')->chunk(100, function($users) {
    foreach ($users as $user) {
        //
    }
});
```

你可以通过从闭包函数中返回 false 来中止组块的运行：

```
DB::table('users')->chunk(100, function($users) {
    // 处理结果集...
    return false;
});
```

## 2.4 获取数据列值列表
如果想要获取包含单个列值的数组，可以使用 lists 方法，在本例中，我们获取所有 title 的数组：

```
$titles = DB::table('roles')->lists('title');

foreach ($titles as $title) {
    echo $title;
}
```

在还可以在返回数组中指定更多的自定义键：

```
$roles = DB::table('roles')->lists('title', 'name');

foreach ($roles as $name => $title) {
    echo $title;
}
```

## 2.5 聚合函数
队列构建器还提供了很多聚合方法，比如 count, max, min, avg, 和 sum，你可以在构造查询之后调用这些方法：

```
$users = DB::table('users')->count();$price = DB::table('orders')->max('price');
```

当然，你可以联合其它查询字句和聚合函数来构建查询：

```
$price = DB::table('orders')
                ->where('finalized', 1)
                ->avg('price');
```

# 3、查询（Select）
## 3.1 指定查询子句
当然，我们并不总是想要获取数据表的所有列，使用 select 方法，你可以为查询指定自定义的 select 子句：

```
$users = DB::table('users')->select('name', 'email as user_email')->get();
```

distinct 方法允许你强制查询返回不重复的结果集：

```
$users = DB::table('users')->distinct()->get();
```

如果你已经有了一个查询构建器实例并且希望添加一个查询列到已存在的 select 子句，可以使用 addSelect 方法：

```
$query = DB::table('users')->select('name');
$users = $query->addSelect('age')->get();
```

## 3.2 原生表达式
有时候你希望在查询中使用原生表达式，这些表达式将会以字符串的形式注入到查询中，所以要格外小心避免被 SQL 注入。想要创建一个原生表达式，可以使用 DB::raw 方法：

```
$users = DB::table('users')
                     ->select(DB::raw('count(*) as user_count, status'))
                     ->where('status', '<>', 1)
                     ->groupBy('status')
                     ->get();
```

# 4、连接（Join）
## 4.1 内连接（等值连接）
查询构建器还可以用于编写基本的 SQL“内连接”，你可以使用查询构建器实例上的join方法，传递给 join 方法的第一次参数是你需要连接到的表名，剩余的其它参数则是为连接指定的列约束，当然，正如你所看到的，你可以在单个查询中连接多张表：

```
$users = DB::table('users')
            ->join('contacts', 'users.id', '=', 'contacts.user_id')
            ->join('orders', 'users.id', '=', 'orders.user_id')
            ->select('users.*', 'contacts.phone', 'orders.price')
            ->get();
```

## 4.2 左连接
如果你是想要执行“左连接”而不是“内连接”，可以使用 leftJoin 方法。该方法和 join 方法的使用方法一样：

```
$users = DB::table('users')
            ->leftJoin('posts', 'users.id', '=', 'posts.user_id')
            ->get();
```

## 4.3 高级连接语句
你还可以指定更多的高级连接子句，传递一个闭包到 join 方法作为该方法的第 2 个参数，该闭包将会返回允许你指定 join 子句约束的 JoinClause 对象：

```
DB::table('users')
        ->join('contacts', function ($join) {
            $join->on('users.id', '=', 'contacts.user_id')->orOn(...);
        })
        ->get();
```

如果你想要在连接中使用“where”风格的子句，可以在查询中使用 where 和 orWhere 方法。这些方法将会将列和值进行比较而不是列和列进行比较：

```
DB::table('users')
        ->join('contacts', function ($join) {
            $join->on('users.id', '=', 'contacts.user_id')
                 ->where('contacts.user_id', '>', 5);
        })
        ->get();
```

# 5、联合（Union）
查询构建器还提供了一条“联合”两个查询的快捷方式，比如，你要创建一个独立的查询，然后使用 union 方法将其和第二个查询进行联合：

```
$first = DB::table('users')
            ->whereNull('first_name');

$users = DB::table('users')
            ->whereNull('last_name')
            ->union($first)
            ->get();
```

unionAll 方法也是有效的，并且和 union 有同样的使用方法。
# 6、Where 子句
## 6.1 简单 where 子句
使用查询构建器上的 where 方法可以添加 where 子句到查询中，调用 where 最基本的方法需要三个参数，第一个参数是列名，第二个参数是一个数据库系统支持的任意操作符，第三个参数是该列要比较的值。
例如，下面是一个验证“votes”列的值是否等于 100 的查询：

```
$users = DB::table('users')->where('votes', '=', 100)->get();
```

为了方便，如果你只是简单比较列值和给定数值是否相等，可以将数值直接作为 where 方法的第二个参数：

```
$users = DB::table('users')->where('votes', 100)->get();
```

当然，你可以使用其它操作符来编写 where 子句：

```
$users = DB::table('users')
                ->where('votes', '>=', 100)
                ->get();

$users = DB::table('users')
                ->where('votes', '<>', 100)
                ->get();

$users = DB::table('users')
                ->where('name', 'like', 'T%')
                ->get();
```

6.2 or
你可以通过方法链将多个 where 约束链接到一起，也可以添加 or 子句到查询，orWhere 方法和 where 方法接收参数一样：

```
$users = DB::table('users')
                    ->where('votes', '>', 100)
                    ->orWhere('name', 'John')
                    ->get();
```

## 6.3 更多 Where 子句
### 6.3.1 whereBetween
whereBetween 方法验证列值是否在给定值之间：

```
$users = DB::table('users')
                    ->whereBetween('votes', [1, 100])->get();
```

### 6.3.2 whereNotBetween
whereNotBetween 方法验证列值不在给定值之间：

```
$users = DB::table('users')
                    ->whereNotBetween('votes', [1, 100])
                    ->get();
```

### 6.3.3 whereIn/whereNotIn
whereIn 方法验证给定列的值是否在给定数组中：

```
$users = DB::table('users')
                    ->whereIn('id', [1, 2, 3])
                    ->get();
```

whereNotIn 方法验证给定列的值不在给定数组中：

```
$users = DB::table('users')
                    ->whereNotIn('id', [1, 2, 3])
                    ->get();
```

6.3.4 whereNull/whereNotNull
whereNull 方法验证给定列的值为 NULL：

```
$users = DB::table('users')
                    ->whereNull('updated_at')
                    ->get();
```

whereNotNull 方法验证给定列的值不是 NULL：

```
$users = DB::table('users')
                    ->whereNotNull('updated_at')
                    ->get();
```

## 6.4 高级 Where 子句
### 6.4.1 参数分组
有时候你需要创建更加高级的 where 子句比如”where exists“或者嵌套的参数分组。Laravel 查询构建器也可以处理这些。作为开始，让我们看一个在括号中进行分组约束的例子：

```
DB::table('users')
            ->where('name', '=', 'John')
            ->orWhere(function ($query) {
                $query->where('votes', '>', 100)
                      ->where('title', '<>', 'Admin');
            })
            ->get();
```

正如你所看到的，传递闭包到 orWhere 方法构造查询构建器来开始一个约束分组，，该闭包将会获取一个用于设置括号中包含的约束的查询构建器实例。上述语句等价于下面的 SQL：

```
select * from users where name = 'John' or (votes > 100 and title <> 'Admin')
```

### 6.4.2 exists 语句
whereExists 方法允许你编写 where existSQL 子句，whereExists 方法接收一个闭包参数，该闭包获取一个查询构建器实例从而允许你定义放置在”exists”子句中的查询：

```
DB::table('users')
            ->whereExists(function ($query) {
                $query->select(DB::raw(1))
                      ->from('orders')
                      ->whereRaw('orders.user_id = users.id');
            })
            ->get();
```

上述查询等价于下面的 SQL 语句：

```
select * from users
where exists (
    select 1 from orders where orders.user_id = users.id
)
```

# 7、排序、分组、限定
## 7.1 orderBy
orderBy 方法允许你通过给定列对结果集进行排序，orderBy 的第一个参数应该是你希望排序的列，第二个参数控制着排序的方向——asc 或 desc：

```
$users = DB::table('users')
                ->orderBy('name', 'desc')
                ->get();
```

## 7.2 groupBy / having / havingRaw
groupBy 和 having 方法用于对结果集进行分组，having 方法和 where 方法的用法类似：

```
$users = DB::table('users')
                ->groupBy('account_id')
                ->having('account_id', '>', 100)
                ->get();
```

havingRaw 方法可以用于设置原生字符串作为 having 子句的值，例如，我们要找到所有售价大于$2500 的部分：

```
$users = DB::table('orders')
                ->select('department', DB::raw('SUM(price) as total_sales'))
                ->groupBy('department')
                ->havingRaw('SUM(price) > 2500')
                ->get();
```

## 7.3 skip / take
想要限定查询返回的结果集的数目，或者在查询中跳过给定数目的结果，可以使用 skip 和 take 方法：

```
$users = DB::table('users')->skip(10)->take(5)->get();
```

# 8、插入（Insert）
查询构建器还提供了 insert 方法来插入记录到数据表。insert 方法接收数组形式的列名和值进行插入操作：

```
DB::table('users')->insert(
    ['email' => 'john@example.com', 'votes' => 0]);
```

你甚至可以一次性通过传入多个数组来插入多条记录，每个数组代表要插入数据表的记录：

```
DB::table('users')->insert([
    ['email' => 'taylor@example.com', 'votes' => 0],
    ['email' => 'dayle@example.com', 'votes' => 0]
]);
```

## 8.1 自增 ID
如果数据表有自增 ID，使用 insertGetId 方法来插入记录将会返回 ID 值：

```
$id = DB::table('users')->insertGetId(
    ['email' => 'john@example.com', 'votes' => 0]
);
```

注意：当使用 PostgresSQL 时 insertGetId 方法默认自增列被命名为 id，如果你想要从其他”序列“获取 ID，可以将序列名作为第二个参数传递到 insertGetId 方法。
# 9、更新（Update）
当然，除了插入记录到数据库，查询构建器还可以通过使用 update 方法更新已有记录。update 方法和 insert 方法一样，接收列和值的键值对数组包含要更新的列，你可以通过 where 子句来对 update 查询进行约束：

```
DB::table('users')
            ->where('id', 1)
            ->update(['votes' => 1]);
```

## 9.1 增加/减少
查询构建器还提供了方便增减给定列名数值的方法。相较于编写 update 语句，这是一条捷径，提供了更好的体验和测试接口。
这两个方法都至少接收一个参数：需要修改的列。第二个参数是可选的，用于控制列值增加/减少的数目。

```
DB::table('users')->increment('votes');
DB::table('users')->increment('votes', 5);
DB::table('users')->decrement('votes');
DB::table('users')->decrement('votes', 5);
```

在操作过程中你还可以指定额外的列进行更新：

```
DB::table('users')->increment('votes', 1, ['name' => 'John']);
```

# 10、删除（Delete）
当然，查询构建器还可以通过 delete 方法从表中删除记录：

```
DB::table('users')->delete();
```

在调用 delete 方法之前可以通过添加 where 子句对 delete 语句进行约束：

```
DB::table('users')->where('votes', '<', 100)->delete();
```

如果你希望清除整张表，也就是删除所有列并将自增 ID 置为 0，可以使用 truncate 方法：

```
DB::table('users')->truncate();
```

## 11、悲观锁
查询构建器还包含一些方法帮助你在 select 语句中实现”悲观锁“。可以在查询中使用 sharedLock 方法从而在运行语句时带一把”共享锁“。共享锁可以避免被选择的行被修改直到事务提交：

```
DB::table('users')->where('votes', '>', 100)->sharedLock()->get();
```

此外你还可以使用 lockForUpdate 方法。”for update“锁避免选择行被其它共享锁修改或删除：

```
DB::table('users')->where('votes', '>', 100)->lockForUpdate()->get();
```