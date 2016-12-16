# # 调取模板
比如模板文件：views/admin/user/edit.tpl
```php
$this->_param = $data; //注入变量
return $this->view('admin.user.edit');

//下面原生的用法，但是在L+中不建议使用
return view('admin.user.edit')->with('param', $data);
```

- 无需后缀
- 目录分隔用[.]，而不是[/]

# # 模板引擎优先级
Laravel按照如下优先级寻找模板文件
## ## 文件后缀
- **.php**
PHP和HTML混杂的模式，不推荐使用
- **.blade.php**
Laravel自带模板引擎
- **.tpl**
Smarty模板引擎，Smarty为L+增加

> 如果同时存在index.php index.blade.php index.tpl，按照优先级，只会读取 index.php

## ## 文件目录优先级

```php
/project_name/resources/views/
/laravel/vendor/addons/core/resources/views/
```

- Laravel会尝试在这两个文件夹中寻找以php/blade.php/tpl结尾的模板文件
- 如果两个目录有同名文件，则会优先读取第一个文件

> L+将一些通用文件放到了<code>/laravel/vendor/addons/core/resources/views/</code>中。
> 比如：主模板、常用脚本、样式，以及前端验证控件、上传图片控件、富文本编辑器


## ## laravel项目视图的优先级
初学者可能看不懂这是什么目录，比如第三方插件下的视图为：
```php
laravel/vendor/name/resources/views/index.tpl
// 普通用户肯定无法修改第三方的文件，但是可以通过下面的路径替换
resources/views/vendor/name/index.tpl
// 此路径优先级较高
```
第三方插件如何使用自己目录下的视图此处不深究，需要可以查看官方文档。

# # 模板使用
本文教程只介绍Smarty的用法
Smarty中文教程：
http://www.smarty.net/docs/zh_CN/

## ## smarty 配置
```
//app配置，默认不存在
/project_name/config/smarty.php
//原始配置文件
/laravel/vendor/addons/smarty/config/smarty.php
```
- **caching** => false
是否缓存模板
- **cache_lifetime** => 60
单位秒，模板更新时间。在这个时间内，读取缓存(即使模板已经更新)
所以在开发时 caching最好关闭，生产环境因为模板不会随时变化，可以开启这两项以缓解模板解析压力
- **left_delimiter** => <{
- **right_delimiter** => }>
原生的smarty这两项为{}，但是在l+中，这两项加了<>，此举的作用是为了避免内嵌javascript造成的在模板解析失败
- **escape_html** => true
自动为所有变量输入增加 |escape，这样可以避免XSS注入漏洞，建议开启


## ## 输出变量
在Controller中使用 $this-> 方式输出变量，此处约定为下划线打头
```php
$this->_title = 'This\'s a title';
$this->_data = ['title' => 'Who am i']；
$this->_html = '<a href="">Title</a>';
$this->_script = '<script>alert('');</script>';
$this->_user = User::find(1);
```
在模板文件中
```html
变量打印
<{$_title}>

数组子项输出
<{$_data.title}>
<{$_data['title']}>

KEY为变量
<{$_data[$_param.value]}>

escape_html 开启，会自动过滤html代码，加上nofilter可以避免过滤
<{$_html nofilter}>

手动调用|escape，过滤script代码
<{$_script|escape:'noscript' nofilter}>

//Model的调取方法
<{$_user->username}>
<{$_user.username}>
<{$_user['username']}>

//Object调取方法
<{$_user->count()}>

//函数调取
<{method_field('PUT') nofilter}> 
//会输出<input type="hidden" name="_method" value="PUT"> 
```

## ## 调取PHP函数
任何PHP函数都可以在Smarty模板中调取
```html
将数据转化为json（注意nofilter）
<{$_data|json_encode nofilter}>
如果报错，可以在前面加上@
<{$_data|@user_func}>
```

## ## 调取多函数，多参数
```html
多个函数：
- 以下代码表示：转义HTML代码，并且将回车转为<br>;
- 使用 | 分隔多个函数
<{$_data.content|escape|nl2br}>

多个参数：
- 以下代码表示：截取20个字符，多余的字符用[...]表示
- 使用逗号分隔参数
<{$_data.title|truncate:20,'...'}>

参数为其他变量
<{$_date.title|user_func:$param['value']}>
```

## ## 表达式
注意Smarty不支持过于复杂的表达式
```html
<{$i * 25 + 1}>
<{($i / 100)|round}>%
```

## ## IF 判断
使用方法类似于PHP，只是没有了括号()和大括号{}
```html
<{if $param == '123' || $param == '456'}>
显示:<{$data1}>
<{else if !in_array($param, array(1,2,3,4,'567'))}>
显示:<{$data2}>
<{else}>
显示:<{$data3}>
</{if}>
```

## ## 数组循环
### foreach
```php
$this->_user = User::all();
```
```html
<{foreach $_data as $key => $item}>
<div><{$item->username}></div>
<{foreachelse}>
Nothing!
<{/foreach}>
Count: <{$_user->count()}>
```
1. 需要获取已经循环了多少次，smarty可以这么使用
	- @index 从0开始计数
	- @iteration 从1开始计数
	- @first 是否第一个
	- @last 是否最后一个
	```html
	<{foreach $_data as $item}>
		<{if !$item@last && $item@iteration % 2 == 0}>
		偶数行,末行除外
		</{if}>
	</{foreach}>
	```
2. 跳出或者继续
	- {break} 跳出循环
	- {continue} 继续下一个循环

	```html
	<{foreach $_data as $item}>
		<{if !$item@first}>
			<{continue}>
		<{else if $item.value == 2}>
			<{break}>
		</{if}>
	</{foreach}>
	```

> 但是不建议在smarty中操作break、continue这些，按照标准的程序的设计，应该是在定义变量时就进行相应处理，保证数据的正确之后，再注入模板。

### for
```php
<{for $i=0 to $_user->count()-1 step=1}>
<{$_user[$i]}>
<{/for}}
```

# # 嵌套模板与模板重载
## ## 嵌套模板的使用
当项目中的HTML存在大量的相同代码时，可以将这些代码提取到一个单独的模板中，简称为「include模板」

> include模板在Smarty中使用的非常广泛

公用的脚本inc模板 views/common/scripts.inc.tpl
```html
<script src="jquery-2.1.0.js"></script>
<script src="bootstrap.js"></script>
```

控制器访问的页面：views/user/edit.tpl
```html
<html>
<head>
<{include file="common/scripts.inc.tpl"}>
</head>
</html>
```
- 嵌套的路径
	- 相对路径表示相对views/开始的路径
	- 绝对路径则输入完整的：/www/website/xx/xx/xx.tpl
- 必须要有文件的后缀
- include模板更新，只会编译该模板，嵌套他的父级模板会自动显示最新数据
- 因为底层是用php的include来实现的

## ## Block 模板重载

**建议大家都使用这种模板重载模式**
按照官方的说明，效率高于嵌套模板，并且有助于缓存、页面更新。
因为嵌套模板的方式，也会有大量重复的代码，大量重复的代码不利于项目的维护。
使用模板重载，只需要修改一个主模板，就可以更新所有的子模板，类似面向对象的做法

### 运用场景
比如在后台的运用场景中，大部分页面都是相似的，这个可以参考后台的「用户管理」。

后台主模板：<code>project_name/resources/views/admin/extends/main.block.tpl</code>
这是一个后台的页面，里面包含了后台需要使用的脚本、样式、侧边栏、以及一个空的内容区域。

常用的后台功能中，包含「列表」、「编辑」、「新增」三个页面，所以在同文件夹，还有*list.block.tpl*、*edit.block.tpl*、*create.block.tpl*，这三个文件重载于main.block.tpl，修改了部分block区域的内容，以完成不同的功能和显示。

在 *Admin\MemberController* 中所指向的的 views/admin/member/list.tpl、edit.tpl、create.tpl 三个视图，便重载于上述三个父模板，大家可以看到里面需要修改内容已经非常少。

### 父模板
<{block '名称'}>默认内容<{/block}>：定义一块区域，以进行模板继承，内容可以为空，或者有默认值，
如果子模板不改写这个block区域，则显示默认内容

```html
parent.block.tpl

<html>
	<head>
		<{block 'head-title'}><title>标题<{block 'head-title-subtitle'}></{block}></title></{/block}>
		<block 'head-styles'>
			<link rel="stylesheet" href="bootstrap.css" />
			<{block 'head-styles-plus'}></{block>
		</block>
	</head>
	<body>
	<{block 'body'}></{block}>
	</body>
</html>
```

### 子模板

```html
sub.block.tpl

<{extends file="parent.block.tpl"}>

<{block 'head-title-subtitle'}> - 子标题<{/block}>

<{block 'head-styles-plus'}>
<style>
.container {width: 80%;}
</style>
</{block>

<{block 'body'}>
...
</{block}>
```
输出
```html
<html>
	<head>
		<title>标题 - 子标题</title>
		<link rel="stylesheet" href="bootstrap.css" />
		<style>
			.container {width: 80%;}
		</style>
	</head>
	<body>
	...
	</body>
</html>
```

- extends 必须放在第一行，表示载入一个父模板
- block 表示修改对应父级的内容，无需修改则不用写该block
- block 中可以使用<{include file=""}>
- 可以在block里面，再次新建一些block，便于子孙模板重载
- extends file中不要使用变量，因为smarty会将这个变量的值编译，当这个变量在下次出现不同值时，模板仍然使用的是上次的变量值
- 启用extends之后，<{block}><{/block}>之外的内容全部忽略
- 同上道理，无法使用if载入不同的block、extends，因为block外的内容全部忽略

> 读者可以参考main.block.tpl，可以看到文中定义了非常多的block，比如head-scripts-before，head-styles-after，这样方便你在脚本、样式的不同区域添加代码，保证了灵活性。
> 如果读者尝试来做一个主模板，也需要站在每个使用者的的角度，尽量定义多个block区域，以方便他人修改样式和脚本，包括可自定义jquery，bootstrap的版本，或者还可以在不载入head-styles这种大类的情况下，删除、替换某些样式或脚本等


# # Smarty 内置函数
<div class="alert alert-warning">代码中有与smarty同名的PHP函数，会优先使用smarty的修饰函数</div>

## ## escape
$esc_type, $char_set, $double_encode
> **@param** string $esc_type [html] 需要转移的方式：html,htmlall,url,urlpathinfo,quotes,hex,hexentity,decentity,javascript,mail,nonstd,noscript,nohtml
> **@param** string $char_set [UTF-8] 字符编码
> **@param** boolean $double_encode 双字节开关
> **@return** string 返回转义之后的结果
> 实现文件在<code>laravel/vendor/addons/smarty/plugins/modifier.escape.php</code>

escape函数是smarty函数中非常重要的函数，如果你使用的smarty从未使用过这个函数，那么你的网站是非常危险的

这个函数主要是用于预防「XSS注入」，XSS的危险性，也许你不太觉得，但是在腾讯网（网站）下，一个XSS漏洞，可以导致数以百万计的QQ处于被盗风险。XSS是盗窃cookie，攻入后台的罪魁祸首！

使用方法
```html
<title><{$_date.title|escape}></title>
```

> 在L+中，默认开启了escape_html，所以你不用加上escape: <{$_item.title}>，Smarty已经自动为你加上这个函数
> 此处是为示例
> 无需转义的场景，请用nofilter，明确指定无需转义

另外还有转换url,hex等功能，用于不同的场景，比如
```html
<a href="/admin/redirect?url=<{$_data.url|escape:'url' nofilter}>"></a>
```

经过L+改良，增加了nohtml、noscript
这样方便过滤富文本编辑器中的script/style
```html
<{$_content|escape:'noscript' nofilter}>
```

------------


## ## url
$nocache
> **@param** boolean $nocache 是否对网址做无缓存处理
> **@return** string 返回绝对网址
> 实现文件在<code>laravel/vendor/addons/smarty/plugins/modifier.url.php</code>

这个函数是L+增加的内容，可以方便你在模板文件中，将网址转化为绝对地址，这样可以减少路径访问带来的错误
```html
<a href="<{'admin/wechat/account'|url}>?id=<{$item->getKey()}>"><img src="<{'static/img/1.jpg'|url}>" /></a>
```

对于一些需要无缓存的网址，比如验证码图片，可以设置$nocache参数
```html
<img src="<{'captcha'|url:true}>" onclick="this.src='<{'captcha'|url:true}>';" alt="点击刷新"/>
```
这个的实现原理，就在在网址后面加入了一个_(下划线)的参数
```
http://.../captcha?_=0.11554461466 (随机数)
http://.../placeholder?text=xxx&_=0.45456465464
```
> 为了效率考虑，并没有判断网址中有#，随机数总会加到网址末尾，这可能不是你想要的结果，所以尽量将#写在smarty模板外：
> ```
<{'captcha'|url:true}>#hash
```


------------


## ## truncate
$length, $etc, $break_words, $middle
> **@param** integer [80] 需要截取的长度
> **@param** string $etc [...] 多余字符的表现形态
> **@param** boolean $break_words [false] 是否截断有意义的英文单词
> **@param** boolean $middle [false] 是否截断的是中间的内容
> **@return** string 返回截取之后的字符串
> 实现文件在<code>laravel/vendor/addons/smarty/plugins/modifier.truncate.php</code>

此函数是用于缩减字符串，对于中文、非英文字符是安全的，不会出现截断半个中文字的情况。

此函数经过L+改良，将汉字的宽度设置为2，也就是说，你需要截断的时候设置$length，1个汉字是2个长度
```
<{'中文中文'|truncate:5}> => 中文中 (长度会修正到完整的汉字，也就是6)
```
这样的用途是为了更好的在页面上，将汉字截取的和英文等长
```
· this is amazing
· 汉字标题和英文等长
```


对于这种场景：「139\*\*\*\*5678」,可以配合$middle函数来实现
```
<{$_phone|truncate:7:'****':false:true}>
```
------------

## ## date_format
$format， $default_date, $formatter
> @param string [null] $format 需要转换的格式
> @param string [] $default_date 默认时间，为空表示当前时间戳
> @param string [auto] $formatter 格式化函数 strftime、date、auto
> @return string 返回格式化后的时间
> 实现文件在<code>laravel/vendor/smarty/smarty/libs/plugins/modifier.date_format.php</code>

根据时间戳（或DateTime、mysql timestamp），以及$format格式，返回对应的时间字符串
$formatter为auto时，程序会自动检测 $format 中是否有 % 字符，以决定使用 strftime 还是 date 的方式来格式化时间

```
$this->_time = time();

...

<{$_time|date_format:'%Y-%m-%d %H:%M:%D'}>
<{$_time|date_format:'Y-m-d H:i:s'，'2015-09-22 03:08:59'}>

```

> Laravel中的create_at、updated_at均为Carbon，无需使用本函数进行转换

------------
## ## PHP 函数的调取方式
```
<{$param|function_name:param2,param3,param4,...}>
```
任何php函数都可以在smarty中调用，中竖线 | 之前的内容为第一参数，后面的参数用逗号分隔
```
<{$_data|substr:1,2}> 等同于 substr($_data, 1, 2);
```
------------
# # CSRF攻击
在Laravel中，使用全局中间件开启了Csrf攻击的检查，对于非GET的请求方式，一律检查csrf数据是否正确。

## ## CSRF攻击的方式和场景
### 偷链重要网址诱导管理员操作
比如：http://.../admin/member/delete/10 是你删除用户的网址。
攻击者无需得到你的管理员账号，只需做一个网页，网页中有一个图片的src为上述网址，然后诱导管理员打开这个网页，接着，因为你访问了这个删除用户的网址，系统便删除了10这个用户，并且毫无察觉。

原因：
因为管理员本地保存了后台登錄的cookie，当管理员打开这个网页时，图片会访问该网址，该网址会自动附上后台的cookie，具备管理员权限的网址删除了该用户。

### 第三方提交
跨域POST提交，虽然每个浏览器都是禁止的行为，但是解决方法太简单：
只要本地建立一个服务器，在host中将该网站的某一子域名指向到 127.0.0.1，以及document.domain = '<mark>domain.com</mark>';
你就可以读取到该域名的cookie保持登录态，使用本地服务器给这个服务器POST提交数据了。

一些刷票、刷稿，用的就是这个原理。
其实还有更简单的方式：cURL可以全能模拟提交，结合上述方式更是如虎添翼。

## ## 通用解决方式

在每一个表单中，附带一个服务器提供的随机字符串。
提交到服务器之后，检测这个字符串是否和session中的一致。

弊端：
**长时间的停留页面，session失效后提交也会crsf出错**
> 所以Discuz!等论坛，由于写贴时间过长，发帖时只检查图片验证码，而不会启用crsf检查
> PHP的Session保存时间20分钟。
> 对于帖子、长内容的编辑页面，最好做好内容备份、恢复的功能，不然页面报错之后内容丢失，是一个非常失败的用户体验

## ## Laravel解决方式
如果你使用jQuery，统一修改ajax头

```html
head中添加如下语句（smarty）：
<meta name="csrf-token" content="<{csrf_token()}>">

<script>
$.csrf = $('meta[name="csrf-token"]').attr('content');
if ($.csrf) {
	$.ajaxSetup({
		headers: {
			'X-CSRF-TOKEN': $.csrf
		}
	});
}
</script>
```
> main.extends.tpl 和 common.js 已经加入了这个功能
> 故而L+项目中使用ajax提交数据，会自动附加_token，无需操心

针对原生表单提交，需加入这行
```html
<form>
<{csrf_field() nofilter}>
</form>
```