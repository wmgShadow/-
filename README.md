# 执行上下文
JS是单线程的语言，执行顺序肯定是顺序执行，但是JS 引擎并不是一行一行地分析和执行程序，而是一段一段地分析执行，会先进行编译阶段然后才是执行阶段。
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

![markdown](https://upload-images.jianshu.io/upload_images/4067575-cdb1fa08a10a66ac.png?imageMogr2/auto-orient/ "markdown")

#### 执行上下文的生命周期

##### 执行上下文分两个阶段创建：1）创建阶段； 2）执行阶段

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


作者：wmg
链接：'https://github.com/wmgShadow/-/blob/master/README.md'
来源：GitHub.com
