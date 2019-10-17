> 引言

可以说是取名废了，把这几个关键词放在一起也是因为看完犀牛书相关章节和很多技术文章之后觉得这几个概念是相互渗透的，需要放在一起理解。而在这之前，我对这几个概念都是久闻其名，真正遇到了似懂非懂还很心虚，每每看完一篇零零散散的技术文章总觉得明白了，但是下一次遇到了还是不明白。这种其实充其量只能说记住了一些知识点碎片，而学习不能只是碎片，必须是找到碎片与碎片之间的关联与规律，才能将知识碎片连成完整的知识体系吃下去。所以，某位大佬说得对，有产出的学习才是真的学习，毕竟如果不是真的搞清楚了，你什么也写不出来，何况写一遍还能加深印象。如果你也有类似的情况，请重视起来，一起加油叭！



本章对标犀牛书第6版**3.9**变量声明、**3.10**变量作用域和第**8**大章**8.6**之前的章节，之后两小节值得在后面的文章里更详细的单独研究！



### 变量声明和函数定义

*你可能觉得这过于基础了，但事实上如果忽略其中的一些细节会影响后面的理解。*

#### 变量声明

我们知道在ES6之前，我们通常用`var`关键字来声明一个或多个变量`var i，sum;`，还可以将变量的初始赋值和变量声明写在一起`var i = 0, j = 1, k = 2;`，如果没有在`var`声明语句中给变量指定初始值，那么虽然声明了变量，但在给他赋值之前，它的值就是undefined。

如果试图读取一个没有声明的变量，会触发一个报错。在严格模式下，给一个未声明的变量赋值也会触发一个报错，在非严格模式下，给一个未声明的变量赋值，js实际上会给全局对象创建一个同名属性，使其工作起来**像**一个正常声明的全局变量。

> `var`声明的属性是不可配置的，也就是不可以用`delete`操作符删除，而给未声明的变量赋值创建的全局对象同名属性是正常的可配置属性，是可以被删除的

#### 函数定义

函数定义可以通过两种方式来进行：

- 函数定义表达式 `var func = function(){...};`
- 函数声明` function func(){...}`

虽然两者都包含相同的函数名，都创建了一个新的函数对象，但还是有所区别，区别就体现在声明提升的时候。



### 作用域和声明提升

#### 全局作用域和函数作用域

ES6之前，JS是没有常用的不受约束的块级作用域的，只有全局作用域和函数作用域。全局作用域是顶级作用域，顾名思义，在顶级声明的所有变量全局中都可访问；函数作用域是指在函数内声明的所有变量在函数体内始终可见，**包括嵌套函数内**。以及，函数体内的局部变量优先级高于同名的全局变量，如果函数内声明的全局变量或函数参数带有的变量和全局变量同名，那么全局变量就被局部变量覆盖。例如：

```javascript
var scope = 'global';
function f() {
  var scope = 'local';
  console.log(scope); //local 同名的全局变量被局部变量覆盖
  function f2() {
    console.log(scope); //local 函数体内声明的变量在嵌套函数内也可见
  }
  f2();
}
f(); 
```

#### 声明提升

正常情况下我们认为js语句是由上到下一句一句执行的，这完全没错，但是有一种情况会使我们很迷惑。

```javascript
a = 2;
var a;
console.log(a); //2

console.log(a); //undefined
var a = 2; 
```

实际上这是因为包括变量和函数在内的所有声明（但**不涉及赋值**）都会在如何代码被执行前也就是js的预编译阶段被首先处理，会被”提升“到相应作用域的顶部，这个过程就是声明提升，关于声明提升，我们只需要注意以下几点：

1. 函数声明也会被提升，但函数表达式不会。也就是说上述两种函数定义的方式，使用函数声明语句，函数变量名和函数体均会提升，但使用`var`的表达式，只是变量声明被提升，函数体会留在原来的位置等待执行；

   ```javascript
   f();
   
   function f(){
     console.log(a);  //会被执行且输出undefined
     var a = 2;
   }
   
   f(); //TypeError f is not a function
   var f = function bar() {
     console.log(a);  
     var a = 2;
   }
   ```

2. `var`定义的具名的函数表达式，函数名变量也不会被提升；

   ```javascript
   f(); //TypeError f is not a function
   bar(); //ReferenceError: bar is not defined
   var f = function bar() {
     console.log(a);  
     var a = 2;
   }
   //注意这里f()抛出的是TypeError而不是ReferenceError 是因为变量名已经被提升但没有被赋值，所以此时f是undefined，对undefined做调用执行，因此抛TypeError异常。
   //这段代码实际上被编译成以下形式：
   var f;
   f(); //TypeError
   bar(); //ReferenceError
   f = function() {
     var bar = self;
     var a
     a = 2;
   }
   ```

3. 函数声明优先，已知变量声明和函数声明都会被提升，如果同一作用域里有相同命名的函数声明和变量声明提升，函数声明优先。

   ```javascript
   foo();  // foo2
   var foo = function() {
       console.log('foo1');
   }
   
   foo();  // foo1，foo重新赋值
   
   function foo() {
       console.log('foo2');
   }
   
   foo(); // foo1
   
   //这段代码实际上被形容成
   function foo() {
     console.log('foo2');
   }
   foo();
   foo = function() {
     console.log('foo1');
   }
   foo();
   foo();
   //尽管var foo出现在function foo(){..}之前，但是函数声明会被提升到普通变量声明之前，因此重复的声明会被忽略，但会被重新赋值。此外，后面的函数声明还是可以覆盖前面的，比如：
   foo(); // foo2
   function foo() {
     console.log('foo1');
   }
   function foo() {
     console.log('foo2');
   }
   ```

4. 条件语句中的函数声明：

   **这个东西迷惑了我好久！！！浪费我的时间！！！**

   MDN中的相关解释：https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/function

   Segmentfault中回复楼的相关解释：https://segmentfault.com/q/1010000005337550

   以及下面的（【阮一峰】ES6入门）中也有相关解释：[【阮一峰】ES6入门](http://es6.ruanyifeng.com/#docs/let#块级作用域与函数声明)

   ```javascript
   foo();
   var a = true;
   if(a) {
   	function foo(){console.log('foo1')}
   } else {
   	function foo(){console.log('foo2')}
   }
   //理论上，这段代码会输出foo2
   //经实验，实际上在大多数浏览器控制台里会抛出TypeError foo is not a function，在Safari里输出了foo2
   
   (function () {
     if (false) {
       // 重复声明一次函数f
       function f() { console.log('I am inside!'); }
     }
     f(); //TypeError foo is not a function
   }());
   
   //总之，在块级作用域和条件语句中声明函数，在不同浏览器中解释的标准都不一样，考虑到环境导致的行为差异太大，不要在块级作用域或条件语句中声明函数！！！
   // 等我有时间了再来纠结这个吧。。。。。。不过感觉遥遥无期。。。。。
   
   ```

#### 块级作用域

偷个懒~  [【阮一峰】ES6入门](http://es6.ruanyifeng.com/#docs/let#块级作用域)

注释一点：

> 绝大部分的文章里都提到在ES6之前，js中是没有块级作用域的，只有函数作用域和全局作用域，实际上，ES3中的try/catch语句中的catch语句就会创建一个块级作用域
>
> ```javascript
> try {
> 	throw 2
> } catch (a) {
> 	console.log(a); //2
> }
> console.log(a);//Reference Error
> ```
>
> 我试着在babel在线编译器里转换
>
> ```javascript
> {
> 	let a = 2;
> 	console.log(a);
> }
> console.log(a);
> ```
>
> 但是得到的是
>
> ```javascript
> "use strict";
> 
> {
> var _a = 2;
> console.log(_a);
> }
> console.log(a);
> 
> ```
>
> 



### 作用域链和闭包

#### 作用域链

说作用域链先理解一下自由变量：当前作用域没有定义的变量即自由变量。如何访问到自由变量：向父级作用域寻找。如果父级作用域没有，则一层一层向上寻找，直到找到全局作用域，如果还是没找到，则宣布放弃。

这个一层一层向上的层级关系，就是作用域链，它是一个对象列表。

只用记住一点：自由变量将从作用域链中去寻找， **依据的是函数定义时的作用域链，而不是函数执行时**，JS采用的是静态作用域，依据的是函数定义时的位置。

```javascript
function F1() {
    var a = 100
    return function () {
        console.log(a)
    }
}
var f1 = F1()
var a = 200
f1() //100

```

#### 闭包

**当函数可以记住并访问函数体内的变量时，就形成了闭包**，因此严格来说每个函数都是闭包。

注意三点：

1. 同一个作用域链中的多个闭包共享私有变量或变量

   ```javascript
   function countFunc() {
   	var funcs = [];
   	for(var i = 0; i < 10; i++){
       funcs[i] = function (){ return i }
     }
     return funcs
   }
   countFunc()[0]() //10
   //这段代码创建了10个闭包，但是都是在一个函数调用中定义的，因此它们共享变量i
   
   ```

2. 每次调用js函数的时候，都会创建一个新的作用域链

   ```javascript
   function counter() {
     var n = 0;
     return {
       counter: function() { return n++; },
       reset: function() { n = 0; }
     }
   }
   var c = counter();
   var d = counter();
   c.counter(); //0
   d.counter(); //0 互不干扰
   c.reset();  //重置c
   c.counter(); //c的reset和counter共享n变量
   
   ```

3. this是JS关键字而不是变量，每一个函数调用都包含一个this值，如果闭包在外部函数中，是无法访问外部函数的this值的，arguments同理，虽然它不是关键字，但是调用函数时会自动声明它。



### 执行上下文**this**

记住大佬做的一张图就够了（侵删）：

![img](https://user-gold-cdn.xitu.io/2018/11/15/16717eaf3383aae8?imageslim)





小结

------

有点曲折的一章，被条件语句和块级作用域中的函数声明搞得头大。

犀牛书没有把这一块说的很清楚，看了《你不知道的JS》相关章节作补充

后续需要补充：js编译执行机制，this详解，面试题详解




