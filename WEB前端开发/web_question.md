[TOC]



# 基本概念（所有课件中涉及的，不限于总结）

### MEAN

MongoDB是⼀个使用JSON⻛格存储的数据库，非常适合 javascript。(JSON是JS数据格式) 

ExpressJS是⼀个Web应用框架，提供有帮助的组件和模块帮助建立⼀个网站应用。 

AngularJS是⼀个前端MVC框架。 

Node.js是⼀个并发异步事件驱动的Javascript服务器后端开发平台。

\1. IP：**Internet Protocol**（网络互联协议），是为**计算机网络相互连接进行通信**而设计的网路层协议， 

\2. TCP：**Transmission Control Protocol**（传输控制协议），是一种**面向连接的、可靠地、基于字节流的传输层通信协议**。 

\3. UDP：**User Datagram Protocol**（用户数据报协议），为应用程序提供了一种**无需建立连接就可以发送封装的****IP****数据报**的方法（如QQ）

\4. DNS：**Domain Name System**（域名系统），是一种用于**域名和****IP****地址相互转换**的**分布式数据库**

\5. URI：**Uniform Resource Identification**（统一资源标识符），是一个**用于标识某一互联网资源名称**的字符串，允许资源位于Internet的任何位置

\6. URL：**Uniform Resource Locator**（统一资源定位符），是Internet的**万维网服务程序上用于指定信息位置**的表示方法，**描述了资源的位置**

\7. URN：**Uniform Resource Name**（统一资源名称），是带有名字的Internet资源

\8. WWW：**World Wide Web**（万维网），是**存储在****Internet****计算机中，数据量巨大的文档的集合**，也是**一组软件和协议的集合**，WWW服务器**通过****HTML****把信息组织成超文本，利用链接从一个站点跳到另一个站点**

\9. HTML：**Hypertext Markup Language**（超文本标记语言），**用于描述网页文档，是创建网页和****Web****应用**的标准标记语言

\10. XHTML：**Extensible Hypertext Markup Language**（可扩展超文本标记语言），是一种**基于****XML**的标记语言，**表现方式**和HTML类似，但**语法更加严格**。是SGML的子集。

\11. XML：**Extensible Markup Language**（可扩展标记语言），是一种**用于标记电子文件使其更具有结构性**的标记语言

\12. RWD：**Responsive Web Design**（响应式Web设计），是一种**新的网站设计模式**，以此构建的网站可以**自动适应不同的访问设备**。

\13. CSS：**Cascading Style Sheet**（层叠样式表），是一种**用来表现****HTML****或XML****等文件样式**的计算机语言，可以**静态**或配合各种脚本语言**动态修饰**页面元素。

\14. JS：**JavaScript**，是一种具有**函数优先的轻量级，解释型或即时编译型**的编程语言，主要用于向HTML页面**添加交互行为**

\15. DOM：**Document Object Model**（文档对象模型），是W3C组织推荐的**处理可扩展置标语言的标准编程接口**，是一种**与平台和语言无关的****API**，**定义了操作文档对象的标准**

\16. BOM：**Browser Object Model**（浏览器对象模型），是用于**描述各种对象与对象之间层次关系**的模型，**提供了独立于内容的、可以与浏览器窗口进行互动的对象结构**

\17. CS：**Client-Server**（客户端-服务器架构），通常采取两层结构，服务器负责数据的管理，客户机负责完成与用户的交互任务

\18. BS：**Browser-Server**（浏览器-服务器架构），极少数事务逻辑在前端实现，主要事务逻辑在服务端实现

\19. AJAX：**Asynchronous JavaScript And XML**（异步JavaScript和XML），是一种**支持异步请求，用于创建交互式网页应用**的网页开发技术

\20. RIA：**Rich Internet Application**（丰富互联网程序），是**具有高度互动性、丰富用户体验以及功能强大的客户端**的Internet程序

\21. HTTP：**Hypertext Transfer Protocol**（超文本传输协议），是一种**运行于****TCP****之上的详细规定了浏览器和万维网服务器之间相互通信**的规则，**支持**浏览器和Web服务器之间的**通信**

\22. HTTPS：**HTTP over Secure Socket Layer**（基于安全套接字层的HTTP协议），是**以安全为目标**的HTTP通道，在HTTP的基础上**通过传输加密和身份认证**保证了传输过程的安全性

\23. RTT：**Round-Trip Time**（往返时延），是计算机网络中一个重要的**性能指标**，用于**表示从发送端发送数据开始到发送端收到来自接收端的确认总共经历的时延**

\24. SEO：**Search Engine Optimization**（搜索引擎优化），是一种利用搜索引擎的规则**提高网站在有关搜索引擎内的自然排名**的方式

\25. WPO：**Web Performance Optimization**（Web性能优化），目的是**提高页面的加载速度**，以提高用户体验

\26. gRPC：**google Remote Procedure Call**（google远程过程调用系统），是一款由**google****开发的语言中立、平台中立、开源**的RPC系统

\27. RPC：**Remote Procedure Call**（远程过程调用协议），一种**通过网络从远程计算机程序上请求服务**，而不需要了解底层网络技术的协议

\28. REST：**Representational State Transfer**（表述性状态传递），一种**针对网络应用的设计和开发方式**，可以**降低开发的复杂性，提高系统的可伸缩性**的软件架构风格

\29. CORS：**Cross-origin Resource Sharing**（跨域资源共享），是一种**允许浏览器向跨源服务器发出****XMLHttpRequest****请求**，从而克服了AJAX只能同源使用的机制

\30. SVG：**Scalable Vector Graph**（可缩放矢量图形），是基于XML，**用于描述二维矢量图形**的一种图形格式

\31. WSDL：**Web Service Description Language**（Web服务描述语言），是**用于描述****Web****服务的公共接口**，**基于****XML****的关于如何与Web****服务通讯和使用**的服务描述语言

\32. XSS：**Cross Site Scripting**（跨站脚本攻击），是指向Web页面**插入恶意脚本代码**，以**利用网站漏洞**从用户处**恶意盗取信息**的攻击

\33. GraphQL：**Query Language for API**，是一门**用于****API****和运行时的专用查询语言**

\34. RR：**Resource Records**，是存放在DNS的name servers中的配置文件上的**分布式数据记录**

\35. CDS: **Content Distribution Service**，通过此服务让客户可以访问最接近本地缓存服务器的缓存内容，减少延迟和负载。

\36. SPA：**Single Page Application**，一种特殊的Web应用，加载一个Html页面并在交互时动态更新该页面。

\37. CDN：**Content Delivery NetWork****，**内容分发网络

# 简答题

### web server

Web Server基本原理

(1)   本质是：接收数据=>Http解析=>逻辑处理=>Http封包=>发送数据

(2)   基本流程如下

①   与浏览器建立TCP连接

②   浏览器将用户事件按照Http协议格式打包为数据包，确认对端可写后推入Internet，包经过网络传至Server

③   Server根据Http协议格式解析数据包

④   解析得到用户事件后处理事件逻辑

⑤   按照Http协议格式将结果数据打包，确认对端可写后推入Internet，包经过网络传至浏览器

⑥   浏览器根据Http协议格式解析数据包，根据数据格式展示相应结果

### 什么是持久连接？在http1.0+和1.1中是如何描述的？

持久连接：http1.1版本中：当发送请求并响应之后，服务器和客户端浏览器之间依然保持连接，文件中的所有对象都可在相同的TCP连接上传送。

在使用HTTP/1.0的情况下，如果打开一个包含一个HTML文件和10个内联图象对象的网页时，HTTP就要建立11次TCP连接才能把文件从服务机传送到客户机

HTTP/1.1都有非持续连接(non-persistent connection)和持续连接(persistent connection)功能。HTTP/1.1的默认设置是持续连接。

HTTP/1.0的默认设置是非持续连接

* 带流水线
	* HTTP1.1 默认
	* web页面所引用的每个对象(上例中的10个图像)都经历1个RTT的延迟
* 不带流水线
	* 允许在客户机接收到服务机的消息响应之前发送多个消息请求

### HTTPS vs HTTP

1. https协议需要到ca申请证书，一般免费证书很少，需要交费。
2. http是超文本传输协议，信息是明文传输，https 则是具有安全性的ssl加密传输协议。
3. http和https使用的是完全不同的连接方式，用的端口也不一样，前者是80，后者是443。
4. http的连接很简单，是无状态的；HTTPS协议是由SSL+HTTP协议构建的可进行加密传输、身份认证的网络协议，比http协议安全。

### HTTP常用请求头、状态码

* Request：GET、POST、HEAD、PUT、DELETE 

	* 常用请求头：accept、host、from、user-agent、referer

* Response： 

	1xx Informational 2xx Success 3xx Redirection 4xx Client Error 5xx Server Error

	* 200 
	* 400 bad request客户端有语法错、不能被服务器理解
	* 401 unauthorized请求未经授权，与www-zuthenticate⼀起使用 
	* 403 forbidden 服务器收到请求，但拒绝服务 
	* 404 not found 请求资源不存在 
	* 500 internal server error 服务器发送不可预期的错误 
	* 503 server unavailable 服务器当前不能处理客户端请求，⼀段时间后可恢复正常

### WEB的发展

web的发展

* web1.0——web of documents
* web2.0——web of persons
	* 架构在知识上的环境，人与人之间交互而产生出的内容
	* Facebook、wiki、微博...
	* 局限性：过度饱和、概念偏差、实践、互动模式、开放
* web3.0——web of data
	* 语义化、3D、AI、去中心化

### 从SEO的角度来解释对http状态码301,302,404的理解？

301是一种永久性的重定向，搜索引擎会删除原页面，收录重定向的页面，并转移权重

302是一种临时性的重定向，大部分搜索引擎把它作为内部的重定向，不会缓存重定向的结果

404页面不存在或链接错误，搜索引擎放弃对该链接的索引

 ### 什么时候使用301重定向

1：网站更换域名时，通过301永久重定向将旧域名重定向⾄新域名，挽回流量损失和 SEO。 

2：当出于需要删除网站中的某些目录时，比如我要删除我博客下的博客导航，这时就可以用301永久重定向到网站首页。 

3：如果你有多个闲置域名时需要指向同⼀网站时，通过301永久重定向可以实现。 

4：你打算实现网址规范化

### 如何理解html结构的语义化

* 在HTML 5出来之前，我们用`div`来表示页面章节，但是这些`div`都没有实际意义

* 语义化是HTML5的新特性，根据结构化的内容选择合适的标签

* 为什么？

	* **有利于SEO**：爬虫依赖标签来确定关键字的权重，因此可以和搜索引擎建立良好的沟通，帮助爬虫抓取更多的有效信息
	* **开发维护体验好**：代码更具有可读性
	* **用户体验好**：例如title、alt可以用于解释名称或者解释图片信息
	* **更好的可访问性，方便任何设备对代码进行解析**：屏幕阅读器、盲人阅读器、移动设备等
	* **代码结构:** 使页面没有css的情况下，也能够呈现出很好的内容结构

* 语义元素

	![image-20210106175628755](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210106175628755.png)

### 为什么使用css？

* 可以从文档中分离出文档的样式，更便于维护和避免重复
* 可以使用相同的内容加以不同的样式实现不同的目的
* 灵活、简短、清晰、基础格式化工具、简单的多文件管理、使用类选择器节省时间

### CSS规则的优先级算法是如何计算的

[1位重要标志位] > [4位特殊性标志] > 声明先后顺序

* 内联样式表的权值最高 1000；

* ID 选择器的权值为 100

* Class 类选择器的权值为 10

* HTML 标签选择器的权值为 1 

* !important的优先级是最高的

统计选择子中以上选择器的个数，得出权值，比较，如果一样大，则采用就近原则（采用后面的）

![image-20210106191642653](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210106191642653.png)

### 响应式网页设计的优劣和主要手段？

* 优势：
	* 网站可用性强，与移动优先设计以及内容策略可以融合
	* 简化服务器端
	* 更容易维护
	* 只提供一个入口给搜索引擎
	* 能够支持未知设备
* 劣势
	* 工作量大
	* 代码多，冗余
	* 限制应用的复杂性
	* 一定程度上改变了网站原有的布局结构，会出现用户混淆
* 主要手段
	* 灵活的网页布局，根据浏览器窗口大小成比例缩放，而不设为固定宽度，用em
	* 灵活、比例适中的图像和视听媒体。
		* H5`<picture>`元素，配置多个`<source>`
		* 图片处理：Liquid Image部分图片始终展示，部分图片截取展示
	* 使用CSS3的媒体查询（视窗的宽高、设备宽高、朝向、DPI）

### 什么是css盒模型

内容(content)、填充(padding)、边框(border)、边界(margin)， CSS盒子模式都具备这些属性。

<img src="C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20210104191450695.png" alt="image-20210104191450695" style="zoom: 50%;" />



### css3新特性

* 边框
	* `border-radius`
	* `box-shadow`
* 背景
	* `background-size`
	* `background-origin`规定 background-position 属性相对于什么位置来定位。
	* `background-clip`：规定背景的绘制区域。
* 渐变
	* `linear-gradient()`：线性渐变。
	* `radial-gradient()`：径向渐变。
* 转换 `transform`
* 渐变 `transition`
* 动画 `animation`
* **媒体查询**

### BFC和IFC

* BFC（block formatting context）
	* 块级格式化上下文
	* 内部box在垂直方向放置
	* BFC就是页面上的⼀个隔离的独立容器，容器里面的子元素不会影响到外面的元素。
	* 反之也如此。 计算BFC的高度时，浮动元素也参与计算
* IFC（inline formatting context）
	* 多个内联元素排列在一起时形成一个IFC，之间不能穿插块级元素
	* 一个IFC内的元素都是水平排列的
	* 横向的margin、border、padding属性对于这些元素都有效
	* 垂直方向可以调整对齐方式

### 移动优先

* 优先显示最重要的内容和功能，如果空间允许，再逐步加入次要内容与功能
* 好处
	* 只要移动端做的好，即使用户使用的是旧版本浏览器、没有 Javascript或者关闭了Javascript的浏览器，或为视力残障人士设计 的读屏浏览器，也能看到⼀个拥有基本功能的网站。 
	* 移动优先是渐进将强理念的良好范例，所有用户都能访问核心内容和功能。不存在不能访问的情况

### CSS3媒体查询

有条件地检测用户显示屏的各个方面，然后根据这些条件有选择地加载出样式表，并提供最合适的布局、排版和图形。

* 视窗的宽高、
* 设备宽高、
* 朝向、
* DPI分辨率

### 表单验证是什么？有哪几种方式并比较。

* 原生JS
* HTML5
	* 在`<input>`中加入required属性，pattern可以自定义正则表达式，form不能有novalidate，否则忽略表单验证

### 为什么不建议table布局？（课件上没，估计删了）

table比其它html标记占更多的字节。(造成下载时间延迟,占用服务器更多流量资源)

table会阻挡浏览器渲染引擎的渲染顺序。(会延迟页面的生成速度,让用户等待更久的时间)

table里显示图片时需要你把单个、有逻辑性的图片切成多个图。(增加设计的复杂度,增加页面加载时间,增加http会话数)

在某些浏览器中,table里的文字的拷贝会出现问题。(会让用户不悦)

table会影响其内部的某些布局属性的生效(比如<td>里的元素的height:100%) (限制页面设计的自由性)

一旦学了CSS的知识,你会发现使用table做页面布局会变得更麻烦。(先花时间学一些CSS知识,会省去你以后大量的时间)

table对于页面布局来说,从语义上看是不正确的。(它描述的是表现,而不是内容)

table代码会让阅读者抓狂。(不但无法利用CSS,而且会不知所云,尤其在进行页面改版或内容抽取的时候)

table一旦设计完成就变成死的,很难通过CSS让它展现新的面貌

### 说明session和cookie的区别

* cookie数据存放在客户的浏览器上，session数据放在服务器上。
* 单个cookie保存的数据<=4KB，对于session来说并没有上限
* cookie中只能保管ASCII字符串，session中能够存储任何类型的数据，包括且不限于string，integer，list，map等
* cookie不是很安全，别人可以分析存放在本地的COOKIE并进行COOKIE欺骗,考虑到安全应当使用session
* session会在一定时间内保存在服务器上。当访问增多，会比较占用你服务器的性能,考虑到减轻服务器性能方面，应当使用cookie
* **cookie机制采用的是在客户端保持状态的方案**。**它是在用户端的会话状态的存贮机制**，他需要用户打开客户端的cookie支持。**cookie的作用就是为了解决HTTP协议无状态的缺陷所作的努力**.而**session机制采用的是一种在客户端与服务器之间保持状态的解决方案**。

### JavaScript注册event handler有哪几种方式？

* DOM0级事件，就是直接通过 onclick 等方式实现相应的事件

	* 标签内直接内onclick

		`<input id="myButton" type="button" value="Click Me" onclick="alert('Hello1');" >`

	* 在 JS 中 使用onclick = function(){}

		```js
		document.getElementById("myButton").onclick = function () {
		    alert('Hello2');
		}
		```

	* DOM0 级添加事件时，后面的事件会覆盖前面的事件，而 DOM2级则不会，多个事件都会执行

* DOM2 级

	* `target.addEventListener(type, listener[, useCapture]);`
	* `target.removeEventListener(type, listener[, useCapture]);`
		* **type**：事件类型，如'click'、'mouseover'、'mouseout'，在事件名前不加'on'
		* **listener**：事件处理方法
		* **useCapture**：布尔参数，不传该参数时默认是 false，表示在事件冒泡阶段处理，如果是 true，则表示在捕获阶段调用事件处理程序
	
* 兼容性处理

	```js
	var x = document.getElementById("myBtn");
	if (x.addEventListener) {
	// For all major browsers, except IE 8 and earlier
	    x.addEventListener("click", myFunction);
	} else if (x.attachEvent) {// For IE 8 and earlier versions
	    x.attachEvent("onclick", myFunction);
	}
	```

### 闭包是什么，使用环境？有什么缺点？

函数可以访问它被创建时的上下文环境，称为闭包

使用环境：

* 实现私有成员
* 保护命名空间
* 避免污染全局变量
* 变量需要长期驻存在内存

缺点：

* 变量污染

* 内存泄漏

### 什么是Vanilla JS，有什么作用？

vanillaJS是世界上最轻量的框架，没有之一！使用vanillaJS的部署策略，能够让在用户访问访问你的网站之前就就从内存中读取vanillaJS的资源。

AKA 原生JS

### 跨域问题是什么，有什么解决办法

 跨域是指跨域名的访问，有三种情况：

- 域名不同的跨域。
- 域名相同、端口不同的跨域。
- 二级域名不同的跨域。

### 什么是ajax

* ajax请求

	* asynchronous JavaScript and XML
	* 是一种使用JavaScript的特殊方式
	* 在后台从服务端下载数据
	* 允许动态更新页面
	* 避免了“点击-等待-刷新”的模式

* ajax优缺点

	* **优点**
		* 更好的交互与响应
		* 对用户更加友好
		* 减少因为部分渲染而与web服务器的连接
		* 只加载更新页面所需的数据，而不是每次都刷新整个页面，进而节省带宽

	* **缺点**
		* 回退和刷新按钮没用
		* 网摘页面会变得无用
		* 需要js，可能有js兼容性问题
		* 网络延迟可能造成问题
		* 通过AJAX加载的数据不会被搜索引擎建立索引，因此可能导致SEO无法很好使用
	* 限制：
		* 本地的网站不可以使用XMLHttpRequest

* AJAX优化

	* 数据优化：
		* 压缩数据
		* key、value对尽量小
	* 代码优化：
		* 内联sql
		* 存储过程

# 问答题

###  html5新功能并举例？

* 新元素
	* 语义化layout，如header、footer、nav、aside
* 新属性
	* Paved Cow Path 验证方式关键值 `<input type=email required>`
* 完全支持css3
* video和audio
	* audio、video
* 2D/3D绘制
	* Canvas像素级操作，放大缩小会变形
	* SVG矢量图，伸缩时细节不会发生变化，基于XML
* 本地存储
	* 比cookie更好
	* localStorage——长久保存
	* sessionStorage——临时，关闭窗口后删除
	* manifast文件告知浏览器被缓存和不缓存的内容
		* CACHE MANIFEST - 在此标题下列出的文件将在首次下载后进行缓存 
		* NETWORK - 在此标题下列出的文件需要与服务器的连接，且不会被缓存 
		* FALLBACK - 在此标题下列出的文件规定当页面⽆法访问时的回退页面（比如 404 页面）
	* 当下列情况发送时，缓存刷新
		* 用户清空浏览器缓存
		* manifast文件发生修改
		* 程序更新应用缓存
* 本地sql应用
* web应用
	* WebSocket

### bigpipe的原理和优缺点？

把页面分为很多模块— —称之为pagelet，比如头部、内容区域、边栏等等

Bigpipe允许客户端跟服务器建立一条管道，内容可以源源不断地输送过来。首先输送的是html+css，接下来会传输很多js对象，然后有专门的js代码来把对象所代表的内容渲染到页面上

### 阐述优化思路

1. 减少DNS解析

	如DNS预解析

2. 减少TCP连接时间

	如TCP预连接、使用持久连接等

3. 尽可能少进行HTTP重定向

4. 使用CDN，减少网络延迟

5. 取消不必要的资源

6. cache资源，客户端缓存、服务端缓存

7. 压缩资源

	如CSS精灵

8. 压缩请求头，减少RTT

9. 流水线，并行地请求和响应

### 针对HTTP1.x和HTTP2的优化

HTTP1.x

* 利用http pipeline
* 域名分区，考虑将资源分散到多个来源
* 打包资源，如CSS精灵
* 嵌入小资源，考虑直接在父文档中嵌入小资源，以减少请求数量

HTTP2.0

* 每个来源使用一个连接，HTTP2.0通过将一个TCP连接吞吐量最大化来提升性能，这样以来使用多个连接反而成为反模式
* 去掉不必要的文件合并和图片拼接，HTTP2.0下很多小资源可以并行发送
* 利用服务器推送

### CSS sprite 是什么？ 谈谈这个技术的优缺点

css精灵，是一种网页图片应用处理方式。它允许你将一个页面涉及到的所有零星图片都包含到一张大图中去，这样一来，当访问该页面时，载入的图片就不会像以前那样一幅一幅地慢慢显示出来了。对于当前网络流行的速度而言，不高于200KB的单张图片的所需载入时间基本是差不多的，所以无需顾忌这个问题。

* 优点：
	* 减少网页的http请求，从而大大提高了页面的性能
	* 能减少图片的字节，曾经多次比较过，把3张图片合并成1张图片的字节总是小于这3张图片的字节总和。
* 缺点：
	* 开发麻烦，需要合成图片
	* 维护麻烦，换一个图需要重新合成
	* 图片合并时，需要留足空间

### js的严格模式和普通模式有什么不同

为什么

* 消除Javascript语法的⼀些不合理、不严谨之处，减少⼀些怪异行为; 
* 消除代码运行的⼀些不安全之处，保证代码运行的安全； 
* 提高编译器效率，增加运行速度； 
* 为未来新版本的Javascript做好铺垫。

内容

* 不允许使用未声明的变量
* 函数参数不能有同名属性
* 不允许删除变量或对象
* 不允许删除函数
* 不允许使用前缀0的八进制
* 不可对只读属性赋值
* 禁止this指向全局变量
* 增加保留字

### nodejs的优缺点，适用场景，什么是回调地狱，如何避免？

* 优点：
	* 非阻塞异步IO
	* V8 JavaScript 引擎
	* 单线程，event loop
	* 超过一百万的modules
	* 跨平台
	* 与前端语言一致
	* 活跃的社区
* 缺点：
	
	* 不适合CPU密集型应用
* 使用场景：
	* 网站、RestAPI、http proxy、反向代理、前端构建工具、打包工具、命令行工具等
	* 快速开发各式各样的能极大提升开发效率的神器

* 回调地狱

	* 异步

		![image-20210106214213386](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210106214213386.png)

	* 解决方案：`async`/`await`语法。