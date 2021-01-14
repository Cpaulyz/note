## 基础知识

### Internet

* IP

	* 电脑之间数据传输的简单协议
	* 发送数据包的基础系统
		* 子网、无差别域内路由CIDR、NAT
	* IPV4 32位，IPV6 128位
	* TCP
		* 多路通讯，通过端口号复用IP地址
		* 80：服务器，25：email，22：ssh，21：ftp

* URI、URL、URN

	* https://www.jianshu.com/p/09ac6fc0f8cb
	* URI is a superset of both URL and URN

* DNS

	* 域名系统

	* 将IP地址与域名进行转换

	* 在之前使用hosts文件进行解析

	* 域名有层次性，易于记录

		![image-20210104152356241](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20210104152356241.png)

	* 每个域的主服务器负责更新，从服务器保存副本

	* 根域名服务器有13个

	* 迭代/递归式

		* 迭代：主机服务器指向另一台服务器
		* 递归：主机服务器帮忙请求另一台服务器

	* ![image-20210104152655441](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20210104152655441.png)

	* cache：缓存加速

* Http1.1、/2、/3协议

	* http1.1

		* 默认使⽤keep-alive连接 
		* http管线化：客户端⽆需在发送后续 HTTP 请求之前等待服务器响应请 求，但是会导致线头阻塞（服务器按照接收到的请求顺序进行响应）

	* http /2

		* **新的二进制格式**（Binary  Format），HTTP1.x的解析是基于文本。基于文本协议的格式解析存在天然缺陷，文本的表现形式有多样性，要做到健壮性考虑的场景必然很多，二进制则不同，只认0和1的组合。基于这种考虑HTTP2.0的协议解析决定采用二进制格式，实现方便且健壮。

			**多路复用**（MultiPlexing），即连接共享，即每一个request都是是用作连接共享机制的。一个request对应一个id，这样一个连接上可以有多个request，每个连接的request可以随机的混杂在一起，接收方可以根据request的 id将request再归属到各自不同的服务端请求里面。

			**header压缩**，如上文中所言，对前面提到过HTTP1.x的header带有大量信息，而且每次都要重复发送，HTTP2.0使用encoder来减少需要传输的header大小，通讯双方各自cache一份header fields表，既避免了重复header的传输，又减小了需要传输的大小。

			**服务端推送**（server push），同SPDY一样，HTTP2.0也具有server push功能。

	* http /3

		* 传输层是UDP协议
		* 在UDP协议之上，新增了QUIC协议，代替TCP协议中关于可靠、流量控制的部分

	* https

		* HTTPS URLs begin with "https://" and use port 443 by default, whereas HTTP URLs begin with "http://" and use port 80 by default.

	* ![image-20210104153721929](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20210104153721929.png)

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

### 网络编程相关--

* 网络机器人

### WEB

* web的发展
	* web1.0——web of documents
	* web2.0——web of persons
		* 架构在知识上的环境，人与人之间交互而产生出的内容，所有人既是信息产生者又是信息客户
		* Facebook、wiki、微博...
		* 局限性：过度饱和、概念偏差、实践、互动模式、开放
	* web3.0——web of data
		* 语义化、3D、AI、去中心化
		* 智能语义程序介入，更有针对性地发送信息和获取信息
* MEAN
	* MongoDB+Express+AngularJS+NodeJS

## HTML

* 结构、表现、行为

	* `<!DOCTYPPE html>`
	* `<head>`
	* `<body>`

* 基本语法、常用标记

* html语义化

	* 根据结构化的内容选择合适的标签
	* 为什么？
		* 有利于SEO
		* 开发维护体验好
		* 用户体验好
		* 更好的可访问性，方便任何设备对代码进行解析

	<img src="C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20210104162621834.png" alt="image-20210104162621834" style="zoom:67%;" />

* html5新特性

	* Paved Cow Path 验证方式关键值 `<input type=email required>`
	* 简化
		* doctype `<!DOCTYPPE html>`
		* 字符集 `<meta charset=utf-8>`
	* 可以使用所有语言
	* 插件的简化
	* 默认安全（跨域）
	* 新的form 
		* 旧浏览器中新的表单控件会平滑降级 input -> text
		* input type
	* audio&video标签
	* Canvas像素级操作，放大缩小会变形
	* SVG矢量图，伸缩时细节不会发生变化，基于XML
	* Web存储
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
	* WebSocket

## CSS

* **why**
	* 可以从文档中分离出文档的样式，更便于维护和避免重复
	* 可以使用相同的内容加以不同的样式实现不同的目的
	* 灵活、简短、清晰、基础格式化工具、简单的多文件管理、使用类选择器节省时间
* **选择器类型**
	* id
	* class
	* 后代选择器 `div p`
	* 子元素选择器`div>p`
	* 相邻兄弟选择器`div+p`
	* 后序兄弟选择器`div~p`
* 2.1，3新特性
* **CSS盒模型**
	* ![image-20210104191450695](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20210104191450695.png)
	* 最终宽度=padding* 2+margin*2+border * 2+contentWidth
* **响应式网页设计，主要手段。**
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
* **优先级顺序和继承关系**
	* https://zhuanlan.zhihu.com/p/125536847
* **布局**
	* **BFC（block formatting context）**
		* 块级格式化上下文
		* 内部box在垂直方向放置
		* BFC就是页面上的⼀个隔离的独立容器，容器里面的子元素不会影响到外面的元素。
		* 反之也如此。 计算BFC的高度时，浮动元素也参与计算
	* **IFC（inline formatting context）**
		* 多个内联元素排列在一起时形成一个IFC，之间不能穿插块级元素
		* 一个IFC内的元素都是水平排列的
		* 横向的margin、border、padding属性对于这些元素都有效
		* 垂直方向可以调整对齐方式
* **移动优先**
	* 优先显示最重要的内容和功能，如果空间允许，再逐步加入次要内容与功能
	* 好处
		* 只要移动端做的好，即使用户使用的是旧版本浏览器、没有 Javascript或者关闭了Javascript的浏览器，或为视力残障人士设计 的读屏浏览器，也能看到⼀个拥有基本功能的网站。 
		* 移动优先是渐进将强理念的良好范例，所有用户都能访问核心内容和功能。不存在不能访问的情况

> **渐进渐强**：首先基于一个具有广泛兼容性的核心方案，创建一个基线版本，然后再根据可能用到的浏览器特性，慢慢添加一些特性和功能

## JavaScript

>  \- 基本语法
>
> \- 严格模式
>
> \- "first-class" functions
>
> \- 事件驱动编程
>
> \- 面向对象
>
> \- 匿名函数
>
> \- 作用域、作用域链、闭包及其用途

### 基本语法

* 大小写敏感
* 分号可以没有
	* 如果当前语句无法和下一句合并解析，JavaScript则在第一行后填补分号
	* 如果语句涉及return、break和continue这三个关键字，并且后面跟有换行符，那么不管是否可以和下一句解析，他都会在第一行后填补分号
* 注释同java
* 对象
	* 对象是properties集合
	* arrays、functions可以看作是对象

* 类型：Number, Boolean, String, Array, Object, Function, Null, Undefined

### 严格模式

* 为什么
	* 消除Javascript语法的⼀些不合理、不严谨之处，减少⼀些怪异行为; 
	* 消除代码运行的⼀些不安全之处，保证代码运行的安全； 
	* 提高编译器效率，增加运行速度； 
	* 为未来新版本的Javascript做好铺垫。
* https://www.runoob.com/js/js-strict.html

### "first-class" functions

函数享有与变量同等的待遇

- 可被赋值给变量、数列元素和对象属性
- 可作为参数传递给其他函数
- 可被函数作为返回值

允许声明**高阶函数（higher-order function）**

- 接受函数作为参数或者返回函数的函数为高阶函数，如`map()`, `filter()`, `reduce()`

### 事件驱动编程

JavaScript是基于对象(object-based)的语言。这与Java不同,Java是面向对象的语言。而基于对象的基本特征，就是采用事件驱动(event-driven)。它是在用形界面的环境下，使得一切输入变化简单化。通常鼠标或热键的动作我们称之为事件(Event)，而由鼠标或热键引发的一连串程序的动作，称之为事件驱动(Event Driver)。而对事件进行处理程序或函数，我们称之为事件处理程序(Event Handler)。

![image-20210103215455944](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20210103215455944.png)

* 观察者模式

	![image-20210103215740427](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20210103215740427.png)

	使用观察者模式添加事件回调

	```javascript
	Event.observe(windows,"load",myFuntion);
	// 或
	var hiddenBox = $( "#banner-message" );
	$( "#button-container button" ).on( "click", function( event ) {
	 hiddenBox.show();
	});
	```

* event类型

* event对象

	![image-20210103220351009](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20210103220351009.png)

* dom0与dom2

	https://blog.csdn.net/qq_23389687/article/details/80166843

	https://blog.csdn.net/xiaoyuer_2020/article/details/109292826?utm_medium=distribute.pc_relevant.none-task-blog-baidujs_title-2&spm=1001.2101.3001.4242

	https://blog.csdn.net/m0_37937502/article/details/82830992

* **事件注册方法汇总**

	![image-20210103231908320](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20210103231908320.png)

### 面向对象

### 匿名函数

### 作用域、作用域链、闭包及其用途

* ES6之前，只有函数作用域和全局作用域 
* ES6增加了通过let和const声明变量的块级作用域
* 作用域分类
	* 全局作用域
	* 局部作用域

[JS基础-作用域和作用域链以及闭包](https://www.cnblogs.com/lingXie/p/11493820.html)

[JavaScript系列----作用域链和闭包](https://www.cnblogs.com/renlong0602/p/4398883.html)

[JS作用域和作用域链](https://www.cnblogs.com/sening/p/4456492.html)

[学习Javascript闭包（Closure）](http://www.ruanyifeng.com/blog/2009/08/learning_javascript_closures.html)

## DOM

### DOM

document object model

基于DOM的XML分析器将一个XML文档转换成一个对象模型的集合（通常称DOM树），应用程序正是通过对这个对象模型的操作，来实现对XML文档数据的操作

HTML DOM定义了一种标准对象模型和对于HTML的程序编程接口，换言之，HTML DOM是一种获取、改变、增加、删除HTML元素的标准

![image-20210106203059352](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210106203059352.png)

### 事件流

* 事件冒泡

	* 即事件开始时由最具体的元素（文档中嵌套层次最深的那个节点）接受，然后逐级向上传播到较为不具体的节点（文档）。
	* 所有浏览器均支持事件冒泡。Firefox、chrome、safari将事件一直冒泡到window对象。

* 事件捕获

	* 从window对象开始捕获（DOM2级规范是从document开始）

* DOM事件流

	* DOM事件流包括三个阶段：
		1. 事件捕获阶段
		2. 处于目标阶段
		3. 事件冒泡阶段

	```js
	var button = document.getElementById('clickMe');
	 
	button.onclick = function() {
	  console.log('1. You click Button');
	};
	document.body.onclick = function() {
	  console.log('2. You click body');
	};
	document.onclick = function() {
	  console.log('3. You click document');
	};
	// window.onclick = function() {
	//   console.log('4. You click window');
	// };
	window.addEventListener('click', function() {
	  console.log('4. You click window');
	}, true);
	```

	```html
	<body>
	  <button id="clickMe">Click Me</button>
	</body>
	```

	![image-20210106203714961](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210106203714961.png)

	`event.stopPropagation();`可以阻止事件冒泡

### DOM0和DOM2

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

## AJAX

* RIA

	* rich internet applications RIAs 富因特网应用

* 同步、异步通信

	* 同步：用户必须等待知道新页面加载完成，几乎所有新数据的变化都会导致页面刷新
	* 异步：在新数据加载过程中，用户可以保持与页面的交互

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

* 安全相关、SOP、跨域

* 数据格式

	* xml
	* json

## NodeJS*

* 特点

	* 非阻塞异步IO

		![image-20210104145559271](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20210104145559271.png)

	* V8 JavaScript 引擎

	* 单线程，event loop

	* 超过一百万的modules

	* 跨平台

	* 与前端语言一致

	* 活跃的社区

* 应用场景

	* 网站、RestAPI、http proxy、反向代理、前端构建工具、打包工具、命令行工具等

* 回调地狱--

* 缺点：

	* 不适合CPU密集型应用

## 框架



## 优化* 

* web speed的对SEO作用
	* 越快的网页，越容易爬取
	* 越快的网页，越容易获取
	* 排名更靠前
	* 更吸引用户

* 基准测试/性能分析

	* 测试工具

		![image-20210104150741389](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20210104150741389.png)

	* 影响基准测试表现的

		![image-20210104150748076](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20210104150748076.png)

	* 性能监控指标

		* ![image-20210104150911762](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20210104150911762.png)
		* ![image-20210104150918549](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20210104150918549.png)
		* ![image-20210104150923513](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20210104150923513.png)

* 基本原理

	* 基本原理 
		* 基于文档的优化：熟悉网络协议，了解文档、CSS 和 JavaScript 解析管道， 发现和优先安排关键网络资源，尽早分派请求并取得页面，使其尽快达到可交互的 状态。主要方法是优先获取资源、提前解析等。 
		* 推测性优化：浏览器可以学习用户的导航模式，执行推测性优化，尝试预测用户 的下⼀次操作。然后，预先解析DNS、预先连接可能的目标。

* 优化思路、技术、方法

	* 浏览器优化
		* 资源预取和排定优先次序：文档、CSS 和 JavaScript 解析器可以与网络协议层 沟通，声明每种资源的优先级:初始渲染必需的阻塞资源具有最⾼优先级，而低 优先级的请求可能会被临时保存在队列中。 
		* DNS预解析：对可能的域名进行提前解析，避免将来 HTTP 请求时的DNS延 迟。预解析可以通过学习导航历史、用户的鼠标悬停，或其他页面信号来触发。 
		* TCP预连接：DNS解析之后，浏览器可以根据预测的HTTP请求，推测性地打开 TCP连接。 如果猜对的话，则可以节省⼀次完整的往返(TCP握⼿)时间。 
		* 页面预渲染：某些浏览器可以让我们提示下⼀个可能的目标，从而在隐藏的标签 ⻚中预先渲染 整个页面。这样，当用户真的触发导航时，就能立即切换过来。