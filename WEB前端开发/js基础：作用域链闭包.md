# JS基础：变量、作用域、闭包

## 1 作用域

* 在 JavaScript 中, 对象和函数同样也是变量。
* 在 JavaScript 中, 作用域为可访问变量，对象，函数的集合。

**作用域分类：**

* **全局作用域** 
* **局部作用域**

### 1.1 全局作用域

全局变量有 全局作用域: 网页中所有脚本和函数均可使用

函数内部可以直接读取全局变量

全局变量是 window 对象: 所有数据变量都属于 window 对象

### 1.2 局部作用域

变量在函数内声明，只能在函数内部访问，即局部变量

函数参数也是局部变量

因为局部变量只作用于函数内，所以不同的函数可以使用相同名称的变量

1：函数内部可以读取全局变量

```js
    　　var n=999;

    　　function f1(){
    　　　　alert(n);
    　　}

    　　f1(); // 999
```

2：在函数外部自然无法读取函数内的局部变量

```js
    　　function f1(){
    　　　　var n=999;
    　　}

    　　alert(n); // error
```

3：如果变量在函数内没有声明（没有使用 var 关键字），该变量为全局变量

```js
    　　function f1(){
    　　　　n=999;
    　　}

    　　f1();

    　　alert(n); // 999
```

### 1.3 无块级作用域

4：可能导致内部变量可能覆盖外层变量

```js
var i = 5;
function func(){
    console.log(i);
    if(true){
        var i = 6;
    }
}
func();//undefined
```

5：局部变量可能泄露为全局变量

```js
for(var i = 0; i < 10; i++){
	console.log(i);
}
console.log('i',i);//10 
```


### 1.4 什么是变量

**变量包括两种，普通变量和函数变量。** 

- 普通变量：凡是用var标识的都是普通变量

	比如下面 ：

	```js
	var x=1;               
	var object={};
	var  getA=function(){};  //以上三种均是普通变量，但是这三个等式都具有赋值操作。所以，要分清楚声明和赋值。声明是指 var x; 赋值是指 x=1; 
	```

- 函数变量：函数变量特指的是下面的这种，fun就是一个函数变量。

	```js
	function fun(){} ;// 这是指函数变量. 函数变量一般也说成函数声明。
	```

	
	类似下面这样，不是函数声明，而是函数表达式

	```js
	var getA=function(){}      //这是函数表达式
	var getA=function fun(){}; //这也是函数表达式，不存在函数声明。关于函数声明和函数表达式的区别，详情见javascript系列---函数篇第二部分
	```

 

**什么是变量声明？**

* **变量有普通变量和函数变量，所以变量的声明就有普通变量声明和函数变量声明。**

-  普通变量声明

	```js
	var x=1; //声明+赋值
	var object={};   //声明+赋值
	```

	 上面的两个变量执行的时候总是这样的

	```js
	var x = undefined;      //声明
	var object = undefined; //声明
	x = 1;                  //赋值
	object = {};            //赋值
	```

	关于声明和赋值，请注意，声明是在函数第一行代码执行之前就已经完成，而赋值是在函数执行时期才开始赋值。所以，声明总是存在于赋值之前。而且，普通变量的声明时期总是等于undefined.

- 函数变量声明

	函数变量声明指的是下面这样的：

	```js
	function getA(){}; //函数声明
	```

**声明提前到什么时候？**

* 所有变量的声明，在函数内部第一行代码开始执行的时候就已经完成。见1.5

### 1.5 函数作用域

一个函数在执行时所用到的变量无外乎来源于下面三种：

1. 函数的参数----来源于函数内部的作用域

2. 在函数内部声明的变量（普通变量和函数变量）----也来源于函数内部作用域

3. 来源于函数的外部作用域的变量，放在1.3中讲。

```js
var x = 1;
function add(num) () {
  var y = 1; 
  return x + num + y;   //x来源于外部作用域，num来源于参数（参数也属于内部作用域），y来源于内部作用域。
}
```

**函数作用域的创建步骤：**

1. 函数形参的声明。
2. 函数变量的声明

3. 普通变量的声明。  

4. 函数内部的this指针赋值
5. .....函数内部代码开始执行！  

需要强调：

* 函数形参在声明的时候已经指定其形参的值

	```js
	function add(num) {
	  var num;
	  console.log(num);   //1
	}
	add(1);
	```

* 在第二步函数变量的生命中，函数变量会覆盖以前声明过的同名声明

	```js
	function add(num1, fun2) {
	  function fun2() {
	    var x = 2;
	  }
	  console.log(typeof num1); //function  
	  console.log(fun2.toString()) //functon fun2(){ var x=2;}
	}
	add(function () {
	}, function () {
	  var x = 1
	}); 
	```

* 在第三步中，普通变量的声明，不会覆盖以前的同名参数

	```js
	function add(fun,num) {
	  var fun,num;
	  console.log(typeof fun) //function
	  console.log(num);      //1
	}
	add(function(){},1);
	```

	其实这是因为js允许重复声明，新声明的东西没赋值，就直接忽略了，可以参考以下

	```js
	function add(fun,num) {
	  var fun=1,num;
	  console.log(typeof fun) //number
	  console.log(num);      //1
	}
	add(function(){},1);
	```

* 一个小例子，检查一下理解了吗？

	```js
	function getA() {
	  if (false) {
	    var a = 1;
	  }
	  console.log(a);  //undefined
	}
	getA();
	```

## 2 作用域链

当声明一个函数时，局部作用域一级一级向上包起来，就是作用域链

1. 函数中，变量先从局部作用域找，未找到，则去上一层局部作用域找，没有，则去全局作用域找，全局未找到，则报错;

2. 当前作用域没有定义的变量，即为‘自由变量’

```js
    var a = 100;
    function fn(){
      var b =201;
      console.log('a',a);//a 自由变量
      console.log('b',b);
      console.log('c',c);
    }  
    fn();
```

![image-20210107103039977](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210107103039977.png)

## 3 闭包

通过作用域链，我们知道内部变量可以很容易地访问外部变量，但是外部变量有没有办法访问内部变量呢？——当然是有的，通过闭包！

### 3.1 闭包的概念

**闭包的概念:有权访问另一个作用域的函数。**

这句话就告诉我们，第一，闭包是一个函数。第二，闭包是一个能够访问另一个函数作用域。

也就是解决了，如何从外部访问局部变量

**构建闭包3步骤：**

1. 使用外层函数封装受保护的局部变量 和 专门操作变量的内层函数

2. 外层函数将内层函数返回(return)到外部

3. 在全局调用外层函数，获得内部函数的对象，保存在全局变量中反复使用

例子：

```js
    　　function f1(){
    　　　　var n=999;
    　　　　function f2(){
    　　　　　　alert(n);
    　　　　}
    　　　　return f2;
    　　}
    　　var result=f1();
    　　result(); // 999
```

### 3.2 闭包的作用

* 实现私有成员
* 保护命名空间
* 避免污染全局变量
* 变量需要长期驻存在内存

### 3.3 闭包的缺点

#### 3.3.1 变量污染

```js
var funB,
funC;
(function() {
  var a = 1;
  funB = function () {
    a = a + 1;
    console.log(a);
  }
  funC = function () {
    a = a + 1;
    console.log(a);
  }
}());
funB();  //2
funC();  //3.
```

对于 funB和funC两个闭包函数，无论是哪个函数在运行的时候，都会改变匿名函数中变量a的值，这种情况就会污染了a变量。

两个函数的在运行的时候作用域如下图：

![image-20210107103926056](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210107103926056.png)

解决方法：

既然外部作用域链上的变量时静态的，那么将外部作用域链上的变量拷贝到内部作用域不就可以啦！！ 具体怎么拷贝，当然是通过函数传参的形式啊。

```js
var funB,funC;
(function () {
  var a = 1;
  (function () {
    funB = function () {
      a = a + 1;
      console.log(a);
    }
  }(a));
  (function (a) {
    funC = function () {
      a = a + 1;
      console.log(a);
    }
  }(a));
}());
funB()||funC();  //输出结果全是2 另外也没有改变作用域链上a的值。
```

![image-20210107104026779](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210107104026779.png)

#### 3.3.2 内存泄漏

由于闭包会使得函数中的变量都被保存在内存中，内存消耗很大，所以不能滥用闭包，否则会造成网页的性能问题，在IE中可能导致内存泄露。解决方法是，在退出函数之前，将不使用的局部变量全部删除

## 4 几道例题

例1：

```javascript
		var name = "The Window";
		var object = {
			name : "My Object",
			getNameFunc : function(){
				return function(){
					return this.name;
				};
			}
		};
		alert(name)
		alert(object.getNameFunc()());
//The Window
//The Window
```

相当于在全局作用域下调用`return this.name`，返回的是The Window

例2：

```javascript
		var name = "The Window";
		var object = {
			name : "My Object",
			getNameFunc : function(){
				return function(){
					name = "My Object"
					return this.name;
				};
			}
		};
		alert(name)
		alert(object.getNameFunc()());
//The Window
//My Object
```

`name = "My Object"`修改了全局变量name的值

例3：

```javascript
		var name = "The Window";
		var object = {
			name : "My Object",
			getNameFunc : function(){
             var that = this;
				return function(){
					return that.name;
				};
			}
		};
		alert(object.getNameFunc()());
//My Object
```

`object.getNameFunc()`时，this是object，that变量被赋值了object，最后在全局调用时`return object.name`

例4：

```javascript
		var name = "The Window";
		var object = {
			name : "My Object",
			getNameFunc : function(){
				return function(){
             	var that = this;
					return that.name;
				};
			}
		};
		alert(object.getNameFunc()());
// The Window
```

在例3的基础上改了一下，这时候that是调用时候的this，也就是window，所以返回的是全局变量

例5：

```javascript
    var name = "The Window";
    function f1(){
        var name = "My Function"
        return function(){
            alert(name);
        }
    }
    var res = f1()
    res()
// My Function
```

简单的局部变量

## 参考

[学习Javascript闭包（Closure）](http://www.ruanyifeng.com/blog/2009/08/learning_javascript_closures.html)

[JS基础-作用域和作用域链以及闭包](https://www.cnblogs.com/lingXie/p/11493820.html)

[JavaScript系列----作用域链和闭包](https://www.cnblogs.com/renlong0602/p/4398883.html)

[JS作用域和作用域链](https://www.cnblogs.com/sening/p/4456492.html)