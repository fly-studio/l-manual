# # 创建控制器
## ## artisan 工具创建
```
php artisan make:controller UserController
```
采用artisan工具创建的控制器是RESTful结构

## ## 工具箱创建

工具箱创建路由是向导可视化的，一键创建控制器、Model；如果是后台页面，则还会创建视图、验证规则等

# # 控制器基本属性

```php
<?php
namespace App\Http\Controllers;

use Illuminate\Http\Request;
use App\Http\Requests;
use App\Http\Controllers\Controller;


use App\User;
class UserController extends Controller {
	
	public function index()
	{
		//读取当前用户的资料
		$this->_profile = User::find($this->user['id']);
		return $this->view('user/profile');
	}
	
	public function update(Request $request, $id)
	{
		//检查用户是否存在
		$user = User::find($id);
		if (empty($user))
			return $this->failure_noexistst();
		//验证表单字段
		$keys = 'nickname,realname,gender,email,phone,avatar_aid';
		$data = $this->autoValidate($request, 'member.store', $keys);
		//更新数据
		$user->update($data);
		//返回成功信息
		return $this->success('user.success_update', url('user'));
	}
	
	public function download()
	{
		$filename = '/www/website/1.zip';
		return response()->download($filename);
	}
	
	public function preview()
	{
		$filename = '/www/website/1.jpg';
		return response()->preview($filename);
	}
}
```

## ## 参数来源
:fa-sign-in: 关于类注入、参数注入的规则请查阅 MVRC - 路由器
```
public function index(Request $request) {
	$data = $request->all();
}
```

## ## 全局变量
> 全局参数属于L+ 添加的内容

这些变量可以在控制器里使用或者修改
也可以在smarty模板里，通过<{$_site.title}>的方式读取（下划线）

- **$this->site**
网站的标题、副标题、页码等
	```php
	[
		'title' => '网站的标题',
		'titles' => [ // 通过 $this->subtitle() 设置子标题
			...
		],
		'pagesize' => [
			'user' => 20, //用户列表20条每页
		]
	]
	```
- **$this->user**
当前用户的资料，未登录返回空数组
```php
[
	'id' => 12,
	'username' => 'xiao',
	'nickname' => 'QQ',
	...
]
```
- ** $this->roles**
所有组的权限，结构为
```
[
	'admin' => [
		'allow_edit',
		'allow_delete',
		...
	],
	'viewer' => [
		...
	],
];
```
- **$this->fields**
读取Field.php的字段集，方便在视图中用select、radio、checkbox展示这些信息，结构为
```php
[
	'gender' => [
		'male' => [
			'id' => 1,
			'name' => 'male',
			'title' => '男',
			'extra' => [],
		],
		'female' => [
			'id' => 2,
			'name' => 'female',
			'title' => '女',
			'extra' => [],
		],
	],
	'other types' => [
		...
	],
]
```

## ## 数据注入视图

### 注入变量
```php
$this->变量名 = 数据;
```
采用这种方式可以将数据注入到视图中，但是为了避免和控制器变量冲突，以及命名规范，建议使用下划线开头<code>$this->_data = [''];</code>

> _site，_user，_roles，_fields 已被全局变量占用，请勿使用

### 构建视图

```php
return $this->view('文件相对位置');
```
注意：原生的<code>return view('');</code>的方式也可以使用，但是需要按照下例使用，而无法通过<code>$this->变量</code>的方式

```php
return view('filename')->with('prarm', $data)->with('param', $data);
```

### 视图
本手册以smarty为模板引擎，blade引擎的使用方法请参见官方手册

```html
<{$_user.nickname}>

<{foreach $_fields.gender as $item}> <{$item.title}> <{/foreach}>

<{if !empty($_data->updated_at)}> ... <{/if}>
```

:fa-sign-in: 关于视图的详细介绍请参见 MVRC - 视图

# # 组件
## ## 用户验证组件

参见 /project_name/app/Http/Controllers/AuthController.php

```php
use Auth;
```
:fa-sign-in: 请阅读 功能模块 - 用户验证模块 以了解更多

## ## 表单验证组件

参见 /project_name/app/Http/Controllers/Admin/Controller.php

```php
$keys = 'nickname,realname,gender,email,phone,avatar_aid';
$data = $this->autoValidate($request, 'member.store', $keys);
```
:fa-sign-in: 请阅读 功能模块 - 用户验证模块 以了解更多

## ## 提示与输出
效果参见 http://.../auth/login
```
return $this->success('auth.success_login', url('user/index'));
```
输出登录成功信息，并跳转到user/index

:fa-sign-in: 请阅读 功能模块 - 表单验证模块 以了解更多


