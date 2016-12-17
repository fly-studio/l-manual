# 常规查询
## 条件 Where OrWhere AndWhere
```php
//SELECT * FROM `users` WHERE `id` = 2
User::where('id', '=', 2)->get(); 
//中间等号可以省略
User::where('id', 2)->get(); 
//两个where相当于andWhere
User::where('created_at', '<', '2015-01-01 00:00:00')->where('created_at', '>', '2014-01-01 00:00:00')->get(); 
//or where
User::where('gender', 2)->orWhere('created_at', '<', '2015-01-01 00:00:00')->get(); 
```

## 数组 Where IN
```php
//SELECT * FROM `users` WHERE `id` IN (1,2,3,4)
User::whereIn('id', [1,2,3,4])->get();
//下面的语句是无效的
User::where('id', 'IN' , [1,2,3,4])->get();
```

## 排序 Order By
```php
//SELECT * FROM `users` ORDER BY `id` ASC
User::orderBy('id')->get();
//SELECT * FROM `users` ORDER BY `id` DESC
User::orderBy('id', 'desc')->get();
```

## 递增 递减 increment decrement
```php
//UPDATE `users` SET `field_name` = `field_name` + 1 WHERE `role_id` = 99
User::where('role_id', 99)->increment('field_name');
//UPDATE `users` SET `field_name` = `field_name` - 1 WHERE `role_id` = 99
User::where('role_id', 99)->decrement('field_name');
```
> increment/decrement 是封装的方法,最终执行的是```update(['field_name' => DB::raw('`field_name` + 1')]);```



# 聚合
## 数量 Count
```php
//SELECT COUNT(*) FROM `users`
$count = User::count();
//SELECT COUNT(*) FROM `users` WHERE `some` = `thing`
$count = User::where('some', '=', 'thing')->count();
```
- 返回结果为数字
- 如果你的意图为 ```SELECT COUNT(*) FROM table```，则请 ** 务必不要 ** ->get() 之后再 ->count()，这样会先查询并返回所有数据之后，再进行的return count(数据集Array)。而这些数据集你却并不需要，这样非常浪费程序资源！

## 最大 最小 总数 平均值 Max Min Sum Avg(Average)
```php
//SELECT MAX(`created_at`) FROM `users`
$max = User::max('created_at');
//SELECT MIN(`created_at`) FROM `users` WHERE `some` = `thing`
$min = User::where('some', '=', 'thing')->min('created_at');
//SELECT SUM(`fields_name`) FROM `users`
$total = User::sum('fields_name');
//SELECT average(`fields_name`) FROM `users`
$avg = User::avg('fields_name'); User::average('fields_name');
```
- 返回结果为数字
- 同上，如果你的意图是要得到最大、最小、平均、总数的值，而并是要这些数据集，则请 ** 务必不要 ** ->get() 之后再 ->max()

## 组 Group By
```php
use DB;
//SELECT `gender`,COUNT(`gender`) as `count` FROM `users` GROUP BY `gender`
User::groupBy('gender')->get(['gender', DB::raw('COUNT(`gender`) as `count`')]);
// gender	count
// 1		234
// 2		123
```
> 在需要使用SQL语句的情况下，可使用 DB::raw() 来包裹

# 多表
## 连接 Join
取所有管理员的用户
```php
//SELECT `users`.*, `role_user`.`role_id` FROM `users` INNER JOIN `role_user` ON `role_user`.`user_id` = `users`.`id` WHERE `role_user`.`role_id` = 99
User::join('role_user', 'role_user.user_id', '=', 'users.id', 'INNER')->where('role_user.role_id', 99)->get(['users.*', 'role_user.role_id'])
```
- join 有5个参数：
	- 中间表
	- ON 字段1
	- ON 的条件
	- ON 的字段2
	- JOIN的链接方式，默认为 INNER

## 别名 AS
### 字段别名
```php
//SELECT `id` as `uid`, `username` as `account` FROM `users` WHERE `id` = 1
$user = User::find(1, ['id as uid', 'username as account']);
$user->uid; //1
```
### 表别名
```php
//SELECT `users`.*, `b`.`role_id` FROM `users` INNER JOIN `role_user` AS `b` ON `b`.`user_id` = `users`.`id` WHERE `b`.`role_id` = 99
User::join('role_user as b', 'b.user_id', '=', 'users.id', 'INNER')->where('b.role_id', 99)->get(['users.*', 'b.role_id'])
```

# # 实际执行数据库的函数
## SELECT
get($columns = ['*'])
- get() 表示表达式为SELECT语句
- get() 得到的就是Model的数据集Collection
- 也就是说，查询时，执行到get()时方才去请求数据库，下同

> first() 封装的 reset(...->take(1)->get());方法，reset表示得到第一条数据并返回
> find() 封装的 ->where('id',1)->first();方法

## UPDATE
update($columns)
```
//UPDATE `users` SET `xxx` = 'xxxx' WHERE `xx` = ``xxx`
User::where('xx', 'xxx')->update(['xxx' => 'xxxxx']);
```
- update() 表示表达式为UPDATE语句

## DELETE
delete()
```
//DELETE FROM `users` WHERE `xx` = ``xxx`
User::where('xx', 'xxx')->delete();
```
- delete() 表示表达式为DELETE语句

## ## 实际调用
- Model中的 where、whereIn、orderBy、groupBy、join等方法，都是通过
	`Illuminate\Database\Eloquent\Builder`
	`Illuminate\Database\Query\Builder`
	来实现的。
- Model中的find，create，destroy，update 都是封装的一些常规方法