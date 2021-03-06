# # 新增 create
使用create方法一键新增
```php
User::create([
	'username' => 'admin',
	'password' => bcrypt($password),
	'gender' => '1', //male
	'nickname' => 'Administrator',
]);
```
使用常规方法新增
```php
$user = new User();
$user->username = 'admin';
$user->password = '...';
...
$user->save(); //保存

echo $user->id; //得到新插入的id
```
> create方法也是对常规方法的一个封装

# # 查询 read
## find 根据ID查找一行数据
```php
$user = User::find(1);
echo $user->username; //admin
echo $user['password'];
```
## findMany 根据ID数组查找多行数据
```php
$user = User::findMany([1,2,3,4]);
foreach ($users as $user)
	echo $user->id;
```
## all 获取所有数据
```php
$users = User::all();
foreach ($users as $user)
	echo $user->id;
```
## where 根据条件查找多行数据
where设置条件
```php
$users = User::where('usename', 'LIKE', '%a%')->get();
```

## take offset取几条数据
```php
//take 取10条数句
$users = User::where('nickname', '=', 'nike')->orderBy('created', 'DESC')->take(10)->get();
//从第20行开始
$users = User::where('nickname', '=', 'nike')->orderBy('created', 'DESC')->take(10)->offset(20)->get();
```

## first 只取一条，并得到Model（非数据集）
通过first可以将数据提取出首条并返回
```php
$user = User::where('id', '=', 1)->first();
```

# # 修改 update
常规方法修改
```php
$user = User::find(1);
$user->nickname = 'SuperMan';
$user->save();
```
批量修改
```php
User::where('nickname', 'LIKE', '%man%')->update([
	'nickname' => 'Spider Man',
]);
```
> 这种方法不不会触发updating、updated、saving、saved事件

# # 删除 delete
常规删除
```php
$user = User::find(1);
$user->delete();
```
destory删除
```php
User::destory(1);
```
批量删除
```php
User::where('nickname', 'LIKE', '%man%')->delete();
```
> 这种方法不不会触发deleting、deleted事件

# #　软删除 SoftDeletes
```
//SQL 库中添加此字段
`deleted_at` timestamp NULL DEFAULT NULL,
//Model 中添加
use Illuminate\Database\Eloquent\SoftDeletes;
class User {
	use SoftDeletes;
}
```
此时，执行删除操作（delete destory）操作时，并非真正删除数据，只是在deleted_at字段中做了删除标记
此时通过find get where update等操作来获取数据时，会自动忽略这些软删除数据

## 强制删除
如需要强制删除数据
```
$user = User::find(1);
$user->forceDelete();
```
## 查询结果附带软删除的数据
此时数据集中会附带这些已被软删除的数据，后续操作时，需要自己甄别
```
User::withTrashed()->where('nickname', 'Leo')->get();
```

## 只返回软删除数据
```
User::onlyTrashed()->where('nickname', 'Leo')->get();
```
## 恢复软删除数据
```
$user = User::onlyTrashed()->find(12);
$user->restore();
```