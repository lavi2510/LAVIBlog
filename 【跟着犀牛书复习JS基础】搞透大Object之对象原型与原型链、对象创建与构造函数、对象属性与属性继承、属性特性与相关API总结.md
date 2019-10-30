> 引言

最近比较忙导致这篇拖了好久啊，第二篇的作用域和闭包因为其中一部分没搞得很清楚也很难受，决定不和自己钻牛角尖了，本篇最后的面试题部分会包含一部分闭包的知识点以弥补上篇没讲清和讲的不够详细的知识点。

本篇对标犀牛书第6大章和第9大章





为什么叫大Object，事实上JS将它单独作为一个基本数据类型应该就足以称之为**大**了，也够复杂。撰写本篇的初心还是想搞清楚原型和原型链所以放在最前面，只是在看的过程中发现和相关的知识点很成体系以及也比较重要，所以都总结记录了一下，可以作为补充看。

### 原型和原型链

#### `_proto_`、`prototype`和`constructor`属性

每个JS对象（`null`除外）都自动拥有一个`_proto_`属性，这个属性是一个对象，指向该对象的原型

每个JS函数（`bind()`方法除外）都自动拥有一个`prototype`属性，这个属性也是一个对象，用该函数做构造函数创建的对象将继承这个`prototype`的属性，也就是说理论上任何一个JS函数都可以用作构造函数，并且调用构造函数需要用到`prototype`属性。

`prototype`属性包含一个唯一不可枚举的属性`constructor`，这个属性是一个函数对象，指向该函数的构造函数。

看完上述两点你可能会认为`_proto_`是不是就是对象的原型，`prototype`是不是就是函数的原型呢？ 事实上并不是，但我们可以说对象的`_proto_`属性指向它的原型，构造函数的`prototype`属性指向调用构造函数创建的实例的原型。

这么说有点绕，用图片（来源：https://github.com/mqyqingfeng/Blog/issues/2  侵删）表示即为：

![实例原型与构造函数的关系图](https://github.com/mqyqingfeng/Blog/raw/master/Images/prototype3.png)

综上所述，如果我们有一段代码：

```javascript
function Person(name, age){
  this.name = name;
  this.age = age;
}
let person = new Person('xiao hong', 18);
```

可以得到：

```javascript
person._proto_ === Person.prototype;
Person.prototype.constructor === Person;
```



#### 原型链

我们知道，当执行属性访问表达式时，首先会将表达式的操作主题转化为对象，然后去对象中查找属性，如果找不到就去找与对象的原型中的属性，如果还找不到，就继续查找原型的原型，直到找到最顶层为止。

那么原型的原型是什么？我们知道，`_proto_`和`prototype`属性也只是普通的对象而已，既然是对象，就也有`_proto_`属性，一个普通对象的`_proto_`属性自然指向其构造函数Obejct的`prototype`属性，即`Object.prototype`

那`Object.prototype`的原型呢？我们可以打印一下:

```javascript
console.log(Object.prototype._proto_) // null
```

所以，当我们向上查找属性的时候，查到`Object.prototype`就可以停止了，由这些对象相互关联的原型之间的关系就是原型链。到这里，我们的图片来源：https://github.com/mqyqingfeng/Blog/issues/2  侵删）也可以更新为：

![原型链示意图](https://github.com/mqyqingfeng/Blog/raw/master/Images/prototype5.png)

综上所述，如果我们有一段代码：

```javascript
function Person(name, age){
  this.name = name;
  this.age = age;
}
Person.prototype = {
  getName: function(){return this.name}
}
let person = new Person('xiao hong', 18);
person.toString();
```

我们在person上找不到toString方法，就会去person的原型上找，`person._proto_ === Person.prototype`，结果`Person.prototype`上也没有这个方法，就会再去原型的原型上找即`Person.prototype._proto_`， 要知道`Person.prototype._proto_`只是一个普通的对象，可以被原始的`new Object()`创建，所以`Person.prototype._proto === Object.prototype`，所幸`Object.prototype`上有`toString()`方法，因此调用它，查找到此结束。

#### 补充

1. `constructor`属性

不是每个对象都有`constructor`属性，因此不是每个对象都可以用作构造函数的`prototype`属性对象，但可以显示的定义`constructor`属性反向引用构造函数来修正这个问题。

`constructor`属性也是普通的对象属性，如果找不到改属性，也会从对象原型上继续寻找。如下：

```javascript
function Person() {

}
var person = new Person();
console.log(person.constructor === Person); // true
```

当获取 `person.constructor` 时，其实 `person` 中并没有 `constructor` 属性,当不能读取到`constructor` 属性时，会从 `person` 的原型也就是 `Person.prototype` 中读取，正好原型中有该属性，所以：

```javascript
person.constructor === Person.prototype.constructor
```

2. `_proto_`

其次是 `__proto__` ，绝大部分浏览器都支持这个非标准的方法访问原型，然而它并不存在于` Person.prototype `中，实际上，它是来自于 `Object.prototype` ，与其说是一个属性，不如说是一个 `getter/setter`，当使用` obj.__proto`__ 时，可以理解成返回了 `Object.getPrototypeOf(obj)`。

3. `Function._proto_ === Function.prototype`

这里并不是因为`Function`是对象，有`_proto_`属性，`Function`又是函数，函数对象的原型指向其构造函数`Function.prototype`，这样理解虽然看上去很正确但是是不对的。引用大佬的话:

> Function.prototype是引擎创造出来的对象，一开始就有了，又因为其他的构造函数都可以通过原型链找到Function.prototype，Function本身也是一个构造函数，为了不产生混乱，就将这两个联系到一起了

4. `Object.__proto__  === Function.prototype`

`Object`是对象的构造函数，那么它也是一个函数，当然它的`__proto__`也是指向`Function.prototype`



实际上原型原型链就是这样，但如果你觉得上述还是很难理解，最好再理解一系列的概念，以下内容可以作为作为对原型和继承的补充理解：

### 对象创建与构造函数

对象的三种创建方法：

#### 对象直接量 

例如`let o = {}`，对象直接量是一个表达式，这个表达式的每次运算都会创建并初始化一个新的对象，也就是说在一个函数中使用对象直接量会函数重复调用时创建很多新对象

#### new+构造函数创建对象

例如`let o =  new Object()`

- 其中`new`关键字做了什么呢，根据MDN的介绍

![image-20191026185858754](/Users/chenxingjian/Library/Application Support/typora-user-images/image-20191026185858754.png)其中第2点即，设置创建的新对象的原型与构造函数的`prototype`属性相关联。

注意：`new`关键字也不是一定创建一个新对象的，例如：`new Object({})`， 根据ES5规范，如果`new Object(value)`中检测到`Value`的类型为`object`就直接返回该对象而不会创建一个新对象。

- 其中构造函数做了什么呢，首先明确 构造函数不是一定要和new关键字一起使用的，当构造函数做为函数单独调用时，做了一些不同的事，比如内置构造函数如果不和new一起使用则将参数做一次类型转换，但不一定创建一个新对象，至于具体的差别，推荐食用http://yanhaijing.com/es5/#334

#### 补充：new运算符优先级

| 优先级 | 运算类型           | 关联性   | 例子            |
| :----- | :----------------- | :------- | :-------------- |
| 20     | 圆括号             | n/a      | （a + b) * c    |
| 19     | 成员访问           | 从左到右 | object.method   |
| 19     | 需要计算的成员访问 | 从左到右 | object[“a”+”b”] |
| 19     | new 带参数列表     | n/a      | new fun()       |
| 19     | 函数调用           | 从左到右 | fun()           |
| 18     | new 无参数列表     | 从右到左 | new fun         |

思考：`new Foo().getName()`和`new Foo.getName()`两个表达式中的运算优先级

我们知道，构造函数没有参数列表的时候是可以省略括号的 也就是 `new Foo()` 等同于 `new Foo`，根据上表，优先级越高的先执行:

`new Foo().getName()`表达式中，new 带参数列表优先级高于Foo()函数调用表达式，先执行`new Foo()`，即表达式等同于`(new Foo()).getName()`

`new Foo.getName()`表达式中，成员访问表达式优先级高于`new` 无带参列表，即表达式等同于`new (Foo.getName())`



#### Object.create()函数

该函数接受两个参数，并且返回一个新创建的对象，并且将第一个参数作为新创建的对象的原型，甚至可以传入`null`来创建一个没有原型的空对象，这样创建的空对象将不继承任何基础方法，比如`toString`，这意味着这样创建的对象将无法和`+`一起正常工作。



### 对象属性与属性继承

#### 对象属性

对象的三个属性 ：原型属性、类属性、可扩展性。

##### 原型

对象有自有属性和继承属性，其中原型属性就是作为继承属性来使用的。通过`new`创建的对象用构造函数的`prototype`属性作为对象的原型，通过`Object.create()`创建的对象使用第一个参数作为创建对象的原型，没有原型的对象为数不多，其中包括`Object.prototype`和`Object.create(null)`。

可以用`a.isPrototypeOf(b)`方法判断`a`是否是`b`原型，即`b`是否继承自`a`.

可以用`Object.getPrototypeOf(a)`来获取`a`对象的原型，若`a`不是对象类型则抛出类型错误。

##### 类

通常和构造函数的名称保持一致，通过内置构造函数创建的对象有类名，并且可以通过类似`Object.prototype.toString.call(new Date())`的方法来获得类名，而自定义的对象没有类名，因为类属性一定为“`Object`”。

##### 扩展性

宿主对象的可扩展性由js引擎决定（任何对象，不是原生对象就是宿主对象），ES5中，所有内置对象和自定义对象都是可扩展的。除非将其转换为不可扩展的。

可以使用`Object.isExtensible()`来检测对象是否可扩展，使用`Object.preventExtensions()`来将对象转换为不可扩展的。一旦将对象转换为不可扩展的就无法再转换为可扩展的。不可扩展只对于对象的自有属性，如果对象的原型扩展了方法那么该对象将仍然继承该方法。

#### 补充

##### 存取器属性getter/setter

如果一个属性同时具有`getter/setter`方法，则该属性具有读/写性，如果只有`getter`方法，则是只读属性，如果只有`setter`方法，则是只写属性。

存取器属性是可继承的，使用方法如下实例：

```javascript
var o = {
  x: 1,
  get y(){return this.x},
  set y(value){this.x = value}
}; //{x: 1, y: 1}
o.y = 2; //{x: 2, y: 2}

var o = {x: 1, get y(){return this.y}}
o.y //RangeError: Maximum call stack size exceeded 读取器里读取它自己无限回调

var o = {x: 1, set y(value){return value}}
o.y = 3;
o.y // undefined 读取只写属性永远返回undefined

var o = {x: 1, set y(){return value}} //Setter must have exactly one formal parameter
```



### 属性特性与相关API总结

由上述可知，`getter/setter`存取器属性与属性的可读可写性密切相关，所以可视为属性的特性。

普通属性的四个特性：值、可写性、可枚举型、可配置性。分别对应{value, writable, enumerable, configurable}

读取器属性的四个特性：读取、写入、可枚举性、可配置型。分别对应{get, set, enmurable, configurable }

可以通过`Object.getOwnPropertyDescriptor({x: 1}, x)`查看对象特定**自有**属性特性。如果要查看继承属性，需要遍历原型链`Object.getPrototypeOf()`

可以通过`Object.definedProperty(o, "x", {writable: false})`新建或修改某对象特定自有属性的特性。对于新建的属性来说，第三个参数中不存在的特性将被描述为`false`或`undefined`，对于修改的属性来说，第三个参数中不存在的特性将不会被修改。

可以通过`Object.seal()`将对象设置为封闭的，即不可扩展的以及将对象属性设置为不可配置的，相对于`Object.preventExtensions()`方法，`Object.seal()`方法处理的对象将不能删除和配置已有属性，但可写属性依然可以修改。封闭的对象将不能解封，可以用`Object.isSealed()`方法检测对象是否是封闭的 

可以通过`Object.freeze()`方法将对象冻结，即不可扩展的以及将对象属性设置为不可配置的还将所有数据属性设置为只读的（对`setter`属性无效），可以使用`Object.isFrozen()`检测对象是否冻结。





### 面试题练习

1. 写出下面代码的执行结果

```javascript
function Foo() {
  getName = function() {
    console.log(1);
  }
  return this;
}
Foo.getName = function() {
  console.log(2);
}
Foo.prototype.getName = function() {
  console.log(3);
}
var getName = function() {
  console.log(4);
}
function getName() {
  console.log(5);
}
Foo.getName();
getName();
Foo().getName();
getName();
new Foo.getName();
new Foo().getName();
new new Foo().getName();
```

2. 写出下面代码的执行结果

```javascript
var A = function() {};
A.prototype.n = 1;
var b = new A();
A.prototype = {
  n: 2,
  m: 3
}
var c = new A();

console.log(b.n);
console.log(b.m);

console.log(c.n);
console.log(c.m);
```

3. 写出下面代码的执行结果

```javascript
var F = function() {};

Object.prototype.a = function() {
  console.log('a');
};

Function.prototype.b = function() {
  console.log('b');
}

var f = new F();

f.a();
f.b();

F.a();
F.b();
```

4. 回答以下两个问题

```javascript
function Person(name) {
    this.name = name
}
let p = new Person('Tom');

//问题1：1. p.__proto__等于什么？

//问题2：Person.__proto__等于什么？
```

5. 写出以下代码的执行结果

```javascript
var foo = {},
    F = function(){};
Object.prototype.a = 'value a';
Function.prototype.b = 'value b';

console.log(foo.a);
console.log(foo.b);

console.log(F.a);
console.log(F.b);
```





------





#### 解析

1. 本题考察的知识点很多，包括函数声明提前，原型链，执行上下文this，因此放在了本篇。

   - Foo.getName() Foo对象上有getName属性，直接调用执行输出2

   - getName()  这里考察声明提前， 函数声明提前但函数定义表达式不提前，因此这一段实际被编译为:

     ```javascript
     var getName;
     function getName(){
       console.log(5);
     }
     getName = function(){
       console.log(4);
     }
     getName();
     ```

     因此输出4

   - Foo().getName()  F()给一个未声明的变量getName赋值了一个函数，实际创建了一个同名的全局对象属性getName并赋值为function(){console.log(1)}，又因为函数是普通调用，没有绑定在对象上或实例上，返回的this即window，调用window.getName() 输出1

   - Foo()创建的全局getName属性覆盖了定义的函数声明，输出1

   - new Foo.getName()  注意运算优先级 先计算Foo.getName() 输出 2 

   - new Foo().getName()  注意运算优先级，先执行new Foo()，new Foo()创建一个新对象，对象关联到 Foo.prototype，返回以新创建的对象为上下文的this，因此调用Foo.prototype.getName() 输出3

   - new new Foo().getName()  执行顺序new  ( (new Foo()).getName()) 同上输出3

2. 本题考察了原型链。

   ```javascript
   var A = function() {};
   A.prototype = {constructor: function(){}};
   A.prototype = {constructor: function(){}, n: 1};
   var b = new A();
   b._proto_ = A.prototype;  
   b._proto_ = {constructor: function(){}, n: 1};
   
   A.prototype = {
     n: 2,
     m: 3
   };
   var c = new A();
   c._proto_ = A.prototype = {
     n: 2,
     m: 3
   };
   
   b.n = 1; b.m = undefined;
   c.n = 2; c.m = 3;
   ```

3. 考察原型链

   ```javascript
   F.prototype = {contructor: function(){}};
   Object.prototype.a = function() {
     console.log('a');
   };
   Function.prototype.b = function() {
     console.log('b');
   }
   f._proto_ = F.prototype;
   f.a => f._proto_.a => F.prototype.a => F.prototype._proto_.a => Object.prototype.a
   f.a()//'a'
   
   f.b => f._proto_.b => F.prototype.b => F.prototype._proto_.b =>
   Object.prototype.b
   f.b()// TypeError undefined is not a function
   
   F.a => F._proto_.a => Function.prototype.a => Function.prototype._proto_.a => Object.prototype.a
   F.a() // 'a'
   
   F.b => F._proto_.b => Function.prototype.b
   F.b() //'b'
   ```

4. (1) `p._proto_ = Person.prototype;`

   (2)`Person._proto_ = Function.prototype;`

5. 如下：

```javascript
foo.a => foo._proto_.a => Object.prototype.a => 'value a'
foo.b => foo._proto_.b => Object.prototype.b => undefined

F.a => Foo._proto_.a => Function.prototype.a => Function.prototype._proto_.a => Object.prototype.a => 'value a'
F.b => Foo._proto_.b => Function.prototype.b => 'value b' 
```

> 引言

最近比较忙导致这篇拖了好久啊，第二篇的作用域和闭包因为其中一部分没搞得很清楚也很难受，决定不和自己钻牛角尖了，本篇最后的面试题部分会包含一部分闭包的知识点以弥补上篇没讲清和讲的不够详细的知识点。

本篇对标犀牛书第6大章和第9大章





为什么叫大Object，事实上JS将它单独作为一个基本数据类型应该就足以称之为**大**了，也够复杂。撰写本篇的初心还是想搞清楚原型和原型链所以放在最前面，只是在看的过程中发现和相关的知识点很成体系以及也比较重要，所以都总结记录了一下，可以作为补充看。

### 原型和原型链

#### `_proto_`、`prototype`和`constructor`属性

每个JS对象（`null`除外）都自动拥有一个`_proto_`属性，这个属性是一个对象，指向该对象的原型

每个JS函数（`bind()`方法除外）都自动拥有一个`prototype`属性，这个属性也是一个对象，用该函数做构造函数创建的对象将继承这个`prototype`的属性，也就是说理论上任何一个JS函数都可以用作构造函数，并且调用构造函数需要用到`prototype`属性。

`prototype`属性包含一个唯一不可枚举的属性`constructor`，这个属性是一个函数对象，指向该函数的构造函数。

看完上述两点你可能会认为`_proto_`是不是就是对象的原型，`prototype`是不是就是函数的原型呢？ 事实上并不是，但我们可以说对象的`_proto_`属性指向它的原型，构造函数的`prototype`属性指向调用构造函数创建的实例的原型。

这么说有点绕，用图片（来源：https://github.com/mqyqingfeng/Blog/issues/2  侵删）表示即为：

![实例原型与构造函数的关系图](https://github.com/mqyqingfeng/Blog/raw/master/Images/prototype3.png)

综上所述，如果我们有一段代码：

```javascript
function Person(name, age){
  this.name = name;
  this.age = age;
}
let person = new Person('xiao hong', 18);
```

可以得到：

```javascript
person._proto_ === Person.prototype;
Person.prototype.constructor === Person;
```



#### 原型链

我们知道，当执行属性访问表达式时，首先会将表达式的操作主题转化为对象，然后去对象中查找属性，如果找不到就去找与对象的原型中的属性，如果还找不到，就继续查找原型的原型，直到找到最顶层为止。

那么原型的原型是什么？我们知道，`_proto_`和`prototype`属性也只是普通的对象而已，既然是对象，就也有`_proto_`属性，一个普通对象的`_proto_`属性自然指向其构造函数Obejct的`prototype`属性，即`Object.prototype`

那`Object.prototype`的原型呢？我们可以打印一下:

```javascript
console.log(Object.prototype._proto_) // null
```

所以，当我们向上查找属性的时候，查到`Object.prototype`就可以停止了，由这些对象相互关联的原型之间的关系就是原型链。到这里，我们的图片来源：https://github.com/mqyqingfeng/Blog/issues/2  侵删）也可以更新为：

![原型链示意图](https://github.com/mqyqingfeng/Blog/raw/master/Images/prototype5.png)

综上所述，如果我们有一段代码：

```javascript
function Person(name, age){
  this.name = name;
  this.age = age;
}
Person.prototype = {
  getName: function(){return this.name}
}
let person = new Person('xiao hong', 18);
person.toString();
```

我们在person上找不到toString方法，就会去person的原型上找，`person._proto_ === Person.prototype`，结果`Person.prototype`上也没有这个方法，就会再去原型的原型上找即`Person.prototype._proto_`， 要知道`Person.prototype._proto_`只是一个普通的对象，可以被原始的`new Object()`创建，所以`Person.prototype._proto === Object.prototype`，所幸`Object.prototype`上有`toString()`方法，因此调用它，查找到此结束。

#### 补充

1. `constructor`属性

不是每个对象都有`constructor`属性，因此不是每个对象都可以用作构造函数的`prototype`属性对象，但可以显示的定义`constructor`属性反向引用构造函数来修正这个问题。

`constructor`属性也是普通的对象属性，如果找不到改属性，也会从对象原型上继续寻找。如下：

```javascript
function Person() {

}
var person = new Person();
console.log(person.constructor === Person); // true
```

当获取 `person.constructor` 时，其实 `person` 中并没有 `constructor` 属性,当不能读取到`constructor` 属性时，会从 `person` 的原型也就是 `Person.prototype` 中读取，正好原型中有该属性，所以：

```javascript
person.constructor === Person.prototype.constructor
```

2. `_proto_`

其次是 `__proto__` ，绝大部分浏览器都支持这个非标准的方法访问原型，然而它并不存在于` Person.prototype `中，实际上，它是来自于 `Object.prototype` ，与其说是一个属性，不如说是一个 `getter/setter`，当使用` obj.__proto`__ 时，可以理解成返回了 `Object.getPrototypeOf(obj)`。

3. `Function._proto_ === Function.prototype`

这里并不是因为`Function`是对象，有`_proto_`属性，`Function`又是函数，函数对象的原型指向其构造函数`Function.prototype`，这样理解虽然看上去很正确但是是不对的。引用大佬的话:

> Function.prototype是引擎创造出来的对象，一开始就有了，又因为其他的构造函数都可以通过原型链找到Function.prototype，Function本身也是一个构造函数，为了不产生混乱，就将这两个联系到一起了

4. `Object.__proto__  === Function.prototype`

`Object`是对象的构造函数，那么它也是一个函数，当然它的`__proto__`也是指向`Function.prototype`



实际上原型原型链就是这样，但如果你觉得上述还是很难理解，最好再理解一系列的概念，以下内容可以作为作为对原型和继承的补充理解：

### 对象创建与构造函数

对象的三种创建方法：

#### 对象直接量 

例如`let o = {}`，对象直接量是一个表达式，这个表达式的每次运算都会创建并初始化一个新的对象，也就是说在一个函数中使用对象直接量会函数重复调用时创建很多新对象

#### new+构造函数创建对象

例如`let o =  new Object()`

- 其中`new`关键字做了什么呢，根据MDN的介绍

![image-20191026185858754](/Users/chenxingjian/Library/Application Support/typora-user-images/image-20191026185858754.png)其中第2点即，设置创建的新对象的原型与构造函数的`prototype`属性相关联。

注意：`new`关键字也不是一定创建一个新对象的，例如：`new Object({})`， 根据ES5规范，如果`new Object(value)`中检测到`Value`的类型为`object`就直接返回该对象而不会创建一个新对象。

- 其中构造函数做了什么呢，首先明确 构造函数不是一定要和new关键字一起使用的，当构造函数做为函数单独调用时，做了一些不同的事，比如内置构造函数如果不和new一起使用则将参数做一次类型转换，但不一定创建一个新对象，至于具体的差别，推荐食用http://yanhaijing.com/es5/#334

#### 补充：new运算符优先级

| 优先级 | 运算类型           | 关联性   | 例子            |
| :----- | :----------------- | :------- | :-------------- |
| 20     | 圆括号             | n/a      | （a + b) * c    |
| 19     | 成员访问           | 从左到右 | object.method   |
| 19     | 需要计算的成员访问 | 从左到右 | object[“a”+”b”] |
| 19     | new 带参数列表     | n/a      | new fun()       |
| 19     | 函数调用           | 从左到右 | fun()           |
| 18     | new 无参数列表     | 从右到左 | new fun         |

思考：`new Foo().getName()`和`new Foo.getName()`两个表达式中的运算优先级

我们知道，构造函数没有参数列表的时候是可以省略括号的 也就是 `new Foo()` 等同于 `new Foo`，根据上表，优先级越高的先执行:

`new Foo().getName()`表达式中，new 带参数列表优先级高于Foo()函数调用表达式，先执行`new Foo()`，即表达式等同于`(new Foo()).getName()`

`new Foo.getName()`表达式中，成员访问表达式优先级高于`new` 无带参列表，即表达式等同于`new (Foo.getName())`



#### Object.create()函数

该函数接受两个参数，并且返回一个新创建的对象，并且将第一个参数作为新创建的对象的原型，甚至可以传入`null`来创建一个没有原型的空对象，这样创建的空对象将不继承任何基础方法，比如`toString`，这意味着这样创建的对象将无法和`+`一起正常工作。



### 对象属性与属性继承

#### 对象属性

对象的三个属性 ：原型属性、类属性、可扩展性。

##### 原型

对象有自有属性和继承属性，其中原型属性就是作为继承属性来使用的。通过`new`创建的对象用构造函数的`prototype`属性作为对象的原型，通过`Object.create()`创建的对象使用第一个参数作为创建对象的原型，没有原型的对象为数不多，其中包括`Object.prototype`和`Object.create(null)`。

可以用`a.isPrototypeOf(b)`方法判断`a`是否是`b`原型，即`b`是否继承自`a`.

可以用`Object.getPrototypeOf(a)`来获取`a`对象的原型，若`a`不是对象类型则抛出类型错误。

##### 类

通常和构造函数的名称保持一致，通过内置构造函数创建的对象有类名，并且可以通过类似`Object.prototype.toString.call(new Date())`的方法来获得类名，而自定义的对象没有类名，因为类属性一定为“`Object`”。

##### 扩展性

宿主对象的可扩展性由js引擎决定（任何对象，不是原生对象就是宿主对象），ES5中，所有内置对象和自定义对象都是可扩展的。除非将其转换为不可扩展的。

可以使用`Object.isExtensible()`来检测对象是否可扩展，使用`Object.preventExtensions()`来将对象转换为不可扩展的。一旦将对象转换为不可扩展的就无法再转换为可扩展的。不可扩展只对于对象的自有属性，如果对象的原型扩展了方法那么该对象将仍然继承该方法。

#### 补充

##### 存取器属性getter/setter

如果一个属性同时具有`getter/setter`方法，则该属性具有读/写性，如果只有`getter`方法，则是只读属性，如果只有`setter`方法，则是只写属性。

存取器属性是可继承的，使用方法如下实例：

```javascript
var o = {
  x: 1,
  get y(){return this.x},
  set y(value){this.x = value}
}; //{x: 1, y: 1}
o.y = 2; //{x: 2, y: 2}

var o = {x: 1, get y(){return this.y}}
o.y //RangeError: Maximum call stack size exceeded 读取器里读取它自己无限回调

var o = {x: 1, set y(value){return value}}
o.y = 3;
o.y // undefined 读取只写属性永远返回undefined

var o = {x: 1, set y(){return value}} //Setter must have exactly one formal parameter
```



### 属性特性与相关API总结

由上述可知，`getter/setter`存取器属性与属性的可读可写性密切相关，所以可视为属性的特性。

普通属性的四个特性：值、可写性、可枚举型、可配置性。分别对应{value, writable, enumerable, configurable}

读取器属性的四个特性：读取、写入、可枚举性、可配置型。分别对应{get, set, enmurable, configurable }

可以通过`Object.getOwnPropertyDescriptor({x: 1}, x)`查看对象特定**自有**属性特性。如果要查看继承属性，需要遍历原型链`Object.getPrototypeOf()`

可以通过`Object.definedProperty(o, "x", {writable: false})`新建或修改某对象特定自有属性的特性。对于新建的属性来说，第三个参数中不存在的特性将被描述为`false`或`undefined`，对于修改的属性来说，第三个参数中不存在的特性将不会被修改。

可以通过`Object.seal()`将对象设置为封闭的，即不可扩展的以及将对象属性设置为不可配置的，相对于`Object.preventExtensions()`方法，`Object.seal()`方法处理的对象将不能删除和配置已有属性，但可写属性依然可以修改。封闭的对象将不能解封，可以用`Object.isSealed()`方法检测对象是否是封闭的 

可以通过`Object.freeze()`方法将对象冻结，即不可扩展的以及将对象属性设置为不可配置的还将所有数据属性设置为只读的（对`setter`属性无效），可以使用`Object.isFrozen()`检测对象是否冻结。





### 面试题练习

1. 写出下面代码的执行结果

```javascript
function Foo() {
  getName = function() {
    console.log(1);
  }
  return this;
}
Foo.getName = function() {
  console.log(2);
}
Foo.prototype.getName = function() {
  console.log(3);
}
var getName = function() {
  console.log(4);
}
function getName() {
  console.log(5);
}
Foo.getName();
getName();
Foo().getName();
getName();
new Foo.getName();
new Foo().getName();
new new Foo().getName();
```

2. 写出下面代码的执行结果

```javascript
var A = function() {};
A.prototype.n = 1;
var b = new A();
A.prototype = {
  n: 2,
  m: 3
}
var c = new A();

console.log(b.n);
console.log(b.m);

console.log(c.n);
console.log(c.m);
```

3. 写出下面代码的执行结果

```javascript
var F = function() {};

Object.prototype.a = function() {
  console.log('a');
};

Function.prototype.b = function() {
  console.log('b');
}

var f = new F();

f.a();
f.b();

F.a();
F.b();
```

4. 回答以下两个问题

```javascript
function Person(name) {
    this.name = name
}
let p = new Person('Tom');

//问题1：1. p.__proto__等于什么？

//问题2：Person.__proto__等于什么？
```

5. 写出以下代码的执行结果

```javascript
var foo = {},
    F = function(){};
Object.prototype.a = 'value a';
Function.prototype.b = 'value b';

console.log(foo.a);
console.log(foo.b);

console.log(F.a);
console.log(F.b);
```





------





#### 解析

1. 本题考察的知识点很多，包括函数声明提前，原型链，执行上下文this，因此放在了本篇。

   - Foo.getName() Foo对象上有getName属性，直接调用执行输出2

   - getName()  这里考察声明提前， 函数声明提前但函数定义表达式不提前，因此这一段实际被编译为:

     ```javascript
     var getName;
     function getName(){
       console.log(5);
     }
     getName = function(){
       console.log(4);
     }
     getName();
     ```

     因此输出4

   - Foo().getName()  F()给一个未声明的变量getName赋值了一个函数，实际创建了一个同名的全局对象属性getName并赋值为function(){console.log(1)}，又因为函数是普通调用，没有绑定在对象上或实例上，返回的this即window，调用window.getName() 输出1

   - Foo()创建的全局getName属性覆盖了定义的函数声明，输出1

   - new Foo.getName()  注意运算优先级 先计算Foo.getName() 输出 2 

   - new Foo().getName()  注意运算优先级，先执行new Foo()，new Foo()创建一个新对象，对象关联到 Foo.prototype，返回以新创建的对象为上下文的this，因此调用Foo.prototype.getName() 输出3

   - new new Foo().getName()  执行顺序new  ( (new Foo()).getName()) 同上输出3

2. 本题考察了原型链。

   ```javascript
   var A = function() {};
   A.prototype = {constructor: function(){}};
   A.prototype = {constructor: function(){}, n: 1};
   var b = new A();
   b._proto_ = A.prototype;  
   b._proto_ = {constructor: function(){}, n: 1};
   
   A.prototype = {
     n: 2,
     m: 3
   };
   var c = new A();
   c._proto_ = A.prototype = {
     n: 2,
     m: 3
   };
   
   b.n = 1; b.m = undefined;
   c.n = 2; c.m = 3;
   ```

3. 考察原型链

   ```javascript
   F.prototype = {contructor: function(){}};
   Object.prototype.a = function() {
     console.log('a');
   };
   Function.prototype.b = function() {
     console.log('b');
   }
   f._proto_ = F.prototype;
   f.a => f._proto_.a => F.prototype.a => F.prototype._proto_.a => Object.prototype.a
   f.a()//'a'
   
   f.b => f._proto_.b => F.prototype.b => F.prototype._proto_.b =>
   Object.prototype.b
   f.b()// TypeError undefined is not a function
   
   F.a => F._proto_.a => Function.prototype.a => Function.prototype._proto_.a => Object.prototype.a
   F.a() // 'a'
   
   F.b => F._proto_.b => Function.prototype.b
   F.b() //'b'
   ```

4. (1) `p._proto_ = Person.prototype;`

   (2)`Person._proto_ = Function.prototype;`

5. 如下：

```javascript
foo.a => foo._proto_.a => Object.prototype.a => 'value a'
foo.b => foo._proto_.b => Object.prototype.b => undefined

F.a => Foo._proto_.a => Function.prototype.a => Function.prototype._proto_.a => Object.prototype.a => 'value a'
F.b => Foo._proto_.b => Function.prototype.b => 'value b' 
```

> 引言

最近比较忙导致这篇拖了好久啊，第二篇的作用域和闭包因为其中一部分没搞得很清楚也很难受，决定不和自己钻牛角尖了，本篇最后的面试题部分会包含一部分闭包的知识点以弥补上篇没讲清和讲的不够详细的知识点。

本篇对标犀牛书第6大章和第9大章





为什么叫大Object，事实上JS将它单独作为一个基本数据类型应该就足以称之为**大**了，也够复杂。撰写本篇的初心还是想搞清楚原型和原型链所以放在最前面，只是在看的过程中发现和相关的知识点很成体系以及也比较重要，所以都总结记录了一下，可以作为补充看。

### 原型和原型链

#### `_proto_`、`prototype`和`constructor`属性

每个JS对象（`null`除外）都自动拥有一个`_proto_`属性，这个属性是一个对象，指向该对象的原型

每个JS函数（`bind()`方法除外）都自动拥有一个`prototype`属性，这个属性也是一个对象，用该函数做构造函数创建的对象将继承这个`prototype`的属性，也就是说理论上任何一个JS函数都可以用作构造函数，并且调用构造函数需要用到`prototype`属性。

`prototype`属性包含一个唯一不可枚举的属性`constructor`，这个属性是一个函数对象，指向该函数的构造函数。

看完上述两点你可能会认为`_proto_`是不是就是对象的原型，`prototype`是不是就是函数的原型呢？ 事实上并不是，但我们可以说对象的`_proto_`属性指向它的原型，构造函数的`prototype`属性指向调用构造函数创建的实例的原型。

这么说有点绕，用图片（来源：https://github.com/mqyqingfeng/Blog/issues/2  侵删）表示即为：

![实例原型与构造函数的关系图](https://github.com/mqyqingfeng/Blog/raw/master/Images/prototype3.png)

综上所述，如果我们有一段代码：

```javascript
function Person(name, age){
  this.name = name;
  this.age = age;
}
let person = new Person('xiao hong', 18);
```

可以得到：

```javascript
person._proto_ === Person.prototype;
Person.prototype.constructor === Person;
```



#### 原型链

我们知道，当执行属性访问表达式时，首先会将表达式的操作主题转化为对象，然后去对象中查找属性，如果找不到就去找与对象的原型中的属性，如果还找不到，就继续查找原型的原型，直到找到最顶层为止。

那么原型的原型是什么？我们知道，`_proto_`和`prototype`属性也只是普通的对象而已，既然是对象，就也有`_proto_`属性，一个普通对象的`_proto_`属性自然指向其构造函数Obejct的`prototype`属性，即`Object.prototype`

那`Object.prototype`的原型呢？我们可以打印一下:

```javascript
console.log(Object.prototype._proto_) // null
```

所以，当我们向上查找属性的时候，查到`Object.prototype`就可以停止了，由这些对象相互关联的原型之间的关系就是原型链。到这里，我们的图片来源：https://github.com/mqyqingfeng/Blog/issues/2  侵删）也可以更新为：

![原型链示意图](https://github.com/mqyqingfeng/Blog/raw/master/Images/prototype5.png)

综上所述，如果我们有一段代码：

```javascript
function Person(name, age){
  this.name = name;
  this.age = age;
}
Person.prototype = {
  getName: function(){return this.name}
}
let person = new Person('xiao hong', 18);
person.toString();
```

我们在person上找不到toString方法，就会去person的原型上找，`person._proto_ === Person.prototype`，结果`Person.prototype`上也没有这个方法，就会再去原型的原型上找即`Person.prototype._proto_`， 要知道`Person.prototype._proto_`只是一个普通的对象，可以被原始的`new Object()`创建，所以`Person.prototype._proto === Object.prototype`，所幸`Object.prototype`上有`toString()`方法，因此调用它，查找到此结束。

#### 补充

1. `constructor`属性

不是每个对象都有`constructor`属性，因此不是每个对象都可以用作构造函数的`prototype`属性对象，但可以显示的定义`constructor`属性反向引用构造函数来修正这个问题。

`constructor`属性也是普通的对象属性，如果找不到改属性，也会从对象原型上继续寻找。如下：

```javascript
function Person() {

}
var person = new Person();
console.log(person.constructor === Person); // true
```

当获取 `person.constructor` 时，其实 `person` 中并没有 `constructor` 属性,当不能读取到`constructor` 属性时，会从 `person` 的原型也就是 `Person.prototype` 中读取，正好原型中有该属性，所以：

```javascript
person.constructor === Person.prototype.constructor
```

2. `_proto_`

其次是 `__proto__` ，绝大部分浏览器都支持这个非标准的方法访问原型，然而它并不存在于` Person.prototype `中，实际上，它是来自于 `Object.prototype` ，与其说是一个属性，不如说是一个 `getter/setter`，当使用` obj.__proto`__ 时，可以理解成返回了 `Object.getPrototypeOf(obj)`。

3. `Function._proto_ === Function.prototype`

这里并不是因为`Function`是对象，有`_proto_`属性，`Function`又是函数，函数对象的原型指向其构造函数`Function.prototype`，这样理解虽然看上去很正确但是是不对的。引用大佬的话:

> Function.prototype是引擎创造出来的对象，一开始就有了，又因为其他的构造函数都可以通过原型链找到Function.prototype，Function本身也是一个构造函数，为了不产生混乱，就将这两个联系到一起了

4. `Object.__proto__  === Function.prototype`

`Object`是对象的构造函数，那么它也是一个函数，当然它的`__proto__`也是指向`Function.prototype`



实际上原型原型链就是这样，但如果你觉得上述还是很难理解，最好再理解一系列的概念，以下内容可以作为作为对原型和继承的补充理解：

### 对象创建与构造函数

对象的三种创建方法：

#### 对象直接量 

例如`let o = {}`，对象直接量是一个表达式，这个表达式的每次运算都会创建并初始化一个新的对象，也就是说在一个函数中使用对象直接量会函数重复调用时创建很多新对象

#### new+构造函数创建对象

例如`let o =  new Object()`

- 其中`new`关键字做了什么呢，根据MDN的介绍

![image-20191026185858754](/Users/chenxingjian/Library/Application Support/typora-user-images/image-20191026185858754.png)其中第2点即，设置创建的新对象的原型与构造函数的`prototype`属性相关联。

注意：`new`关键字也不是一定创建一个新对象的，例如：`new Object({})`， 根据ES5规范，如果`new Object(value)`中检测到`Value`的类型为`object`就直接返回该对象而不会创建一个新对象。

- 其中构造函数做了什么呢，首先明确 构造函数不是一定要和new关键字一起使用的，当构造函数做为函数单独调用时，做了一些不同的事，比如内置构造函数如果不和new一起使用则将参数做一次类型转换，但不一定创建一个新对象，至于具体的差别，推荐食用http://yanhaijing.com/es5/#334

#### 补充：new运算符优先级

| 优先级 | 运算类型           | 关联性   | 例子            |
| :----- | :----------------- | :------- | :-------------- |
| 20     | 圆括号             | n/a      | （a + b) * c    |
| 19     | 成员访问           | 从左到右 | object.method   |
| 19     | 需要计算的成员访问 | 从左到右 | object[“a”+”b”] |
| 19     | new 带参数列表     | n/a      | new fun()       |
| 19     | 函数调用           | 从左到右 | fun()           |
| 18     | new 无参数列表     | 从右到左 | new fun         |

思考：`new Foo().getName()`和`new Foo.getName()`两个表达式中的运算优先级

我们知道，构造函数没有参数列表的时候是可以省略括号的 也就是 `new Foo()` 等同于 `new Foo`，根据上表，优先级越高的先执行:

`new Foo().getName()`表达式中，new 带参数列表优先级高于Foo()函数调用表达式，先执行`new Foo()`，即表达式等同于`(new Foo()).getName()`

`new Foo.getName()`表达式中，成员访问表达式优先级高于`new` 无带参列表，即表达式等同于`new (Foo.getName())`



#### Object.create()函数

该函数接受两个参数，并且返回一个新创建的对象，并且将第一个参数作为新创建的对象的原型，甚至可以传入`null`来创建一个没有原型的空对象，这样创建的空对象将不继承任何基础方法，比如`toString`，这意味着这样创建的对象将无法和`+`一起正常工作。



### 对象属性与属性继承

#### 对象属性

对象的三个属性 ：原型属性、类属性、可扩展性。

##### 原型

对象有自有属性和继承属性，其中原型属性就是作为继承属性来使用的。通过`new`创建的对象用构造函数的`prototype`属性作为对象的原型，通过`Object.create()`创建的对象使用第一个参数作为创建对象的原型，没有原型的对象为数不多，其中包括`Object.prototype`和`Object.create(null)`。

可以用`a.isPrototypeOf(b)`方法判断`a`是否是`b`原型，即`b`是否继承自`a`.

可以用`Object.getPrototypeOf(a)`来获取`a`对象的原型，若`a`不是对象类型则抛出类型错误。

##### 类

通常和构造函数的名称保持一致，通过内置构造函数创建的对象有类名，并且可以通过类似`Object.prototype.toString.call(new Date())`的方法来获得类名，而自定义的对象没有类名，因为类属性一定为“`Object`”。

##### 扩展性

宿主对象的可扩展性由js引擎决定（任何对象，不是原生对象就是宿主对象），ES5中，所有内置对象和自定义对象都是可扩展的。除非将其转换为不可扩展的。

可以使用`Object.isExtensible()`来检测对象是否可扩展，使用`Object.preventExtensions()`来将对象转换为不可扩展的。一旦将对象转换为不可扩展的就无法再转换为可扩展的。不可扩展只对于对象的自有属性，如果对象的原型扩展了方法那么该对象将仍然继承该方法。

#### 补充

##### 存取器属性getter/setter

如果一个属性同时具有`getter/setter`方法，则该属性具有读/写性，如果只有`getter`方法，则是只读属性，如果只有`setter`方法，则是只写属性。

存取器属性是可继承的，使用方法如下实例：

```javascript
var o = {
  x: 1,
  get y(){return this.x},
  set y(value){this.x = value}
}; //{x: 1, y: 1}
o.y = 2; //{x: 2, y: 2}

var o = {x: 1, get y(){return this.y}}
o.y //RangeError: Maximum call stack size exceeded 读取器里读取它自己无限回调

var o = {x: 1, set y(value){return value}}
o.y = 3;
o.y // undefined 读取只写属性永远返回undefined

var o = {x: 1, set y(){return value}} //Setter must have exactly one formal parameter
```



### 属性特性与相关API总结

由上述可知，`getter/setter`存取器属性与属性的可读可写性密切相关，所以可视为属性的特性。

普通属性的四个特性：值、可写性、可枚举型、可配置性。分别对应{value, writable, enumerable, configurable}

读取器属性的四个特性：读取、写入、可枚举性、可配置型。分别对应{get, set, enmurable, configurable }

可以通过`Object.getOwnPropertyDescriptor({x: 1}, x)`查看对象特定**自有**属性特性。如果要查看继承属性，需要遍历原型链`Object.getPrototypeOf()`

可以通过`Object.definedProperty(o, "x", {writable: false})`新建或修改某对象特定自有属性的特性。对于新建的属性来说，第三个参数中不存在的特性将被描述为`false`或`undefined`，对于修改的属性来说，第三个参数中不存在的特性将不会被修改。

可以通过`Object.seal()`将对象设置为封闭的，即不可扩展的以及将对象属性设置为不可配置的，相对于`Object.preventExtensions()`方法，`Object.seal()`方法处理的对象将不能删除和配置已有属性，但可写属性依然可以修改。封闭的对象将不能解封，可以用`Object.isSealed()`方法检测对象是否是封闭的 

可以通过`Object.freeze()`方法将对象冻结，即不可扩展的以及将对象属性设置为不可配置的还将所有数据属性设置为只读的（对`setter`属性无效），可以使用`Object.isFrozen()`检测对象是否冻结。





### 面试题练习

1. 写出下面代码的执行结果

```javascript
function Foo() {
  getName = function() {
    console.log(1);
  }
  return this;
}
Foo.getName = function() {
  console.log(2);
}
Foo.prototype.getName = function() {
  console.log(3);
}
var getName = function() {
  console.log(4);
}
function getName() {
  console.log(5);
}
Foo.getName();
getName();
Foo().getName();
getName();
new Foo.getName();
new Foo().getName();
new new Foo().getName();
```

2. 写出下面代码的执行结果

```javascript
var A = function() {};
A.prototype.n = 1;
var b = new A();
A.prototype = {
  n: 2,
  m: 3
}
var c = new A();

console.log(b.n);
console.log(b.m);

console.log(c.n);
console.log(c.m);
```

3. 写出下面代码的执行结果

```javascript
var F = function() {};

Object.prototype.a = function() {
  console.log('a');
};

Function.prototype.b = function() {
  console.log('b');
}

var f = new F();

f.a();
f.b();

F.a();
F.b();
```

4. 回答以下两个问题

```javascript
function Person(name) {
    this.name = name
}
let p = new Person('Tom');

//问题1：1. p.__proto__等于什么？

//问题2：Person.__proto__等于什么？
```

5. 写出以下代码的执行结果

```javascript
var foo = {},
    F = function(){};
Object.prototype.a = 'value a';
Function.prototype.b = 'value b';

console.log(foo.a);
console.log(foo.b);

console.log(F.a);
console.log(F.b);
```





------





#### 解析

1. 本题考察的知识点很多，包括函数声明提前，原型链，执行上下文this，因此放在了本篇。

   - Foo.getName() Foo对象上有getName属性，直接调用执行输出2

   - getName()  这里考察声明提前， 函数声明提前但函数定义表达式不提前，因此这一段实际被编译为:

     ```javascript
     var getName;
     function getName(){
       console.log(5);
     }
     getName = function(){
       console.log(4);
     }
     getName();
     ```

     因此输出4

   - Foo().getName()  F()给一个未声明的变量getName赋值了一个函数，实际创建了一个同名的全局对象属性getName并赋值为function(){console.log(1)}，又因为函数是普通调用，没有绑定在对象上或实例上，返回的this即window，调用window.getName() 输出1

   - Foo()创建的全局getName属性覆盖了定义的函数声明，输出1

   - new Foo.getName()  注意运算优先级 先计算Foo.getName() 输出 2 

   - new Foo().getName()  注意运算优先级，先执行new Foo()，new Foo()创建一个新对象，对象关联到 Foo.prototype，返回以新创建的对象为上下文的this，因此调用Foo.prototype.getName() 输出3

   - new new Foo().getName()  执行顺序new  ( (new Foo()).getName()) 同上输出3

2. 本题考察了原型链。

   ```javascript
   var A = function() {};
   A.prototype = {constructor: function(){}};
   A.prototype = {constructor: function(){}, n: 1};
   var b = new A();
   b._proto_ = A.prototype;  
   b._proto_ = {constructor: function(){}, n: 1};
   
   A.prototype = {
     n: 2,
     m: 3
   };
   var c = new A();
   c._proto_ = A.prototype = {
     n: 2,
     m: 3
   };
   
   b.n = 1; b.m = undefined;
   c.n = 2; c.m = 3;
   ```

3. 考察原型链

   ```javascript
   F.prototype = {contructor: function(){}};
   Object.prototype.a = function() {
     console.log('a');
   };
   Function.prototype.b = function() {
     console.log('b');
   }
   f._proto_ = F.prototype;
   f.a => f._proto_.a => F.prototype.a => F.prototype._proto_.a => Object.prototype.a
   f.a()//'a'
   
   f.b => f._proto_.b => F.prototype.b => F.prototype._proto_.b =>
   Object.prototype.b
   f.b()// TypeError undefined is not a function
   
   F.a => F._proto_.a => Function.prototype.a => Function.prototype._proto_.a => Object.prototype.a
   F.a() // 'a'
   
   F.b => F._proto_.b => Function.prototype.b
   F.b() //'b'
   ```

4. (1) `p._proto_ = Person.prototype;`

   (2)`Person._proto_ = Function.prototype;`

5. 如下：

```javascript
foo.a => foo._proto_.a => Object.prototype.a => 'value a'
foo.b => foo._proto_.b => Object.prototype.b => undefined

F.a => Foo._proto_.a => Function.prototype.a => Function.prototype._proto_.a => Object.prototype.a => 'value a'
F.b => Foo._proto_.b => Function.prototype.b => 'value b' 
```


