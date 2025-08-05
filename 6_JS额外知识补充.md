## with语句

with语句 扩展一个语句的作用域链。

![image-20220617073630003](.\6_JS额外知识补充\image-20220617073630003.png)



会现在obj中找name，如果找不到，就在obj的外层找（注意，不是with的外层作用域找）

他会先在with传过来的对象中寻找，如果找不到，再往上找

- 不建议使用with语句，因为它可能是混淆错误和兼容性问题的根源。
- 严格模式下使用with会报错

![image-20220617073658620](.\6_JS额外知识补充\image-20220617073658620.png)





## eval函数

eval是一个特殊的函数，它可以将传入的字符串当做JavaScript代码来运行。

![image-20220617073952926](.\6_JS额外知识补充\image-20220617073952926.png)



不建议在开发中使用eval： 

- eval代码的可读性非常的差（代码的可读性是高质量代码的重要原则）；
- eval是一个字符串，那么有可能在执行的过程中被刻意篡改，那么可能会造成被攻击的风险；
- eval的执行必须经过JS解释器，不能被JS引擎优化；

![image-20220617074016641](.\6_JS额外知识补充\image-20220617074016641.png)

我们的代码都会通过webpack进行打包，如果通过webpack进行打包的时候给webpack进行配置，把devtool配置为eval，那么打包后的代码就都是字符串，这样做的原因是性能会稍微高一点

但是V8在解析这种字符串代码的时候，是没有优化的





## 认识严格模式

在ECMAScript5标准中，JavaScript提出了严格模式的概念（Strict Mode）：

- 严格模式很好理解，是一种具有限制性的JavaScript模式，从而使代码隐式的脱离了 ”懒散（sloppy）模式“；
- 支持严格模式的浏览器在检测到代码中有严格模式时，会以更加严格的方式对代码进行检测和执行；

严格模式对正常的JavaScript语义进行了一些限制：

- 严格模式通过 抛出错误 来消除一些原有的 静默（silent）错误；

```javascript
非严格模式下
let obj = {};
Object.definedProperty(obj, "name", {
writable: false
}
);
obj.name = 'wts'    //非严格模式下不会报错，也不会赋值上
// 严格模式下就会报错
```

- 严格模式让JS引擎在执行代码时可以进行更多的优化（不需要对一些特殊的语法进行处理）；
- 严格模式禁用了在ECMAScript未来版本中可能会定义的一些语法；







## 严格模式限制

这里我们来说几个严格模式下的严格语法限制： 

- JavaScript被设计为新手开发者更容易上手，所以有时候本来错误语法，被认为也是可以正常被解析的；
- 但是这种方式可能给带来留下来安全隐患；
- 在严格模式下，这种失误就会被当做错误，以便可以快速的发现和修正；
- 严格模式的开启，针对的是每一个文件，也就是文件1开启，不影响文件2
- webpack默认会帮我们开启严格模式
- 可以单独对函数开启严格模式



1. 无法意外的创建全局变量

```javascript
禁止意外创建全局变量
message = "Hello World"
console.log(message)

function foo() {
  age = 20    //非严格模式下，这样创建的是一个全局变量，这是一个语法错误
}

foo()
console.log(age)
```



2. 严格模式会使引起静默失败(silently fail,注:不报错也没有任何效果)的赋值操作抛出异常

```javascript
true.name = "abc"
NaN = 123
var obj = {}
Object.defineProperty(obj, "name", {
  configurable: false,
  writable: false,
  value: "why"
})
console.log(obj.name)
obj.name = "kobe"    //非严格模式下不会报错， 被称为静默错误
delete obj.name    //非严格模式下是不会报错的
```



3. 严格模式下试图删除不可删除的属性
4. 严格模式不允许函数参数有相同的名称

```javascript
function foo(x, y, x){
    console.log(x, y, x)    //非严格模式下，x指向都是最后一个x，并且不会报错
}
```



5. 不允许0的八进制语法

```javascript
// 4.不允许使用原先的八进制格式 0123
// var num = 0o123 // 八进制
// var num2 = 0x123 // 十六进制
// var num3 = 0b100 // 二进制
// console.log(num, num2, num3)
```



6. 在严格模式下，不允许使用with
7. 在严格模式下，eval不再为上层引用变量

```javascript
var jsString = 'var message = "Hello World"'; console.log(message);'    

//可以单独为eval开启严格模式
var jsString = '"user strict";var message = "Hello World"'; console.log(message);'    
eval(jsString)
console.log(message)    //开启严格模式以后，eval中的message不会加到window中，所以这里找不到了，如果不开启严格模式，message会添加到全局中
```



8. 严格模式下，this绑定不会默认转成对象

```javascript
// 在严格模式下, 自执行函数(默认绑定)会指向undefined
// 之前编写的代码中, 自执行函数我们是没有使用过this直接去引用window， 我们一般直接用window   window.localStorage.setItem
function foo() {
  console.log(this)    
}
foo()    //undefined

var obj = {
  name: "why",
  foo: foo
}

foo()    //undefined

obj.foo()    //这样foo的this指向obj
var bar = obj.foo
bar()    //undefined

// setTimeout的this
// fn.apply(this = window)， setTimeout不是V8实现的，而是浏览器实现的
setTimeout(function() {
  console.log(this)    //严格模式下  这样写，this指向的是window,所以说明，这里的函数并不是一个自执行函数，而是绑定到了window中
}, 1000);

setTimeout是一个黑盒子，它是浏览器实现的，它接收一个function,还接收一个deay，接收到这个函数以后，我们把这个函数交给浏览器执行了，那么它是怎么执行的呢，我们猜测它是怎么调用的呢？因为它里面的this是指向window的，所以我么猜测它可能是自执行，但是在严格模式下这个函数里面应该指向的是undefined,因为自执行函数里面的this指向的都是windown，但是在setTimeout中的this指向的是window， 所以他应该不是自执行函数，它有可能通过apply绑定到了window上。
```

![image-20220617074546297](.\6_JS额外知识补充\image-20220617074546297.png)

浏览器的源码里面setTimeout中函数的源码指向了外层的this（也就是fakeWin, 也就是window），这个this就指向的是window

setTimeout也可以传字符串，如果传的是字符串就是通过eval来执行



9.NAN在严格模式下不能赋值