# CSRF-Attack

## 配置

软件：Vmware Workstation 12 Player

系统：SEEDUbuntu12.04

网络：NAT

其他：

1. Firefox浏览器(安装好LiveHTTPHeaders插件，用于检查HTTP请求和响应)；
2. Apache网络服务器；
3. Elgg网页应用

### Starting the Apache Server

开启Apache服务器很简单，因为已经预装在SEEDUbuntu中了，只需要使用一下命令开启就行：

![配置](https://raw.githubusercontent.com/familyld/CSRF-Attack/master/graph/image2.png)

### The Elgg Web Application

这是一个开源基于网页的社交应用，同样已经预装在SEEDUbuntu里，并且创建了一些用户，信息如下：

![配置](https://raw.githubusercontent.com/familyld/CSRF-Attack/master/graph/image3.png)

###配置DNS

![配置](https://raw.githubusercontent.com/familyld/CSRF-Attack/master/graph/image4.png)

这个项目需要用到以上两个URL，启动Apache之后就可以在虚拟机内部访问到了，因为在hosts文件中已经配置了这两个URL是指向localhost的：

![配置](https://raw.githubusercontent.com/familyld/CSRF-Attack/master/graph/image5.png)

###配置Apache服务器

项目中使用Apache服务器作为网站的host，借助name-based virtual hosting技术，可以在同一台主机上host多个网站。配置文件default在/etc/apache2/sites-available/路径下：

![配置](https://raw.githubusercontent.com/familyld/CSRF-Attack/master/graph/image6.png)

可以看到Apache用的是80端口，每个网站都有一个VirtualHost块，指定网站的URL和对应的资源目录。比如上面这个block，就绑定了[www.CSRFLabElgg.com](www.CSRFLabElgg.com)和/var/www/CSRF/elgg路径，修改该路径下的文件就能改变网站的配置。

## 什么是CSRF攻击

CSRF全称Cross-Site Request Forgery，是跨站请求伪造攻击的意思，这里面有两个关键词，一个是跨站，一个是伪造。前者说明CSRF攻击发生时所伴随的请求的来源，后者说明了该请求的产生方式。所谓伪造即该请求并不是用户本身的意愿，而是由攻击者构造，由受害者被动发出的。流程如下：

![CSRF](https://raw.githubusercontent.com/familyld/CSRF-Attack/master/graph/image7.png)

用户首先登录了一个普通网站A，这时新的session就会被创建，用户的浏览器会自动存储cookie信息以及session id，这样短期内再打开网站或者网站的其他链接，用户就不需要重复登陆了。如果此时，用户访问了攻击者制作的恶意网站B，攻击者设置的对A的跨站请求就会被触发，并且由于浏览器的机制，cookie信息和session id会被附加到这个请求中(即使它是跨站的)。于是，A就会以为这个请求是用户发出的从而执行请求内容。只要稍加设计，攻击者就能利用该漏洞进行盗取用户隐私信息，篡改用户数据等行为。

具体来说，恶意网站可以伪造HTTP的GET和POST请求，在HTML的img,iframe,frame等标签中可以发起GET请求，而form标签可以发起POST请求。前者相对简单，后者需要用到JavaScript的技术。因为Elgg只使用POST，所以实验中会涉及到HTTP POST请求和JavaScript的使用。

## CSRF Ataack using GET Request

这个任务要求通过CSRF手段伪造GET请求，通过引诱对方打开恶意网站来实现GET请求的发送，并借助受害者浏览器保存的Cookie进行伪装。

假设矮穷矬Boby想和白富美Alice成为好友，但Alice不同意，于是Boby灵机一动，决定通过CSRF的手段强制让Alice加自己好友。具体来说Boby给Alice发了一个URL，Alice对此表示好奇，就点开了这个URL，此时她就会在不知情的情况下把Boby添加为好友了。

要实现这样的功能首先要了解Elgg中添加好友的机制：

![CSRF1](https://raw.githubusercontent.com/familyld/CSRF-Attack/master/graph/image8.png)

打开两个浏览器应用，分别登录Alice和Boby的账户。要添加对方为好友可以先在导航栏找到More，然后点击Members选项来到Members页面，这里可以看到全部用户：

![CSRF1](https://raw.githubusercontent.com/familyld/CSRF-Attack/master/graph/image9.png)

找到Boby后，点击进入Boby的Profile页面，可以看到头像下方有一个Add friend按钮，点击后就会自动添加Boby为好友了，我们需要观察这个动作是如何实现的。

![CSRF1](https://raw.githubusercontent.com/familyld/CSRF-Attack/master/graph/image10.png)

要了解Add friend触发什么请求我们可以使用LiveHttpHeader这个插件，SEEDUbuntu已经预装好，在Firefox菜单栏的Tools选项卡中选择LiveHttpHeader就可以打开，然后勾选Capture，即可进行抓包：

![CSRF1](https://raw.githubusercontent.com/familyld/CSRF-Attack/master/graph/image11.png)

点击Add friend后，查看Live HTTP headers的抓包结果，可以清晰地看到这是一个GET请求，高亮的部分就是发出的链接。于是，我们需要做的就是把这个GET请求放在自己做的恶意网站中，并且把网站URL发给Alice，Alice点击后，这个跨站点的GET请求会被触发，浏览器识别出这是向Elgg发送的GET请求，于是自动把Cookie和Session ID附加上去，这时Elgg的服务器就会认为这个GET请求是Alice自己发出的，把Boby添加为Alice的朋友，Boby龌龊的计划就成功了。

### 具体实现

在/var/www/CSRF/Attacker/路径下创建task1.html：

![CSRF1](https://raw.githubusercontent.com/familyld/CSRF-Attack/master/graph/image12.png)

编写一个简单的HTML网页，借助img标签的src属性来发起跨站请求，把刚刚抓包得到的链接填进去：

![CSRF1](https://raw.githubusercontent.com/familyld/CSRF-Attack/master/graph/image13.png)

然后把这个网站的URL(http://www.csrflabattacker.com/task1.html)发给Alice，Alice点击后img标签自动触发GET请求，于是刷新一下就会发现Boby已经变成Alice的friend了：

![CSRF1](https://raw.githubusercontent.com/familyld/CSRF-Attack/master/graph/image14.png)

## CSRF Attack using POST Request

这个任务要求通过CSRF手段伪造POST请求，通过引诱对方打开恶意网站来实现POST请求的发送，并借助受害者浏览器保存的Cookie进行伪装。

在这个任务中依然是Alice和Boby两位同学，但变成了Alice有求于Boby。Alice是SEED计划的发起人，她希望Boby可以在自己的profile中写上”I support SEED project!”，然而Boby十分讨厌做实验，不愿意写。于是黑化的Alice决定以其人之道还治其人之身，使用CSRF攻击手段强制修改Boby的profile。

类似Task1，我们需要用Live HTTP Header来抓包，只是这次需要抓包的对象是提交Profile改动这个动作，显然这是一个提交表单发起的POST请求。

![CSRF2](https://raw.githubusercontent.com/familyld/CSRF-Attack/master/graph/image15.png)

首先进入Edit profile界面，然后填写好各种信息，最后点击Save按钮提交：

![CSRF2](https://raw.githubusercontent.com/familyld/CSRF-Attack/master/graph/image16.png)

此时观测抓包状况，可以看到这个POST包，把内容提取出来：

![CSRF2](https://raw.githubusercontent.com/familyld/CSRF-Attack/master/graph/image17.png)

正是刚刚填写的表单内容，其中最重要的就是用来标识用户的guid字段。并且注意到每一个表单项后面都会跟着对应的accesslevel字段，且字段值是2，这些字段都是表单中隐藏的项，值是默认的，在伪造POST请求时要注意把这些字段也加入进去。
P.S.:后台处理表单提交的是/var/www/CSRF/elgg/actions/profile路径下的edit.php脚本文件。

基本代码在instruction中的Figure 1已经给出来了，思路是利用JavaScript来实现跨站POST请求。在Boy打开恶意网站URL时，网站的script会执行，而我们在script标签中伪造的表单会被提交，触发跨站POST请求。和Task1类似，浏览器会识别出这是向Elgg发送的请求，于是自动把Cookie和Session ID附加上去，这时Elgg的服务器就会认为这个POST请求是Boby自己发出的，修改了Boby的Profile，Alice的计划就成功了。

### 具体实现

同Task1类似，编写一个task2.html网页，在script标签中进行函数定义以及执行方式，script中没有被定义在函数内的代码会在网页加载时被执行。
在csrf_hack函数中填写表单的各个字段，URL对应edit.php脚本文件的路径：

![CSRF2](https://raw.githubusercontent.com/familyld/CSRF-Attack/master/graph/image18.png)

在post函数中执行提交表单的操作，这是首先是在网站中动态生成一个表单对象，然后把表单内容设置为之前填写好的字段，触发动作设置为edit.php脚本文件对应的URL，方法自然是post。设置好并放入网站后，就可以使用submit方法提交表单了：

![CSRF2](https://raw.githubusercontent.com/familyld/CSRF-Attack/master/graph/image19.png)

最后，在函数外部定义函数执行的方式，这里定义为加载页面时执行csrf_hack函数就可以了：

![CSRF2](https://raw.githubusercontent.com/familyld/CSRF-Attack/master/graph/image20.png)

诱骗Boby点击Task2的URL(([http://www.csrflabattacker.com/task2.html](http://www.csrflabattacker.com/task2.html))后，再次浏览他的profile页：

![CSRF2](https://raw.githubusercontent.com/familyld/CSRF-Attack/master/graph/image21.png)

果然可怜的Boby已经被卖了~

### Hint

- 完成添加好友的操作需要知道对方的id（guid），如何获取到对方的id（guid）？

> Alice可以通过一个类似Taks1这样的CSRF攻击来获取Boby用户的guid。不妨回顾一下Task1，当我们点击一个用户Profile页的Add friend按钮时，网站会发起一个GET请求：

![CSRF2](https://raw.githubusercontent.com/familyld/CSRF-Attack/master/graph/image22.png)

> 使用抓包工具可以看到这条链接中的friend字段后面跟着一个数值，这个数值就是朋友的guid。所以Alice要知道Boby的guid只需要添加他做好友，并且执行添加动作时抓包下来看看就可以了。

- 如果需要让任意访问恶意网站的人都被修改掉其个人简介。但是，你又不能事先知道谁会访问这个恶意网站。这能还能通过CSRF来完成修改个人简介的操作么？HOW&WHY？

> 能~ 假设CSRF攻击成功的前提是拥有用户的guid，我们可以先通过使用CSRF伪造GET请求来获取用户的guid，然后在伪造POST请求时赋值给guid字段就行了。无论是谁，只要没退出Elgg系统，那么访问了恶意网站的同时就会被篡改profile信息。

## Implementing a countermeasure(对策) for Elgg

事实上Elgg是有反CSRF的措施的，只是前面为了体现攻击效果所以关闭了而已。CSRF是很容易防御的，下面是两种常见的防御方式：

####Secret-token方法：

> 在HTTP请求中附加一些secret token，这些secret token只能在正常用户的浏览器上产生，从恶意网站发出的请求是无法获取这些secret token的。这样就能轻易地分辨出请求是合法还是非法的了。

####Referrer header方法：

> 可以通过验证请求来源页面的Referer字段来分辨请求的合法性。但由于隐私考虑，这个Referer经常被客户端过滤；

![CSRF3](https://raw.githubusercontent.com/familyld/CSRF-Attack/master/graph/image23.png)

抓包中可以看到Referer字段，Firefox没有过滤掉这个字段，可以看到从csrflabelgg和从csrflabattacker发起的请求是完全不同的。

Elgg使用的是Secret-token方法，通过在请求中嵌入__elgg_ts和__elgg_token两个参数来保证请求的合法性。在POST请求中它们被添加进HTTP消息的body里面，前面task2抓包截图中可以看到；在GET请求中它们会直接附在URL后面，在task1的链接中可以看到。

这两个参数举例来说一个是timestamp，一个是秘密令牌。在所有需要进行表单提交操作的页面，这两个参数都会作为隐藏的表单元素被一起提交：

![CSRF3](https://raw.githubusercontent.com/familyld/CSRF-Attack/master/graph/image24.png)

它们是通过/var/www/CSRF/elgg/views/default/input/路径下的securitytoken.php脚本文件生成并添加到网页页面的。

![CSRF3](https://raw.githubusercontent.com/familyld/CSRF-Attack/master/graph/image25.png)

上面这段代码就展示了它们是如何被动态地添加到网页中的。此外，还可以通过JavaScript的elgg.security.token.__elgg_ts和elgg.security.token.__elgg_token进行访问。

Elgg中的安全令牌是通过对secret参数(从数据库中抽取)，timestamp，user session ID进行md5哈希生成的，代码如下：

![CSRF3](https://raw.githubusercontent.com/familyld/CSRF-Attack/master/graph/image26.png)

这段PHP代码中用到的session_id函数不仅可以获取当前session的ID。也可以用来设定当前session的ID。我们可以把__elgg_session设定一个随机生成的session ID而非公开的用户session ID：

![CSRF3](https://raw.githubusercontent.com/familyld/CSRF-Attack/master/graph/image27.png)

在用户发起请求时，服务器会调用validate_action_token函数进行合法性验证：

![CSRF3](https://raw.githubusercontent.com/familyld/CSRF-Attack/master/graph/image28.png)

可以看到如果验证失败或者没有提供安全令牌，请求会无效，用户被重定向到错误页面。

### Turn on countermeasure

要打开Elgg防御CSRF攻击的功能，需要修改/var/www/CSRF/elgg/engine/lib/目录下actions.php脚本文件的action_gatekeeper函数，原来是默认不进行验证，任何操作都直接返回true表示通过验证，我们把return true语句注释掉，这样验证函数就能够正常执行了：

![CSRF3](https://raw.githubusercontent.com/familyld/CSRF-Attack/master/graph/image29.png)

### 再次执行CSRF攻击

Task1攻击抓包下来，启用前后只有Date和Keep-Alive字段是不同的，其他没有差别，但从效果上来看，启用后确实无法令Alice添加Boby为好友。而Task2攻击抓包下来差别就比较大了，未启用防御措施之前直接发送POST请求并成功执行返回到用户Profile界面；启用防御措施后，点击URL会返回如下链接：

![CSRF3](https://raw.githubusercontent.com/familyld/CSRF-Attack/master/graph/image30.png)

并且后台会继续一直发POST请求，Keep-Alive字段的max值会逐渐减少。

## 为什么恶意网站不能在起get或post请求中加入secret token？什么机制保证了这些secret token不被恶意第三方知晓呢？

其实这两个问题的答案是一致的，secret token是用户在网站A执行动作时在A的**后端服务器**随机生成并置入发送的请求中的。而CSRF的原理仅仅是伪造请求，并借助用户浏览器附加cookie的特性实现伪装，如果secret token不储存在cookie中，攻击者就无法凭空捏造出这个token(由于同源策略的限制，攻击者无法使用JavaScript获取其他域的token值)，而要暴力猜出token又几乎是不可能的(生成token足够随机)，所以CSRF在这种状况下就无机可乘了。
