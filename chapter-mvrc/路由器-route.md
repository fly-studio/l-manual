# # 文件位置
总路由表位于 
```
/project_name/app/Http/routes.php
```
下文的路由均添加到本文件

# # 路由规则

## ## 基本路由
- Route::http_method('route_name', 'Controller@method');
> @param string http_method 请求方法：get/post/delete/put/patch/any
> @param string route_name 路由名
> @param string Controller 类名
> @param string method 方法名

- Route::match(['get', 'post'], 'route_name', 'Controller@method_name');
> @param ['get', 'post'] 请求方法的数组 
> @param string route_name 路由名
> @param string Controller 类名
> @param string method 方法名

▲ 路由器名 **route_name**
> - / 代表路径
- {param} 必填参数，对应方法中的```function method($param)```
- {param?} 可选参数，表示本参数可以不输入，对应方法中的```function method($param = 'default value')```

▲ 控制器 ** Controller@method **
> - Controller为类名，比如UserController
- method 类中方法，比如 show，对应```function show()```

★ 首页

	Route::get('/', function ()    {});
	Route::get('/', 'HomeController@index');

★ 比如 *project_name/app/Http/Controller/UserController.php*
```php
class UserController extends Controller {
	
	//GET 示例
	//Route::get('user/{id}', 'UserController@show');
	//GET http://.../user/123
	public function show($id)
	{
		return 'user '.$id;
	}
	
	//GET 示例
	//Route::get('user/{id}/edit', 'UserController@edit');
	//GET http://.../user/123/edit
	public function edit($id)
	{
		return 
		'<form action="user/'.$id.'/update" method="POST">
			<inupt type="text" name="nickname" value="" />
		</form>';
	}
	
	//POST 示例
	//Route::post('user/{id}/update', 'UserController@update');
	//POST http://.../user/123/edit
	public function update(Request $request, $id)
	{
		return $id . 'updated: ' . $request->input('nickname');
	}
	
	//可选参数示例
	//Route::get('user/getdata/{of?}', 'UserController@data')
	//GET http://.../user/getdata
	//GET http://.../user/getdata/text
	public function data($of = 'json')
	{
		$data = ['xx' => 'xxx'];
		return $of == 'json' ? json_encode($data) : var_export($data, true);
	}
}
```


## ## RESETful路由（资源路由）
- Route::resource('user', 'UserController');
- Route::resources([
	'user' => 'UserController',
	'comment' => 'CommentController',
]);

| 请求方法 | 路径 | 对应函数 | 路由名字 | 解释 |
| ------------ | ------------ | ------------ | ------------ | ------------ |
| GET | /user | index() | user.index | 首页 |
| GET | /user/{id}  | show($id) | user.show | ID对应的用户资料 |
| GET | /user/create  | create()  | user.create | 新建用户页面 |
| POST | /user  | store()  | user.store | 保存新建的资料 |
| GET | /user/{id}/edit | edit($id) | user.edit | 编辑用户资料 |
| PUT/PATCH | /user/{id} | update($id) | user.update | 保存编辑的资料 |
| DELETE | /user/{id} | destory($id) | user.destory | 删除用户 |

<div class="alert alert-warning">如果要和标准路由配合使用，请将标准路由放置到本路由前</div>

#### 模拟请求
由于&lt;form&gt;表单不支持DELETE/PUT/PATCH等提交方式
可以通过这种方式来模拟：
```html
<input type="hidden" name="_method" value="PUT" >
```
或者调用method_field函数（smarty）
```html
<{method_field('PUT') nofilter}>
```

#### 只开放制定的方法
```php
//只包含 index show
Route::resource('user', 'UserController',
                ['only' => ['index', 'show']]);

//排除 create store update destroy
Route::resource('photo', 'PhotoController',
                ['except' => ['create', 'store', 'update', 'destroy']]);
```

#### 设置函数别名
```php
// 创建函数别名为 build
Route::resource('user', 'UserController',
                ['names' => ['create' => 'user.build', 'destory' => 'user.delete']]);

...

public function build(){

}

public function delete() {

}
```

## ## 控制器路由（懒人路由）
- Route::controller('user', 'UserController');

**{ctrl}/{action}/{param1}/{pram2}/...**

| 请求方式 | 路径(举例) | 函数  |
| ------------ | ------------ |
| GET | /user/show | getShow |
| POST | /user/store | postStore |
| GET | /user/show-profile | getShowProfile |
| DELETE | /user/data | deleteData  |
|   | /user/xx  | anyXx  |
> 通过HTTP请求的方式去查找对应函数
> 当没有找到请求方式对应的函数时，会尝试执行anyMethod
> 故上例也可以为anyShow、anyStore、anyData

> \- _ 会被自动转换为驼峰式的函数名 show-profile => getShowProfile

<div class="alert alert-warning">如果要和标准路由、RESTful路由配合使用，请将这些路由放置到本路由前</div>


★ 比如 *project_name/app/Http/Controller/UserController.php*
```php
class UserController extends Controller {
	
	//GET http://.../user/show/123
	public function getShow($id)
	{

	}
	
	//POST 示例
	//POST http://.../user/update/123/243/Kitty
	public function postUpdate(Request $request, $id, $cid, $name)
	{

	}
	
	//GET http://.../user/xx
	//POST http://.../user/xx
	//DELETE http://.../user/xx
	public function anyXx()
	{

	}
}
```
#### 设置函数别名
GET http://.../show 指向到```show```
```php
Route::controller('users', 'UserController', [
    'getShow' => 'user.show',
]);

...

public function show() {}
```



## ## 默认路由
> (*) 默认路由属于L+新增的部分
> 默认路由器，表示当所以路由匹配失败时，调用本路由。

实现语句
```
$this->setUndefinedRoutes();
```

<div class="alert alert-warning"> 其他路由必须放在此条路由前</div>

### 1. 根目录
**{ctrl?}/{action?}**

> @param string [home] 控制器类名，默认为 ```HomeController```
> @param string [index] 类中函数名，默认为 ```function index()```

#### 首页
★ 比如 *project_name/app/Http/Controller/HomeController.php*
```php
class HomeContoller extends Controller {
	//http://.../
	//或 http://.../home
	//或 http://.../home/index
	public function index()
	{

	}

	//http://.../home/foo
	public function foo()
	{

	}
}
```
#### 其他页
★ 比如 *base/app/Http/Controller/UserController.php*
```php
class UserContoller extends Controller {

	//http://.../user/bar?id=12
	public function bar($id, $page = 1)
	{

	}
}
```
#### 复杂的类名或函数名
★ 比如 *base/app/Http/Controller/UserCommentController.php*
```php
//- _ 会被自动转换为驼峰式的类名
class UserCommentController extends Controller {

	//http://.../user-comment/foo-bar
	//\- _ 会被自动转换为驼峰式的函数名
	public function fooBar(Request $request)
	{

	}
}
```
### 2. admin目录
**admin/{ctrl?}/{action?}**
> @param string [home] 控制器类名，默认为 ```Admin\HomeController```
> @param string [index] 类中函数名，默认为 ```function index()```

★ 比如 *base/app/Http/Controller/Admin/HomeController.php*
```php
class HomeContoller extends Controller {

	//http://www.domain.com/admin
	//或 http://www.domain.com/admin/home
	//或 http://www.domain.com/admin/home/index
	public function index()
	{
		return 'admin home page';
	}

	//http://www.domain.com/admin/home/bar
	public function bar()
	{
		return 'admin bar';
	}
}
```
### 3. 其它目录 
如果想设置其他目录的默认路由，请参照routes.php中Admin的配置

```php
// Admin目录下的路由表
Route::group(['namespace' => 'Admin','prefix' => 'admin'], function($router) {
	
	//admin目录下的其它路由需放置在本条前
	$this->setUndefinedRoutes();
});
```

### 4. 与其他路由的冲突
- 默认路由一定要放在路由表的结尾
- :exclamation:如果该Controller.php还设置了RESTful方式的路由
user/some 会匹配到 user/{id} 而不是some函数
- 默认路由与控制器路由是冲突关系
因为「控制器路由」涵盖了整个网址的参数段（查看：控制器路由的参数）
- 方法名不能使用list、sort等名称，因为list、sort是php的函数名:sweat_smile:

## ## 路由匹配顺序
在路由表中，设置路由，应该遵循如下的顺序
```
//http://.../user/get-something/1
1: Route::any('user/get-something/{id}', 'UserController@getSomething');
//http://.../user/1
2: Route::resource('user', 'UserController');
//http://.../user/id/1/name/Foo
3: Route::controller('user', 'UserController');
```
系统会由上至下的匹配路由，如下图，路由的精准度 依次降低

```flow
op1=>operation: 基本路由
op2=>operation: RESTful路由
op3=>operation: 默认路由 <-> 控制器路由

op1->op2->op3
```
- 所以设置路由的顺序一定是 最精确放最前 最模糊放最后
	

- 使用了控制器路由，该路由名下的「默认路由」会作废，比如
```
// http://.../member/create
Route::controller('member', 'MemberController');
指向的是 function anyCreate(); 
而不是 function create();
```

# # 路由组
- Route::group(['namespace' => 'Admin','prefix' => 'admin', 'middleware' => 'auth'], 		function($router) {

	});
> @param namespace 命名空间前缀
> @param prefix 路径前缀
> @param middleware 需执行的中间件

路由组，表示在该路由组下添加的路由，遵循该组的 namespace middleware prefix

★ 如Admin目录
```php
Route::group(['namespace' => 'Admin','prefix' => 'admin'], function($router) {
	
	//prefix 对应 http://.../admin/
	//namespace 对应 App\\Http\\Controllers\\Admin\\HomeController
	Route::get('/', 'HomeController@index')
	
});
```
★ 如Admin\Wechat目录
```php
Route::group(['namespace' => 'Admin','prefix' => 'admin'], function($router) {
	Route::group(['namespace' => 'Wechat','prefix' => 'wechat'], function($router) {
		//prefix 对应 http://.../admin/wechat/user
		//namespace 对应 App\\Http\\Controllers\\Admin\\Wechat\\UserController
		Route::resource('User', 'UserController')
	});
});
```

> 关于中间件，请参见 MVRC - 中间件

# # 设置路由别名
设置别名，可以方便路由调用、组合网址
### 表示将本路由别名为 show_user
```php
//不带参数的别名 show_me
Route::get('user/profile', [
'as' => 'show_me', function(){

}
]);
//带参数的别名 show_user
Route::get('user/profile/{id}', [
'as' => 'show_user',
'uses' => 'UserController@show',
]);
//组别名 admin::dashboard
Route::group(['as' => 'admin::'], function ($router) {
    Route::get('dashboard', ['as' => 'dashboard', function () {

    }]);
});
```
### 这样可以通过这些方式使用别名
```php
//无参数
$url = route('show_me');
redirect()->route('show_me');
//传参
$url = route('show_user', ['id' => 12]);
//组别名
$url = route('admin::dashboard');
```

# # 参数注入方式(IOC)
参数注入：表示给路由所指向的函数的参数赋值
如下两例
```php
use Illuminate\Http\Request;
Route::any('user/edit/{id}', function(Request $request, $id){

});
```

```php
Route::any('user/edit/{id}', 'UserController@edit');
Route::any('user/{uid}/comment/{cid}', 'UserController@show');
...

use Illuminate\Http\Request;
class UserController extends Controller {
	
	public function edit(Request $request, $id)
	{
	
	}
	
	public function show(Request $request, $userID, $commentID)
	{
	
	}
}
```

## 1. 标准参数注入方式
{id} {uid} {cid} 是路由的参数
$id $userID $commentID 是函数的参数

&clubs; {id} => $id
&clubs; {uid} => $userID
&clubs; {cid} => $commentID

- 此注入方式的「函数参数」顺序，必须按照路由中的参数顺序
- 可以看到{cid} 和 commentID名称并不一致
- 因为函数的参数名不是重要的，Laravel是按照顺序赋值的

那*Request $request*是什么呢？见下文

## 2. 类的注入(DI)
$request 并不存在于「路由参数」中，它的类型是```Illuminate\Http\Request```
在Laravel中，任何可以通过```app('class_name')```>访问的类都可以以这种方式注入到Controller函数的参数中，这个操作是自动的
所以```Request $request```会自动转换为```app('Illuminate\Http\Request')```，然后赋值给$request

关于类的注入，有如下特点：
- DI参数可以放在函数的任何位置，但是通常情况下放置首位
- 一个函数中，只能有一个同类型的DI参数
多个同类型的DI参数，只会注入到第一个DI参数中，其它同类型的DI参数，会被注入错误的值导致报错
- 可以拥有多个不同类型的DI参数
- DI参数不会影响标准参数的顺序，系统会自动跳过DI

> 关于app函数，请查看 **Helper - Laravel运作方式**

## 3. 默认路由的参数
默认路由，因为在路由名已固定为```{ctrl?}/{action?}```，所以函数中的参数均来自于```Request::input($param)```，并且支持DI

<div class="alert alert-warning">除非你能确定你的请求网址会被默认路由捕获，如果被其他路由捕获，以下参数的设置将会导致页面报错</div>
如：http://.../home/index?id=2&order[create_at]=desc&filters[title]=xxx
```php
public function index(Request $request, $id, $filters, $order = array('id' => 'desc'))
{

}
```
- 参数顺序没有要求
- 支持DI参数
- 支持默认值
- 赋值顺序：NULL >> default value >> Request::input($param)
	- 当没有 Request::input($param)，参数被赋值为默认值
	- 当 Request::input($param) 和 默认值 都没有，参数被赋值为NULL
- Request::input中即使没有该参数，不会报错

## 4. 控制器路由的参数
比如搜索
GET http://.../user/filter/kitty/male/1
```php
public function getFilter(Request $request, $keyword, $gender, $has_avatar = 0) 
{
	echo $keyword; //kitty
	echo $gendar; // male
	echo $hash_avatar; // 1
}
```
- 按照函数参数的顺序依次赋值
- 支持DI参数
- 支持默认值
- 路由中如果没有该参数，并且函数参数中没有默认值，会报错


## 5. 注入运行实例到参数

```php
//绑定到{user}参数
Route::bind('user', function($value, $route){
	return \App\User::find($value);
});

//http://.../profile/12
Route::get('profile/{user}', function(\App\User $user) {
	//此处的$user为用户ID为12的\App\User的Model
	echo $user->username;
})->where('user', '[0-9]+');
```

同理，只要修改bind函数体的内容，可以绑定更多的实例到参数

# # 参数的筛选
- ->where('param_name', 'regexp');
- ->where([
'param_name1' => 'regexp',
'param_name2' => 'regexp',
]);

&diams; 仅当id为数字时，匹配本路由：http://&hellip;/user/123
```php
Route::get('user/{id}', 'UserController@show')->where('id', '\d+');
```
&diams; 仅当id为数字、name为英文时，匹配本路由：http://&hellip;/user/123/abc
```php
Route::get('user/{id}/{name}', 'UserController@show')->where(['id' => '\\d+', 'name' => '[a-zA-Z]+']);
```
也可以设置全局的筛选
&diams; 仅当所有路由中{id}参数为数字时方可匹配该路由
```php
# RouteServiceProvider.php

public function boot(Router $router)
{
    $router->pattern('id', '[0-9]+');

    parent::boot($router);
}
```

