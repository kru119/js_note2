# JavaScript原型
### 原型使用方式1
```
Calculator.prototype = {
    add: function (x, y) {
        return x + y;
    },

    subtract: function (x, y) {
        return x - y;
    }
};
//alert((new Calculator()).add(1, 3));
```
### 原型使用方式2
第二种方式是，在赋值原型prototype的时候使用function立即执行的表达式来赋值，即如下格式：

Calculator.prototype = function () { } ();

它的好处在前面的帖子里已经知道了，就是可以封装私有的function，通过return的形式暴露出简单的使用名称，以达到public/private的效果，修改后的代码如下：
```
Calculator.prototype = function () {
    add = function (x, y) {
        return x + y;
    },

    subtract = function (x, y) {
        return x - y;
    }
    return {
        add: add,
        subtract: subtract
    }
} ();

//alert((new Calculator()).add(11, 3));
```
### 分步声明

上述使用原型的时候，有一个限制就是一次性设置了原型对象，我们再来说一下如何分来设置原型的每个属性吧。
```
var BaseCalculator = function () {
    //为每个实例都声明一个小数位数
    this.decimalDigits = 2;
};
        
//使用原型给BaseCalculator扩展2个对象方法
BaseCalculator.prototype.add = function (x, y) {
    return x + y;
};

BaseCalculator.prototype.subtract = function (x, y) {
    return x - y;
};
```
首先，声明了一个BaseCalculator对象，构造函数里会初始化一个小数位数的属性decimalDigits，然后通过原型属性设置2个function，分别是add(x,y)和subtract(x,y)，当然你也可以使用前面提到的2种方式的任何一种，我们的主要目的是看如何将BaseCalculator对象设置到真正的Calculator的原型上。

创建完上述代码以后，我们来开始：
```
var Calculator = function () {
    //为每个实例都声明一个税收数字
    this.tax = 5;
};
        
Calculator.prototype = new BaseCalculator();
```
我们可以看到Calculator的原型是指向到BaseCalculator的一个实例上，目的是让Calculator集成它的add(x,y)和subtract(x,y)这2个function，还有一点要说的是，由于它的原型是BaseCalculator的一个实例，所以不管你创建多少个Calculator对象实例，他们的原型指向的都是同一个实例。
```
var calc = new Calculator();
alert(calc.add(1, 1));
//BaseCalculator 里声明的decimalDigits属性，在 Calculator里是可以访问到的
alert(calc.decimalDigits); 
```
上面的代码，运行以后，我们可以看到因为Calculator的原型是指向BaseCalculator的实例上的，所以可以访问他的decimalDigits属性值，那如果我不想让Calculator访问BaseCalculator的构造函数里声明的属性值，那怎么办呢？这么办：
```
var Calculator = function () {
    this.tax= 5;
};

Calculator.prototype = BaseCalculator.prototype;
```
通过将BaseCalculator的原型赋给Calculator的原型，这样你在Calculator的实例上就访问不到那个decimalDigits值了，如果你访问如下代码，那将会提升出错。
var calc = new Calculator();
alert(calc.add(1, 1));
alert(calc.decimalDigits);
### 重写原型

在使用第三方JS类库的时候，往往有时候他们定义的原型方法是不能满足我们的需要，但是又离不开这个类库，所以这时候我们就需要重写他们的原型中的一个或者多个属性或function，我们可以通过继续声明的同样的add代码的形式来达到覆盖重写前面的add功能，代码如下：
```
//覆盖前面Calculator的add() function 
Calculator.prototype.add = function (x, y) {
    return x + y + this.tax;
};

var calc = new Calculator();
alert(calc.add(1, 1));
```
这样，我们计算得出的结果就比原来多出了一个tax的值，但是有一点需要注意：那就是重写的代码需要放在最后，这样才能覆盖前面的代码。
### 原型链
在将原型链之前，我们先上一段代码：
```
function Foo() {
    this.value = 42;
}
Foo.prototype = {
    method: function() {}
};

function Bar() {}

// 设置Bar的prototype属性为Foo的实例对象
Bar.prototype = new Foo();
Bar.prototype.foo = 'Hello World';

// 修正Bar.prototype.constructor为Bar本身
Bar.prototype.constructor = Bar;

var test = new Bar() // 创建Bar的一个新实例

// 原型链
test [Bar的实例]
    Bar.prototype [Foo的实例] 
        { foo: 'Hello World' }
        Foo.prototype
            {method: ...};
            Object.prototype
                {toString: ... /* etc. */};
```
上面的例子中，test 对象从 Bar.prototype 和 Foo.prototype 继承下来；因此，它能访问 Foo 的原型方法 method。同时，它也能够访问那个定义在原型上的 Foo 实例属性 value。需要注意的是 new Bar() 不会创造出一个新的 Foo 实例，而是重复使用它原型上的那个实例；因此，所有的 Bar 实例都会共享相同的 value 属性。
### 属性查找
当查找一个对象的属性时，JavaScript 会向上遍历原型链，直到找到给定名称的属性为止，到查找到达原型链的顶部 - 也就是 Object.prototype - 但是仍然没有找到指定的属性，就会返回 undefined，我们来看一个例子：
```
function foo() {
    this.add = function (x, y) {
        return x + y;
    }
}

foo.prototype.add = function (x, y) {
    return x + y + 10;
}

Object.prototype.subtract = function (x, y) {
    return x - y;
}

var f = new foo();
alert(f.add(1, 2)); //结果是3，而不是13
alert(f.subtract(1, 2)); //结果是-1
```
通过代码运行，我们发现subtract是安装我们所说的向上查找来得到结果的，但是add方式有点小不同，这也是我想强调的，就是属性在查找的时候是先查找自身的属性，如果没有再查找原型，再没有，再往上走，一直插到Object的原型上，所以在某种层面上说，用 for in语句遍历属性的时候，效率也是个问题。

还有一点我们需要注意的是，我们可以赋值任何类型的对象到原型上，但是不能赋值原子类型的值，比如如下代码是无效的：
```
function Foo() {}
Foo.prototype = 1; // 无效
```
### hasOwnProperty函数
hasOwnProperty是Object.prototype的一个方法，它可是个好东西，他能判断一个对象是否包含自定义属性而不是原型链上的属性，因为hasOwnProperty 是 JavaScript 中唯一一个处理属性但是不查找原型链的函数。
```
// 修改Object.prototype
Object.prototype.bar = 1; 
var foo = {goo: undefined};

foo.bar; // 1
'bar' in foo; // true

foo.hasOwnProperty('bar'); // false
foo.hasOwnProperty('goo'); // true
```
只有 hasOwnProperty 可以给出正确和期望的结果，这在遍历对象的属性时会很有用。 没有其它方法可以用来排除原型链上的属性，而不是定义在对象自身上的属性。

但有个恶心的地方是：JavaScript 不会保护 hasOwnProperty 被非法占用，因此如果一个对象碰巧存在这个属性，就需要使用外部的 hasOwnProperty 函数来获取正确的结果。
```
var foo = {
    hasOwnProperty: function() {
        return false;
    },
    bar: 'Here be dragons'
};

foo.hasOwnProperty('bar'); // 总是返回 false

// 使用{}对象的 hasOwnProperty，并将其上下为设置为foo
{}.hasOwnProperty.call(foo, 'bar'); // true
```
当检查对象上某个属性是否存在时，hasOwnProperty 是唯一可用的方法。同时在使用 for in loop 遍历对象时，推荐总是使用 hasOwnProperty 方法，这将会避免原型对象扩展带来的干扰，我们来看一下例子：
```
// 修改 Object.prototype
Object.prototype.bar = 1;

var foo = {moo: 2};
for(var i in foo) {
    console.log(i); // 输出两个属性：bar 和 moo
}
```
我们没办法改变for in语句的行为，所以想过滤结果就只能使用hasOwnProperty 方法，代码如下：
```
// foo 变量是上例中的
for(var i in foo) {
    if (foo.hasOwnProperty(i)) {
        console.log(i);
    }
}
```
### 原型链图
```
 
var a = { 
x: 10, 
calculate: function (z) { 
return this.x + this.y + z 
} 
}; 
var b = { 
y: 20, 
__proto__: a 
}; 

var c = { 
y: 30, 
__proto__: a 
}; 

// call the inherited method 
b.calculate(30); // 60 
```
![](https://img.jbzj.com/file_images/article/201207/201207071647183.png)

当定义一个prototype的时候，会构造一个原形对象，这个原型对象存储于构造这个prototype的函数的原形方法之中.
```
 
function Foo(y){ 
this.y = y ; 
} 

Foo.prototype.x = 10; 

Foo.prototype.calculate = function(z){ 
return this.x+this.y+z; 
}; 

var b = new Foo(20); 

alert(b.calculate(30)); 
```
![](https://img.jbzj.com/file_images/article/201207/201207071647184.png)
                
## 闭包
要理解闭包，首先必须理解Javascript特殊的变量作用域。

变量的作用域无非就是两种：全局变量和局部变量。

Javascript语言的特殊之处，就在于函数内部可以直接读取全局变量。
```
　　var n=999;

　　function f1(){
　　　　alert(n);
　　}

　　f1(); // 999
```
另一方面，在函数外部自然无法读取函数内的局部变量。
```
　　function f1(){
　　　　var n=999;
　　}

　　alert(n); // error
```
这里有一个地方需要注意，函数内部声明变量的时候，一定要使用var命令。如果不用的话，你实际上声明了一个全局变量！
```
　　function f1(){
　　　　n=999;
　　}

　　f1();

　　alert(n); // 999
```
### 二、如何从外部读取局部变量？

出于种种原因，我们有时候需要得到函数内的局部变量。但是，前面已经说过了，正常情况下，这是办不到的，只有通过变通方法才能实现。

那就是在函数的内部，再定义一个函数。
```
　　function f1(){

　　　　var n=999;

　　　　function f2(){
　　　　　　alert(n); // 999
　　　　}

　　}
```
在上面的代码中，函数f2就被包括在函数f1内部，这时f1内部的所有局部变量，对f2都是可见的。但是反过来就不行，f2内部的局部变量，对f1就是不可见的。这就是Javascript语言特有的"链式作用域"结构（chain scope），子对象会一级一级地向上寻找所有父对象的变量。所以，父对象的所有变量，对子对象都是可见的，反之则不成立。

既然f2可以读取f1中的局部变量，那么只要把f2作为返回值，我们不就可以在f1外部读取它的内部变量了吗！
```
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
### 三、闭包的概念

上一节代码中的f2函数，就是闭包。

各种专业文献上的"闭包"（closure）定义非常抽象，很难看懂。我的理解是，闭包就是能够读取其他函数内部变量的函数。

由于在Javascript语言中，只有函数内部的子函数才能读取局部变量，因此可以把闭包简单理解成"定义在一个函数内部的函数"。

所以，在本质上，闭包就是将函数内部和函数外部连接起来的一座桥梁。
* 闭包就是函数的局部变量集合，只是这些局部变量在函数返回后会继续存在。
* 闭包就是就是函数的“堆栈”在函数返回后并不释放，我们也可以理解为这些函数堆栈并不在栈上分配而是在堆上分配
* 当在一个函数内定义另外一个函数就会产生闭包

### 四、闭包的用途

闭包可以用在许多地方。它的最大用处有两个，一个是前面提到的可以读取函数内部的变量，另一个就是让这些变量的值始终保持在内存中。

怎么来理解这句话呢？请看下面的代码。
```
　　function f1(){

　　　　var n=999;

　　　　nAdd=function(){n+=1}

　　　　function f2(){
　　　　　　alert(n);
　　　　}

　　　　return f2;

　　}

　　var result=f1();

　　result(); // 999

　　nAdd();

　　result(); // 1000
```
在这段代码中，result实际上就是闭包f2函数。它一共运行了两次，第一次的值是999，第二次的值是1000。这证明了，函数f1中的局部变量n一直保存在内存中，并没有在f1调用后被自动清除。

为什么会这样呢？原因就在于f1是f2的父函数，而f2被赋给了一个全局变量，这导致f2始终在内存中，而f2的存在依赖于f1，因此f1也始终在内存中，不会在调用结束后，被垃圾回收机制（garbage collection）回收。

这段代码中另一个值得注意的地方，就是"nAdd=function(){n+=1}"这一行，首先在nAdd前面没有使用var关键字，因此nAdd是一个全局变量，而不是局部变量。其次，nAdd的值是一个匿名函数（anonymous function），而这个匿名函数本身也是一个闭包，所以nAdd相当于是一个setter，可以在函数外部对函数内部的局部变量进行操作。

### 五、使用闭包的注意点

1）由于闭包会使得函数中的变量都被保存在内存中，内存消耗很大，所以不能滥用闭包，否则会造成网页的性能问题，在IE中可能导致内存泄露。解决方法是，在退出函数之前，将不使用的局部变量全部删除。

2）闭包会在父函数外部，改变父函数内部变量的值。所以，如果你把父函数当作对象（object）使用，把闭包当作它的公用方法（Public Method），把内部变量当作它的私有属性（private value），这时一定要小心，不要随便改变父函数内部变量的值。
### ECMAScript闭包模型
在ECMAscript的脚本的函数运行时，每个函数关联都有一个执行上下文场景(Execution Context) ，这个执行上下文场景中包含三个部分

文法环境（The LexicalEnvironment）
变量环境（The VariableEnvironment）
this绑定
其中第三点this绑定与闭包无关，不在本文中讨论。文法环境中用于解析函数执行过程使用到的变量标识符。我们可以将文法环境想象成一个对象，该对 象包含了两个重要组件，环境记录(Enviroment Recode)，和外部引用(指针)。环境记录包含包含了函数内部声明的局部变量和参数变量，外部引用指向了外部函数对象的上下文执行场景。全局的上下文 场景中此引用值为NULL。这样的数据结构就构成了一个单向的链表，每个引用都指向外层的上下文场景。
```
function greeting(name) {
     var text = 'Hello ' + name; // local variable
     // 每次调用时，产生闭包，并返回内部函数对象给调用者
     return function () { alert(text); }
}
var sayHello=greeting( "Closure" );
sayHello()  // 通过闭包访问到了局部变量text
```
例如上面我们例子的闭包模型应该是这样，sayHello函数在最下层，上层是函数greeting，最外层是全局场景。如下图：
![](http://static.oschina.net/uploads/img/201203/07193845_BKpn.png)
### 闭包的样列
例子1:闭包中局部变量是引用而非拷贝
```
例子2：多个函数绑定同一个闭包，因为他们定义在同一个函数内。

```
function setupSomeGlobals() {
    // Local variable that ends up within closure
    var num = 666;
    // Store some references to functions as global variables
    gAlertNumber = function() { alert(num); }
    gIncreaseNumber = function() { num++; }
    gSetNumber = function(x) { num = x; }
}
setupSomeGolbals(); // 为三个全局变量赋值
gAlertNumber(); //666
gIncreaseNumber();
gAlertNumber(); // 667
gSetNumber(12);//
gAlertNumber();//12
```
例子3：当在一个循环中赋值函数时，这些函数将绑定同样的闭包

```
