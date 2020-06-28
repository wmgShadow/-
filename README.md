# JavaScript 内存机制
JS是单线程的语言，执行顺序肯定是顺序执行，但是JS 引擎并不是一行一行地分析和执行程序，而是一段一段地分析执行，会先进行编译阶段然后才是执行阶段。

#### JavaScript 内存机制

JS内存空间分为栈(stack)、堆(heap)、池(一般也会归类为栈中)。 其中栈存放变量，堆存放复杂对象，池存放常量。

##### 基本数据类型（简单数据类型）
JS中的基本型共有五种：string，number，Boolean，undefined，null。分别对应：字符串类型，数字类型，布尔类型，undefined（变量声明未初始化），null（空对象或理解为空指针）。

##### 引用数据类型
JS中的引用型：Array，Function，Object。但是实际上就是一种：Object型，没错，就是对象，毕竟Array，Function也是对象。JS一切皆对象这句话并不为过······

#####栈内存和堆内存
介绍完了基本型和引用型就可以真正的进入正题了。我们知道声明一个变量并且给它赋值这样的操作对于这两种类型而言没什么区别，但是对这两种类型的具体的操作却大不相同。
在JS中，栈内存用于存储基本型的变量值，堆内存用于存储引用型的值。这是为什么呢？因为JS这门语言和其他语言有一个不同之处：不允许直接访问内存的位置，也就是说不能直接操作对象的内存空间，操作的是对象的引用而已，那么引用可以理解为是一个指针，是一个具体的堆内存的地址。我写下这几行代码，并且附上一张图，让我们来详细的看一下：
``` javascript
var str = `我是字符串`,
    num = 1,
    bl = true,
    nu = null,
    un = undefined,
    obj = {
        name: 'reslicma'
    }
```
在内存中就发生了如下图这样的事情：

![](https://user-gold-cdn.xitu.io/2019/2/23/1691ae275c7589df?imageslim)


```javascript
var a = 20;
var b = a;
b = 30;

// 这时a的值是多少？
```

```javascript
var a = { name: '前端开发' }
var b = a;
b.name = '进阶';

// 这时a.name的值是多少
```
```javascript
var a = { name: '前端开发' }
var b = a;
a = null;

// 这时b的值是多少
```
#### 执行上下文
**当控制器转到可执行的代码时，会进入该代码对应的执行上下文，可以理解为该代码对应的一个执行环境，就叫做执行上下文。**

##### 在JavaScript中运行环境有三种，分别是

- 全局执行上下文：只有一个，浏览器中的全局对象就是 window 对象，this 指向这个全局对象。
- 函数执行上下文：存在无数个，只有在函数被调用的时候才会被创建，每次调用函数都会创建一个新的执行上下文。
- Eval 函数执行上下文： 指的是运行在 eval 函数中的代码，很少用而且不建议使用。


所以在一个JavaScript程序中，就会产生多个不同的执行上下文，这时候就需要用到前面提到的栈数据结构来管理了，我们称之为调用栈。当代码在执行过程中，遇到上面说的三种情况，就会产生三种执行上下文，然后分别压入调用栈中，等一个执行上下文执行完毕，弹出栈，才能执行下一个执行上下文中的代码，这就是栈结构的特点。
#### 执行上下文的特点
- 单线程，其实javascript就是单线程，所以很好理解。
- 同步执行，同步就是按顺序，不能同时执行。
- 全局上下文只有一个，它在浏览器关闭时才会弹出栈。
- 函数的执行上下文的数目没有限制。
- 每次某个函数被调用时，就会有新的执行上下文，即使是调用的自身函数。

#### demo01
```javascript

function f1(){
     var n = 999;
     function f2(){
         alert(n);
    }
    return f2;
}
var result = f1();
result();//999
```
以上面这样一个例子讲解，执行上下文在调用栈中的创建过程

![](http://chuantu.xyz/t6/702/1559302355x2073530529.png)

#### 执行上下文的生命周期

##### 执行上下文分两个阶段创建：1）创建阶段； 2）执行阶段

##### 1）创建阶段；
- 1、确定 this 的值，也被称为 This Binding。

- 2、LexicalEnvironment（词法环境） 组件被创建。

- 3、VariableEnvironment（变量环境） 组件被创建。

直接看伪代码可能更加直观
```javascript
ExecutionContext = {  
  ThisBinding = <this value>,     // 确定this 
  LexicalEnvironment = { ... },   // 词法环境
  VariableEnvironment = { ... },  // 变量环境
}
```
##### This Binding

* 全局执行上下文中，this 的值指向全局对象，在浏览器中this 的值指向 window对象，而在nodejs中指向这个文件的module对象。

* 函数执行上下文中，this 的值取决于函数的调用方式。具体有：默认绑定、隐式绑定、显式绑定（硬绑定）、new绑定、箭头函数。

##### 词法环境（Lexical Environment）

+ 词法环境有两个组成部分

  + 1、环境记录：存储变量和函数声明的实际位置

  + 2、对外部环境的引用：可以访问其外部词法环境

+ 词法环境有两种类型

  + 1、全局环境：是一个没有外部环境的词法环境，其外部环境引用为 null。拥有一个全局对象（window 对象）及其关联的方法和属性（例如数组方法）以及任何用户自定义的全局变量，this 的值指向这个全局对象。

  + 2、函数环境：用户在函数中定义的变量被存储在环境记录中，包含了arguments 对象。对外部环境的引用可以是全局环境，也可以是包含内部函数的外部函数环境。

直接看伪代码可能更加直观
```javascript
GlobalExectionContext = {  // 全局执行上下文
  LexicalEnvironment: {          // 词法环境
    EnvironmentRecord: {           // 环境记录
      Type: "Object",                 // 全局环境
      // 标识符绑定在这里
      outer: <null>                 // 对外部环境的引用
  }
}

FunctionExectionContext = { // 函数执行上下文
  LexicalEnvironment: {        // 词法环境
    EnvironmentRecord: {          // 环境记录
      Type: "Declarative",         // 函数环境
      // 标识符绑定在这里               // 对外部环境的引用
      outer: <Global or outer function environment reference>
  }
}
```
##### 变量环境

变量环境也是一个词法环境，因此它具有上面定义的词法环境的所有属性。

在 ES6 中，词法 环境和 变量 环境的区别在于前者用于存储函数声明和变量（ let 和 const ）绑定，而后者仅用于存储变量（ var ）绑定。
```javascript
let a = 20;
const b = 30;
var c;

function multiply(e, f) {
 var g = 20;
 return e * f * g;
}

c = multiply(20, 30);
```
执行上下文如下所示
```javascript
GlobalExectionContext = {

  ThisBinding: <Global Object>,

  LexicalEnvironment: {
    EnvironmentRecord: {
      Type: "Object",
      // 标识符绑定在这里
      a: < uninitialized >,
      b: < uninitialized >,
      multiply: < func >
    }
    outer: <null>
  },

  VariableEnvironment: {
    EnvironmentRecord: {
      Type: "Object",
      // 标识符绑定在这里
      c: undefined,
    }
    outer: <null>
  }
}

FunctionExectionContext = {

  ThisBinding: <Global Object>,

  LexicalEnvironment: {
    EnvironmentRecord: {
      Type: "Declarative",
      // 标识符绑定在这里
      Arguments: {0: 20, 1: 30, length: 2},
    },
    outer: <GlobalLexicalEnvironment>
  },

  VariableEnvironment: {
    EnvironmentRecord: {
      Type: "Declarative",
      // 标识符绑定在这里
      g: undefined
    },
    outer: <GlobalLexicalEnvironment>
  }
}
```
变量提升的原因：在创建阶段，函数声明存储在环境中，而变量会被设置为 undefined（在 var 的情况下）或保持未初始化（在 let 和 const 的情况下）。所以这就是为什么可以在声明之前访问 var 定义的变量（尽管是 undefined ），但如果在声明之前访问 let 和 const 定义的变量就会提示引用错误的原因。这就是所谓的变量提升。

#### 执行阶段
此阶段，完成对所有变量的分配，最后执行代码。

如果 Javascript 引擎在源代码中声明的实际位置找不到 let 变量的值，那么将为其分配 undefined 值。

有如下两段代码，执行的结果是一样的，但是两段代码究竟有什么不同？
##### dome2
```javascript
var scope = "global scope";
function checkscope(){
    var scope = "local scope";
    function f(){
        return scope;
    }
    return f();
}
checkscope();
var scope = "global scope";
function checkscope(){
    var scope = "local scope";
    function f(){
        return scope;
    }
    return f;
}
checkscope()();
```
第一段代码：

```javascript
ECStack.push(<checkscope> functionContext);
ECStack.push(<f> functionContext);
ECStack.pop();
ECStack.pop();
```

第二段代码：

```javascript
ECStack.push(<checkscope> functionContext);
ECStack.pop();
ECStack.push(<f> functionContext);
ECStack.pop();
```
##### 执行过程
执行上下文的代码会分成两个阶段进行处理
+ 1、进入执行上下文

+ 2、代码执行

##### 1、进入执行上下文
很明显，这个时候还没有执行代码

此时的变量对象会包括（如下顺序初始化）：

* 1、函数的所有形参 (only函数上下文)：没有实参，属性值设为undefined。

* 2、函数声明：如果变量对象已经存在相同名称的属性，则完全替换这个属性。

* 3、变量声明：如果变量名称跟已经声明的形参或函数相同，则变量声明不会干扰已经存在的这类属性。

上代码就直观了
```javascript
function foo(a) {
  var b = 2;
  function c() {}
  var d = function() {};

  b = 3;
}

foo(1);
```
对于上面的代码，这个时候的AO是
```javascript
AO = {
    arguments: {
        0: 1,
        length: 1
    },
    a: 1,
    b: undefined,
    c: reference to function c(){},
    d: undefined
}
```
形参arguments这时候已经有赋值了，但是变量还是undefined，只是初始化的值

##### 代码执行

这个阶段会顺序执行代码，修改变量对象的值，执行完成后AO如下
```javascript
AO = {
    arguments: {
        0: 1,
        length: 1
    },
    a: 1,
    b: 3,
    c: reference to function c(){},
    d: reference to FunctionExpression "d"
}
```
总结如下：

- 1、全局上下文的变量对象初始化是全局对象

- 2、函数上下文的变量对象初始化只包括 Arguments 对象

- 3、在进入执行上下文时会给变量对象添加形参、函数声明、变量声明等初始的属性值

- 4、在代码执行阶段，会再次修改变量对象的属性值
### 内存空间管理

#### 垃圾回收

进行前端开发时几乎不需要关心内存问题，V8限制的内存几乎不会出现用完的情况，而且我们只要关闭了浏览器，一切都结束。如果是node后端，后端程序往往进行更加复杂的操作，加上长期运行在服务器不重启，如果不关注内存管理，积少成多就会导致内存泄漏。
node中的内存第一个部分还是和上面的一样，有栈、堆、运行时环境，另外还有一个缓冲区存放Buffer。你可以通过process.memoryUsage()查看node里面进程内存使用情况。堆中的对象，被划分为新生代和老生代，他们会被不同的垃圾回收机制清理掉。

##### node内存处理机制

新生代用Scavenge算法进行垃圾回收，利用复制的方式实现内存回收的算法。
他的过程是：

- 将新生代的总空间一分为二，只使用其中一个，另一个处于闲置，等待垃圾回收时使用。使用中的那块空间称为From，闲置的空间称为To
当触发垃圾回收时，V8将From空间中所有存活下来的对象复制到To空间。
- From空间所有应该存活的对象都复制完成后，原本的From空间将被释放，成为闲置空间，原本To空间则成为使用中空间，也就是功能交换。
- 如果某对象已经经历一次新生代垃圾回收而且第二次依旧存活，或者To空间已经使用了25%，都会晋升至老生代


![](https://user-gold-cdn.xitu.io/2018/4/22/162ebbfcc425e714?imageslim)

##### 标记法垃圾回收机制
老生代利用了标记-清除（后面又加上了标记-整理）的方式进行垃圾回收。
在标记阶段（周期比较大）遍历堆中的所有对象，标记活着的对象，在随后的清除阶段中，只清除没有被标记的对象。每个内存页有一个用来标记对象的位图。这个位图另外有两位用来标记对象的状态，这个状态一共有三种：未被垃圾回收器发现、被垃圾回收器发现但邻接对象尚未全部处理、不被垃圾回收器发现但邻接对象全部被处理。分别对应着三种颜色：白、灰、黑。
遍历的时候，主要是利用DFS。刚刚开始的时候，所有的对象都是白色。从根对象开始遍历，遍历过的对象会变成灰色，放入一个额外开辟的双端队列中。标记阶段的每次循环，垃圾回收器都会从双端队列中取出一个对象染成黑对象，并将邻接的对象染色为灰，然后把其邻接对象放入双端队列。一直循环，最后所有的对象只有黑和白，白色的将会被清理。
假设全局根对象是root，那么活对象必然是被连接在对象树上面的，如果是死对象，比如var a = {};a=null我们创建了一个对象，但把他从对象树上面切断联系。这样子，DFS必然找不到他，他永远是白色。
此外，在过程中把垃圾对象删除后，内存空间是一块一块地零星散乱地分布，如果是遇到一个需要很大内存空间的对象，需要连续一大片内存存储的对象，那就有问题了。所以还有一个整理的阶段，把对象整理到在内存上连续分布。

##### 引用计数

  另一种不太常见的垃圾回收策略是引用计数。引用计数的含义是跟踪记录每个值被引用的次数。当声明了一个变量并将一个引用类型赋值给该变量时，则这个值的引用次数就是1。相反，如果包含对这个值引用的变量又取得了另外一个值，则这个值的引用次数就减1。当这个引用次数变成0时，则说明没有办法再访问这个值了，因而就可以将其所占的内存空间给收回来。这样，垃圾收集器下次再运行时，它就会释放那些引用次数为0的值所占的内存。
该方式会引起内存泄漏的原因是它不能解决循环引用的问题：
```
function sample(){
    var a={};
    var b={};
    a.prop = b;
    b.prop = a;
}
```
这种情况下每次调用sample()函数，a和b的引用计数都是2，会使这部分内存永远不会被释放，即内存泄漏。

低版本IE中有一部分对象并不是原生JS对象。例如，其BOM和DOM中的对象就是使用C++以COM(Component Object Model)对象的形式实现的，而COM对象的垃圾收集机制采用的就是引用计数策略。

因此即使IE的js引擎是用的标记清除来实现的，但是js访问COM对象如BOM,DOM还是基于引用计数的策略的，也就是说只要在IE中设计到COM对象，也就会存在循环引用的问题。


#### 闭包
闭包的概念各有各的说法，平时人家问闭包是什么，大概多数人都是说在函数中返回函数、函数外面能访问到里面的变量，这些显而易见的现象，或者把一些长篇大论搬出来。简单来说，就是外部访问内部变量，而且内部临时开辟的内存空间不会被垃圾回收。查找值的时候沿着作用域链查找，找到则停止。
对于js各种库，是一个庞大的IIFE包裹着，如果他被垃圾回收了，我们肯定不能利用了。而我们实际上就是能利用他，就是因为他暴露了接口，使得全局环境保持对IIFE内部的函数和变量的引用，我们才得以利用。
各种书对于闭包的解释：
《权威指南》：函数对象通过作用域链相互关联起来，函数内部变量都可以保持在函数的作用域中，有权访问另一个函数作用域中的变量
《忍者秘籍》：一个函数创建时允许自身访问并操作该自身函数以外的变量所创建的作用域
《你不知道的js》：是基于词法的作用域书写代码时所产生的结果，当函数记住并访问所在的词法作用域，闭包就产生了
闭包的产生，会导致内存泄漏。
前面已经说到，js具有垃圾回收机制，如果发现变量被不使用将会被回收，而闭包相互引用，让他不会被回收，一直占据着一块内存，长期持有一块内存的引用，所以导致内存泄漏。
``` javascript
//r被垃圾回收
function a(){
  var r = 1
  var s = 1
  return function count(){
    s++
    console.log(s)
  }
}
var b = a()//我们可以打个断点，在谷歌浏览器看他的调用栈，发现闭包里面没有r了

// 这时b的值是多少
```
对于最后一个例子，r、s并不是像一些人认为的那样，有闭包了，r、s都会留下，其实是r已经被回收了。在执行的函数时候，将会为这个函数创建一个上下文ctx，最开始这个ctx是空的，从上到下执行到函数a的闭包声明b时，由于b函数依赖变量s ，因此会将 s 加入b的ctx——ctx2。a内部所有的闭包，都会持有这个ctx2。（所以说，闭包之所以闭包，就是因为持有这个ctx）
每一个闭包都会引用其外部函数的ctx（这里是b的ctx2），读取变量s的时候，被闭包捕捉，加入ctx中的变量，接着被分配到堆。而真正的局部变量是r ，保存在栈，当b执行完毕后出栈并且被垃圾回收。而a的ctx被闭包引用，如果有任何一个闭包存活，他对应的ctx都将存活，变量也不会被销毁。

![](https://user-gold-cdn.xitu.io/2018/4/22/162ebbfcd3fd4351?imageslim)

我们也听说一句话，尽量避免全局变量。其实也是这样的道理，一个函数返回另一个函数，也就是分别把两个函数按顺序压入调用栈。我们知道栈是先进后出，那全局的变量（也处于栈底），越是不能得到垃圾回收，存活的时间越长。但也许全局变量在某个时候开始就没有作用了，就不能被回收，造成了内存泄漏。所以又引出另一个常见的注意事项：不要过度利用闭包。用得越多，栈越深，变量越不能被回收。

浏览器的全局对象为window，关闭浏览器自然一切结束。Node中全局对象为global，如果global中有属性已经没有用处了，一定要设置为null，因为只有等到程序停止运行，才会销毁。而我们的服务器当然是长期不关机的，内存泄漏积少成多，爆内存是早晚的事情。

Node中，当一个模块被引入，这个模块就会被缓存在内存中，提高下次被引用的速度（缓存代理）。一般情况下，整个Node程序中对同一个模块的引用，都是同一个实例（instance），这个实例一直存活在内存中。所以，如果任意模块中有变量已经不再需要，最好手动设置为null，不然会白白占用内存



作者：wmg
链接：'https://github.com/wmgShadow/-/blob/master/README.md'
来源：GitHub.com
