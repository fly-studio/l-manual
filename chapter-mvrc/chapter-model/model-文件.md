# # 介绍
Model 是Laravel操作数据库的基础类，可以快速的CRUD。也是返回查询结果的类
基本上一个表对应一个Model，关联表（中间表）无Model

# # 表名和Model类名的关系
## 默认表名
默认情况下，系统会猜测类名的复数为表名，比如： 
- **类名 -> 表名**
- **User -> users**
	类名的复数，便是表名
- **UserAvatarAttachment -> user_avatar_attachments**
	多个单词，类名每个单词的首字母大写，表名会自动加上下划线，并在结尾加上复数
- **Media -> media**
	media 没有复数
- **Factory -> factories**
	识别元音，转换-ies
- **Woman -> women**
	特殊单词的复数也可以正常识别，还有man、leaf等
	
Laravel 中有一套完善的单词复数转换程序，在大部分情况下，都能得到单词的正确复数
无意义的单词，会按照常规，在末字母加上-s、-es或-ies
## 自定义表名
加入 $table 参数即可
```php
class Test {
	protected $table = 'table_name';
}
```

# # 主键
## 默认主键
在Model中，主键默认为 id

## 自定义主键
```php
protected $primaryKey = 'factory_id';
```

## 获取主键的值
```php
$user = User::find(1);
$user->getKey();
等同于 $user->id;
```
> 建议获取主键使用 getKey() 
> 或者不自定义主键，所有的表都使用 id，这样方便程序的编写

## 非自增的主键
有些数据库设计时，使用了合并主键，或者字符串主键，则需要关闭本值
```
public $incrementing = false;
```

# # 时间
## 默认时间
Model中默认启用了两个时间字段，如上例SQL语句中
- created_at 数据创建时间
- updated_at 数据最后更新时间

> 这两个字段是Model自动维护的
> 遇到函数或者变量中有 touch，代表该操作是否更新updated_at

## 关闭默认时间
```
public $timestamps = false;
```

## 增加其他时间字段
如上例 users 表中 lastlogin_at 字段
加入 $dates ，则Model在读取、写入时，会自动将时间转为DateTime型，便于操作
```
protected $dates = ['lastlogin_at'];
```
> ```$timestamps = true;```时，系统会自动为 $dates 加上 created_at、updated_at

# # 转换字段
model可在字段写入库或输出时，转换成你想要的类型
```php
public $casts = [
	'data' => 'array',
	'married' => 'boolean',
];

$user = User::find(1);
//输出
echo $user->married; //true
print_r($user->data); //array
//写入
$user->married = false;
$user->data = ['xx' => 'xxxx', 'xxx' => 'xxxxx'];
$user->save();
```
如上例，被$casts设置的字段，会在输出时转换为相应的类型
**输入时，赋值对应类型的值即可，**在数据库中会自动转换并存储

## 可设置的类型
- **int、integer**
	将数据转化为**整形**并输出
- **real、float、double**
	将数据转化为**浮点数**并输出
	数据库字段一般为real,float,double,decimal,numeric类型
- **string**
	将数据转化为**字符串**并输出
	数据库字段一般为varchar、text类型
- **bool、boolean**
	将数据转化为**True/False**并输出
	支持 1、0、true、false 这四个值的输入
	数据库字段一般设置为tinyint型
- **object**
	数据在输出时会转化为对应Object类型
	数据库字段一般设置为text、longtext型
- **array、json**
	数据在输出时会转化为**数组**类型
	数据库字段一般设置为text、longtext型
- **collection**
	数据在输出时会转化为**Collection**类型
	Collection类型是Laravel对数据集的一次封装
	数据库字段一般设置为text、longtext型
- **date、datetime**
	数据在输出时会转化为**DateTime**类型
	数据库字段一般设置为timestamp型

> 整形、浮点数、字符串，无需特别指定，
> datetime 请使用上述的 $dates 来指定
> boolean、array为常用指定类型，其它类型非必要时请勿使用，这样会增加系统负担

# # 字段白名单、黑名单
## 字段黑名单(写入)
如果你不希望数据库中插入id（因为id是自增量）
```php
protected $guarded = ['id'];
```
这样，creat、update、save等方法都会过滤这个字段的写入
```php
User::create([
	'id' => 25, //会自动过滤这项
	'username' => 'dick',
])
```
> Model默认配置```$guarded = ['*']``` 表示拒绝所有字段的写入

## 关闭黑名单
Model默认启用黑名单
```php
protected static $unguarded = true; //关闭

或

User::$unguarded = true;
```

## 字段白名单(写入)
```php
protected $fillable = ['username', 'password'];
```
白名单一般配合```$guarded = ['*']```来使用。在全黑名单的情况下，设置某些字段为白名单

# # 数据隐藏或显示
## 数据隐藏(读取时)
如果希望在输出数据时，不显示password、token等字段
```php
protected $hidden = ['password','token'];
```
这样在** toArray()、toJson() **时，会自动去除这些数据

<div class="alert alert-warning">注意：$user->password、$user['password'] 可正常使用，仅仅在toArray()、toJson()时过滤这些数据</div>

## 数据显示
当$hidden = ['*']时，可启用
```php
protected $visible = ['username','gender'];
```


# # 附加字段/数据
Model 可以自创一些字段来增加程序的可用性。
注意：这些字段是不会插入数据库的
```php
class Role extends Model {
	protected $appends = ['is_admin'];
	
	public function getIsAdminAttribute()
	{
		return $this->getKey() == 99;
	}
}

$role = Role::find(99);
echo $role->is_admin; //true
```

> $appends 也可以结合 $hidden/$visible 用