# # 获取程序
> 代码托管于作者自建的git服务器中
> 关于GIT的安装和使用方法，请查看[「GIT使用手册」](http://www.load-page.com/base/manual "「GIT使用手册」")

建议将代码存放在 <code>$webroot/project/</code> 下
```
$ git clone -b 5.3 http://git.load-page.com/r/laravel.git
$ git clone -b 5.3 http://git.load-page.com/r/lpp.git l++
$ git clone -b 5.3 http://git.load-page.com/r/lp.git project_name
$ git clone http://git.load-page.com/r/static.git
```
请修改上文中的 project_name

> TortoiseGit用户在 windows资源管理器中，点击右键 选择 Clone

* 本源代码并没有做安装程序，无法从0开始使用（目前只有作者周围的同事和朋友在用，如果你对这套程序感兴趣，欢迎和我联系）

* l+.git 在 clone 之后，如需将本项目托管到其它版本管理平台，需删除<code>/project_name/.git</code>文件夹（这是一个隐藏文件夹）

* 获取Git的更多使用方法，请查看 [GIT 操作手册](http://www.load-page.com/base/manual "GIT 操作手册")