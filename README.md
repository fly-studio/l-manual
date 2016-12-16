# # 概述
L+(L plus)，全称为Laravel+，顾名思义，这是Laravel框架的plus。
L+诞生于2013年，最初是基于Kohana 3.3研发，现基于Laravel 5.3研发。
本补给工具包含众多优秀的方法、函数和编程思想。并且大部分架构是由作者基于其十多年的研发经验所创！

# # 为什么是Laravel？
Laravel并不是一个新型的框架，它的成功源自于广大PHPer对源代码的贡献和支持。Laravel可以让你从面条一样杂乱的代码中解脱出来；它可以帮你构建一个完美的网络APP，而且每行代码都可以简洁、富于表达力。
这是2015年PHP框架排名：[sitepoint](http://www.sitepoint.com/best-php-framework-2015-sitepoint-survey-results/ "sitepoint")
![](/assets/1427547421php_framework_popularity_at_work_-_sitepoint2c_2015.png)

# # 学习难度
由于Laravel的学习曲线非常陡峭（入门易、深入难），所以本plus也同样有此特点。但并不代表增加了入门的难度，反而在一定程度上降低了入门难度。
L+可以帮助新手快速的入门Laravel，也可以帮助老手释放枯燥的CRUD，因而L+适合每一个水平层面的使用者。

# # L+特点
- L+ 没有修改原框架核心程序，可以随时使用composer升级核心laravel程序
- L+ 的主程序以 addons 开头，已经提交到 packagist.org
- 简单的验证流程
	- 前台自动调取jQuery.validation验证
	- 后台自动返回验证结果（json、错误页面、返回前页报错）
- 非常方便插件机制
	- 可以将每个功能解耦成单独的文件夹，提高团队、代码管理效率
- 全文搜索引擎
	- 集成 php-elasticsearch，可支持拼音、ik中文分词等
	- 优化了 scout 中对 elasticsearch 5.x 的搜索
	- 集成 logstash 日志输出
- 调试方便
	- 将后台日志实时输出到 Chrome、Firefox（需要安装对应的插件）
- 附件上传，适应任何浏览器（包括手机）
	可限制文件后缀、文件大小。以及秒传大文件等
- 附件预览、下载
	- 支持上传 >1 GB 的文件，分片上传，并且去重
	- 输出Etag，自动304
	- xSendFile，支持Apache、nginX，squid，lighttpd
	- 预览时，图片等比缩放
	- 视频自动截取缩略图
	- 自适应手机
	- 支持分布式存储等
- 数据导出
	- 支持Excel、PDF、YAML等数据的导出
	- 支持JSON、JSONP、Javascript、XML以及跨域数据的获取
- CRUD集成解决方案，可以非常快捷的实现增删查改
- 以上每个功能的实现，只需要**一到三行代码**

# # 本文档特点
- 简体中文
- 简化了原手册中不太常用的方法
- 着重讲解路由、控制器、验证、ORM

