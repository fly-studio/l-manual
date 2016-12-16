> 本章对PHP的要求较高，普通开发者可忽略

# # 什么是插件
在project_name 和 l++ 目录下，都有一个文件夹vendor

这个vendor下存放的文件，便是l+的**「可拔插」插件**

```l++/vendor``` 是由作者维护的一套插件，包含attachment、wechat、tools等

```project_name/vendor``` 是由开发者维护的插件，目前为空

就目前来说，L+能如此便利的原因，这些插件功不可没。
比如附件系统、二维码生成、工具箱等

> **L++** 和 **L+** 意义不一样，**L++** 是 **L+ plugins**的意思

# # 插件编写
## ## 文件夹：
tools 插件为例
```
l++/vendor/tools/
```
```
/app
	/Http
		/Controllers
			QrController.php  二维码Controller文件
		/Middleware
			Local.php
	Manual.php                Model文件
/config
	plugin.php               必要文件
	validation.php           验证文件
/resources
	/lang
		/zh-CN
	/views
routes.php                   路由文件
```

可以看出，文件夹的结果和普通项目一模一样。

## ## 主配置

```
//主配置文件
laravel/vendor/addons/core/config/plugin.php
//插件下的配置文件
l++/vendor/tools/config/plugin.php
```
解释
```php
<?php
//本文件只是示例文件，请使用config('plugins.{PLUGINNAME}')调取对应插件的配置，
//{PLUGINNAME}对应下文的'name'，这里假设为tools
//插件的namespace假设为Plugins\Tools

return [
	'enable' => TRUE, //开关，已停止的插件，建议设置FALSE，以避免浪费资源
	//插件名(英文、数字、-、_)，全局唯一，符合PHP变量名规范，为空代表使用当前文件夹名，
	'name' => NULL, 
	'display_name' => '', //本插件的名称，比如：工具箱
	'description' => '', //本插件功能简介
	//本插件的namespace，为空代表使用name，-_在转换为namespace会变为驼峰
	//[name]      转    [namespace]
	//tools       ->    Plugins\Tools
	//wechat-abc  ->    Plugins\WechatAbc
	//hi_world    ->    Plugins\HiWorld
	'namespace' => NULL, 
	'path' => NULL, //本插件的路径，这个值不用赋值，会被程序自动配置
	'register' => [ //注册namespace
		//是否注册/tools/resources/views到视图
		//- Controller中这样调用：view('tools:system.xxx'); 对应/tools/resources/views/system/xxx.tpl
		//- smarty模板中这样调用：<{include file="[tools]system/nav.inc.tpl"}>
		'view' => false, 
		//是否注册/tools/database/migrations到迁移
		'migrate' => false, 
		//是否注册/tools/resources/lang到语言包
		//- Controller中这样调用：lang('tools:valition.alpha_dash');
		//- smarty模板中这样调用：<{'tools:valition.alpha_dash'|lang}>
		'translator' => false,
		//是否自动设置路由
		//- 自动加载/tools/routes.php
		//- 当为true时，调用下文router的namespace,prefix,middleware配置
		'router' => false,
		//是否将/tools/config/validation.php合并到主配置config/validation.php
		//- 示例可以查看tools/config/valition.php
		//- 为避免覆盖掉主配置，请谨慎设置键值
		'validation' => false,
	],
	'routers' => [ //Route::group(['namespace' => '?', 'prefix' => '?', 'middleware' => '?']);
		'web' => [
			'namespace' => NULL, //本插件下Controller的路由的namespace，空代表使用Plugins\tools\App\Http\Controllers
			'prefix' => '/', //路由的prefix，空代表 / (根目录)
			'middleware' => [], //路由的中间件，不启用中间件时，必须为空数组，不能设为NULL
		],
		'api' => [
			'namespace' => NULL, //本插件下Controller的路由的namespace，空代表使用Plugins\tools\App\Http\Controllers
			'prefix' => 'api', //路由的prefix，空代表 / (根目录)
			'middleware' => [], //路由的中间件，不启用中间件时，必须为空数组，不能设为NULL
		],
		//....
	],
	//全局中间件，会被自动调用
	//参考 /app/Http/Kernel.php
	'middleware' => [
		// \Plugins\Tools\App\Http\Middleware\VerifyCsrfToken::class,
	],
	'middlewareGroups' => [
		// 'web' => [],
		// 'api' => [],
	],
	//路由中间键 附加到路由中
	//参考 /app/Http/Kernel.php
	'routeMiddleware' => [
		// 'cry' => \Plugins\Tools\App\Http\Middleware\Cry::class, 
	],
	//自定义artisan命令
	//参考 /app/Console/Kernel.php
	'commonds' => [
        //\Plugins\Tools\App\Console\Commands\Inspire::class,
	],
	//插件中的模板注入到主模板（确保相同路径）暂只支持smarty
	//- 注意：注入不是智能的，只有当主模板中有<{pluginclude file='admin/sidebar.inc.tpl'}>时，程序会尝试按照顺序插入所有插件中的模板
	//- 插入指定插件的模板：<{pluginclude file='admin/sidebar.inc.tpl' plugins="tools;wechat;xxx"}> 或者使用原生语句<{include file='[tools]admin/sidebar.inc.tpl'}>
	//- 为避免模板被重复(死递归)嵌套 pluginclude子模板中的pluginclude会直接忽略
	'injectViews' => [
		// 比如 管理员后台的菜单，会尝试<{include file="[tools]admin/sidebar.inc.tpl"}>
		// 没有这行，不会插入
		// 'admin/sidebar.inc.tpl',
	],
	//需要读取的配置文件，请勿加入plugin,validation
	'config' => [
		//比如config/attachment.php
		//'attachment',
	],

];
```


## ## namespace 前缀
```
Plugins\{PLUGINNAME}  //PLUGINNAME 来自于plugin.php 中的 namespace
```


比如：
```
l++/vendor/tools/app/Manual.php

namespace Plugins\Tools\App;
```
```
l++/vendor/tools/app/Http/Controllers/ManualController.php

namespace Plugins\Tools\App\Http\Controllers;
```

## ## 静态文件
l++插件中的```l++/static``` 被软链接到 ```project_name/plugins```

调取方式如：
```
<img src="<{'img/wechat.jpg'|plugins}>"> # 推荐
或
<img src="<{'plugins/img/wechat.jpg'|url}>">
```
这种url的重写，在项目htaccess中已经集成，nginX的重写请参考[运行环境](http://www.load-page.com/base/manual/3#h2--nginx--4-1 "运行环境")

## ServiceProvider.php
有能力写这个文件的，说明你已经通读了插件的实现代码，手册也帮不了你了 :p

插件的实现文件在这里：
```
laravel/vendor/addons/core/src/ServiceProvider.php

registerPlugins();
bootPlugins();
```

