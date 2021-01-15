# this是什么

JavaScript this 关键词指的是它所属的对象。

# this的指向

* 在函数体中，非显式或隐式地简单调用函数时，在严格模式 下，函数内的this会被绑定到undefined上，在非严格模式下则会被绑定到全局对象window/global上。 
* 一般使用new方法调用构造函数时，构造函数内的this会被绑定到新创建的对象上。 
* 一般通过call/apply/bind方法显示调用函数时，函数体内的 this会被绑定到指定参数的对象上。
*  一般通过上下文对象调用函数时，函数体内的this会被绑定到该对象上。 
* 在箭头函数中，this的指向是由外层（函数或全局）作用域来决定的

## 全局环境中的this

* 在函数体中，非显式或隐式地简单调用函数时，在严格模式 下，函数内的this会被绑定到undefined上，在非严格模式下则会被绑定到全局对象window/global上。 

例子1：

```javascript
    function f1() {
        console.log(this);
    }
    function f2() {
        "use strict";
        console.log(this);
    }
    f1(); // window
    f2(); // undefined
```

例子2：

```js
    var foo = {
        bar: 10,
        fn: function() {
            console.log(this)
            console.log(this.bar)
        }
    }
    var fn1 = foo.fn
    fn1()
// window
// undefined
```

## 上下文对象调用中的this

* this指向最后调用它的对象

例子3：

```js
    var foo = {
        bar: 10,
        fn: function() {
            console.log(this)
            console.log(this.bar)
        }
    }
    foo.fn()
```

![image-20210104132008085](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210104132008085.png)

例子4：

```js
    var person = {
        name: 'Lucas',
        brother: {
            name: 'Mike',
            fn: function() {
                return this.name
            }
        }
    }
    console.log(person.brother.fn())// Mike
// 最后的this指向的是brother
```

例子5：

```javascript
    var o1 = {
        text: 'o1',
        fn: function() {
            return this.text
        }
    }
    var o2 = {
        text: 'o2',
        fn: function() {
            return o1.fn()
        }
    }
    var o3 = {
        text: 'o3',
        fn: function() {
            var fn = o1.fn
            return fn()
        }
    }
    console.log(o1.fn()) //o1
    console.log(o2.fn()) //o1
    console.log(o3.fn()) //undefined  此时this为window，没有text
```

例子5（在例子4的基础上加上了严格模式）：

```js
    var o1 = {
        text: 'o1',
        fn: function() {
            "use strict";
            return this.text
        }
    }
    var o2 = {
        text: 'o2',
        fn: function() {
            return o1.fn()
        }
    }
    var o3 = {
        text: 'o3',
        fn: function() {
            var fn = o1.fn
            return fn()
        }
    }
    console.log(o1.fn()) //o1
    console.log(o2.fn()) //o1
    console.log(o3.fn()) //error
```

> 关于前两种情况，建议参考[彻底理解js中的this](https://www.cnblogs.com/pssp/p/5216085.html)，可以直接理解

## 通过bind、call、apply改变this指向

* 一般通过call/apply/bind方法显示调用函数时，函数体内的 this会被绑定到指定参数的对象上。

例子6：

```js
    const foo = {
        name: 'lucas',
        logName: function() {
            console.log(this.name)
        }
    }
    const bar = {
        name: 'mike'
    }
    foo.logName.call(bar) // mike
```

## 构造函数中的this

* 一般使用new方法调用构造函数时，构造函数内的this会被绑定到新创建的对象上。 

例子7：

```js
    function Foo() {
        this.bar = "Lucas"
    }
    const instance = new Foo()
    console.log(instance.bar)  // Lucas
```

## 事件绑定中的this

* 事件源.onclik = function(){ } //this -> 事件源
* 事件源.addEventListener(function(){ }) //this->事件源

例子8：

```js
	var div = document.querySelector('div'); 
    div.addEventListener('click',function() {
        console.log(this); //this->div
    });
    
    div.onclick = function() {
    console.log(this) //this->div
    }
```

## 定时器中的this

* 定时器中的this->window，因为定时器中采用回调函数作为处理函数，而回调函数的this->window

* 箭头函数使用this 不适用以上标准规则，而是根据外层（函数 或者全局）上下文来决定。

例子9：

```js
    const foo = {
        fn: function(){
            setTimeout(function() {
                console.log(this)
            })
        }
    }
    foo.fn() // window
```

例子10：

```js
    const foo = {
        fn: function(){
            setTimeout(()=>{
                console.log(this)
            })
        }
    }
    foo.fn() //{fn:f}
```

> 参考：
>
> https://www.cnblogs.com/lingXie/p/11493723.html
>
> https://blog.csdn.net/chen_junfeng/article/details/109235442