# Flask的大型网站模板
第一次学习使用Flask的时候是为了赶鸭子上架，那时候python和flask的优势就出来了，没有任何网站开发经验，两天部署出一个简单的微信公众平台，说到底这都是开源的力量，开源让我们处处有免费的代码使用，甚至可以进行二次开发。一个流行的开源软件，势必会银乐公众的瞩目，然后经过社区或者开发人员的验证，我们可以非常放心的使用，不过那次开发仅仅使用flask的路由功能，然会数据都是通过**make_response**函数来手动构造的，随后潜心学习了Falsk，从《FlaskWeb开发：基于Python的Web应用开发实战》一书中，收货很多，坐着丰富的开发经验，给Pythoner和Flask爱好者带来了盛宴。

好了废话已经很多了，现在该正题了。Flask是Python中很流行的Web后端框架，其受欢迎度不输于Django，Flask是一个使用 Python 编写的轻量级 Web 应用框架。有些人喜欢叫Flask是微框架，下面给出一些解释：
>“微”并不代表整个应用只能塞在一个 Python 文件内，当然塞在单一文件内也是可以的。 “微”也不代表 Flask 功能不强。微框架中的“微”字表示 Flask 的目标是保持核心既简 单而又可扩展。 Flask 不会替你做出许多决定，比如选用何种数据库。类似的决定，如 使用何种模板引擎，是非常容易改变的。 Flask 可以变成你任何想要的东西，一切恰到 好处，由你做主。

所以Flask相当小型，而且开发起来也很顺手。现在，如果要使用Flask进行大型网站开发，怎么办？Flask其实提供了开发网站的方式，我们可以从程序结构上来入手。让我们慢慢进入主题。

### 一、Flask的Web引擎
对Web或者TCP/IP有所研究的人，都对HTTP不陌生吧，HTTP其实是一个非常简单的协议，运行在可靠的TCP层，然后通过四种方法来对服务器资源进行控制，当然现代的Web应用，让HTTP支持更多特性，必须长连接什么的~

在Python中有WSGI（Web Server Gateway Interface）来定制一个Web应用应该如何响应客户端的HTTP请求，在Flask，以来两大底层库，Werkzeug作为WSGI的工具箱，还有一个Jinja2的模板引擎。他们已经被深度集成在Flask中，在开发时，路由和响应使用了Werkzeug，render_temolate函数应用了Jinja2。

### 二、Flask的MVC影子
我不能断言Flask是依照MVC设计的，但是确实可以进行类似MVC的程序设计，这也是第一部分为什么非要说明Python的底层库，MVC互联网上有更专业的说明。
>MVC 是一种使用 MVC（Model View Controller 模型-视图-控制器）设计创建 Web 应用程序的模式：
>Model（模型）表示应用程序核心（比如数据库记录列表）。
>View（视图）显示数据（数据库记录）。
>Controller（控制器）处理输入（写入数据库记录）。
>MVC 模式同时提供了对 HTML、CSS 和 JavaScript 的完全控制。
>Model（模型）是应用程序中用于处理应用程序数据逻辑的部分。
>　　通常模型对象负责在数据库中存取数据。
>View（视图）是应用程序中处理数据显示的部分。
>　　通常视图是依据模型数据创建的。
>Controller（控制器）是应用程序中处理用户交互的部分。
>　　通常控制器负责从视图读取数据，控制用户输入，并向模型发送数据。

对于Flask来说，可以进行如下对照：
- Flask的路由功能，属于MVC的Controller，控制请求的响应方法；
- 对于路由覆盖的函数，就是MVC的View，处理请求，并返回需要渲染的html；
- 而类似的SQLalcehmy，就是MVC的Model，利用ORM或者预封装的数据模型，来控制数据的修改和更新。

当然，对于Flask来说，或者说现代的Web架构来说，这里面有许多边界模糊的地方，但是我们可以根据MVC模型指引，来引出对Flask的结构化开发，来达到软件的耦合度降低，减小软件维护难度。
需要说明的是，Flask并没有提供ORM模型，需要使用别的开源程序，比如流行的SQLAlchemy，ORM可以让我们更注重数据模型的建立，并建立数据绑定行为，来降低Model模型的耦合度。

### 三、大型网站的程序结构
有了前几部分的铺垫，我们现在可以说明基于Flask的Web程序结构了，让我们先一睹为快：
```bash
|-Project
	|-app/
		|-templates/
		|-static/
		|-user/
			|-__init__.py
			|-forms.py
			|-views.py
		|-main/
			|-__init__.py
			|-errors.py
			|-views.py
		|-models.py
		|-__init__.py
	|-requirements.txt
	|-config.py
	|-manage.py
```
让我们基本介绍一下各文件的作用：
* manage.py
		该脚本是开发者与程序的交互入口，比如启动Web程序，调试数据库，运行自定义命令等等
* config.py
		该脚本是Web的配置文件，基本包括了该Web应用需要的配置信息，比如某些常数配置，或者环境变量（通常是处于保护隐私），数据库配置等等
* requirements.txt
		该文件由pip自动生成，是本应用的python环境依赖包及其版本。
* app文件夹
		app文件夹是本Web程序的主要功能设计部分，包括了路由路径设计，模板文件，静态文件和模型设计。
由于app部分的重要性，我们继续展开讲。
1. 模板与静态文件
模板就是服务器传给客户端的视图部分，可以通过Jinja2的模板引擎吧需要的内容，渲染到模板中，然后通过网络呈现给用户。
静态文件是网页所以来的css样式、js脚本、图标等部分，这一部分可以在部署生产环境的时候，交给反向代理器去做，减小Web后端的负担。
2. 数据库模型设计
数据库模型一般由ORM方式设计，然后提高程序的可用性。
3. 功能设计
在真实的设计场景中，会涉及多个功能，比如用户登录，博客文章，评论，用户管理等等。这时我们可以把他们分开设计，然后用上述文件结构进行功能性划分，每个功能文件夹中依照具体需求，可能会有不同的设计文件，但是肯定会有views视图文件的，这个是功能展示的基础。

好了这就是一个可以良好Web架构的程序结构了，欢迎大家拍砖。随后我会开源一个模板工程，同时会出更多的教程来讲解这个模板工程的功能，及其使用。谢谢大家捧场~~


我的个人Github主页：[GalaIO](https://github.com/GalaIO)
一个开源的Flask博客系统：[GalaCoding](https://github.com/GalaIO/GalaCoding)