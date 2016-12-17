# 中间件
中间件，在匹配路由之后，调取Controller之前执行，所以常用于用户验证、权限验证等

中间件按照功能分为全局执行、路由执行

# # 启动文件
```
/project_name/app/Http/Kernel.php
```
# # 全局中间件
表示任何HTTP请求，都会执行这些中间件
```php
protected $middleware = [
        \Illuminate\Foundation\Http\Middleware\CheckForMaintenanceMode::class,
];
```
系统会执行：
- **CheckForMaintenanceMode**

## ## Laravel 5.3
在`Laravel 5.3`中，将路由分为了：`api` `web`
因此中间件也分成了两部分
```
protected $middlewareGroups = [
    'web' => [
        \App\Http\Middleware\EncryptCookies::class,
        \Illuminate\Cookie\Middleware\AddQueuedCookiesToResponse::class,
        \Illuminate\Session\Middleware\StartSession::class,
        \Illuminate\View\Middleware\ShareErrorsFromSession::class,
        \App\Http\Middleware\VerifyCsrfToken::class,
        \Illuminate\Routing\Middleware\SubstituteBindings::class,
    ],
    'api' => [
        'throttle:60,1',
        'bindings',
    ],
];
```
### web路由
- **EncryptCookies**

 加解密cookie
- **AddQueuedCookiesToResponse**

 cookie自动添加到header，一般情况下用户不用干预
- **StartSession**

 开启start_session()
- **ShareErrorsFromSession**

 从session读取表单提交的历史数据，便于取值old('field_name')，以及读取错误
- **VerifyCsrfToken**

 对每一个POST DELETE PUT请求的页面，都会做Csrf 检查，
 
 详情请参见 [控制器 Controller](chapter-mvrc/控制器-controller.md)相关篇章
### api路由

# # 路由的中间件
只有当路由开启本中间件时，方会调用。比如用户验证、权限验证等操作

```php
protected $routeMiddleware = [
	'alias_name' => \App\Http\Middleware\XXXX::class
];
```
系统默认自带这些
- **auth -> Authenticate**
验证用户登录
- **auth.basic -> AuthenticateWithBasicAuth**
基本的用户登录验证
大家在登录一些「FTP（网页）、交换机、路由器」页面时，会弹出一个系统的登录框，便是这个
- **guest -> RedirectIfAuthenticated**
检查访客是否有已经有登录态（如果保存了登录态）

## ## 路由开启中间件
```php
// 检查登录态，否则跳转登录
// auth 为 中间件 Authenticate 的别名
Route::group(['middleware' => 'auth'])
```

# # 控制器中间件（5.3）
在每个Controller的构造函数中，也可以调用中间件
```
class UserController extends Controller {
	public function __construct()
	{
		$this->middleware(['auth']);
	}
}
```

带来的问题


# # 创建中间件
## ## artisan 创建
```
$ php artisan make:middleware WechatOAuth2
```
## ## 工具箱 创建

访问 `http://127.0.0.1/**project_name**/artisans` 使用

## ## 基本结构

此处以微信公众号的OAuth2访问为例
下图为该中间的运行逻辑
```flow
st=>start: 访问页面
cond=>condition: 检查Session中的
wechatUser是否存在
op=>operation: 跳转微信页面
进行授权
e=>end: 正常显示页面

st->cond
op->e
cond(yes)->e
cond(no)->op
```

```php
<?php

use Closure;
class WechatOAuth2
{
	public function handle($request, Closure $next)
	{
		if (empty('读取session中的wechatUser'))
		{
			if ($request->ajax()) {
				return response('Unauthorized.', 401);
			} else {
				return redirect('微信oauth2授权地址');
			}
		}

		return $next($request);
	}
}
```

