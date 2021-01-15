# JavaScript提升

引擎会在解释JavaScript代码之前首先进行编译，编译过程中 的⼀部分工作就是找到所有的声明，并用合适的作用域将他们关联起来，这也正是词法作用域的核心内容。 

提升（Hoisting）是 JavaScript 将声明移至顶部的默认行为。

例子1：

```javascript
    var x=1;
    (function () {
        console.log(x);
        var x=2;
    }());
// undefined
```

以上的代码在提升后相当于

```js
    var x=1;
    (function () {
        var x;
        console.log(x);
        x=2;
    }());
// undefined
```

# let和const

* ES6新增块级作用域。这个区块对这些变量从⼀开始就形成了 封闭作用域，直到声明语句完成，这些变量才能被访问（获取 或设置），否则会报错ReferenceError。 
* 暂时性死区（英temporal dead zone，简 TDZ），即代码块开始到变量声明语句完成之间的区域。 
* 通过 let 声明的变量没有变量提升、拥有暂时性死区，作用于块级作用域： 
	* 当进入变量的作用域（包围它的语法块），立即为它创建（绑定）存储空间， 不会立即初始化，也不会被赋值
	* 访问（获取或设置）该变量会抛出异常 ReferenceError 
	* 当执行到变量的声明语句时，如果变量定义了值则会被赋值，如果变量没有定 义值，则被赋值为undefined

例子2：

```js
    { // TDZ starts at beginning of scope
        console.log(bar); // undefined
        console.log(foo); // ReferenceError
        var bar = 1;
        let foo = 2; // End of TDZ (for foo)
    } 
```

例子3（函数提升只会提升函数声明，不会提升函数表达式）：

```js
    console.log(foo1); // [Function: foo1]
    foo1(); // foo1
    console.log(foo2); // undefined
    foo2(); // TypeError: foo2 is not a function
    function foo1 () {
    console.log("foo1");
    };
    var foo2 = function () {
    console.log("foo2");
    };
```

![image-20210104131015716](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210104131015716.png)