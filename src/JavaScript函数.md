## 1、函数描述

### 1.1 函数是头等(first-class)对象
- 因为它们可以像任何其他对象一样具有属性和方法。
- 它们与其他对象的区别在于函数可以被调用。
- 每个函数其实都是一个Function对象

### 1.2 返回值
- 默认返回undefined
- 使用 return 语句来指定一个要返回的值（使用new关键字调用一个构造函数除外）

### 1.3 函数参数
- 调用函数时，传递给函数的值被称为函数的实参（值传递），
- 对应位置的函数参数名叫作形参。
- 如果实参是一个包含原始值(数字，字符串，布尔值)的变量，则就算函数在内部改变了对应形参的值，返回后，该实参变量的值也不会改变。
- 如果实参是一个对象引用，则对应形参会和该实参指向同一个对象。假如函数在内部改变了对应形参的值，返回后，实参指向的对象的值也会改变。

### 1.4 this 指向

- 在函数执行时，this 关键字并不会指向正在运行的函数本身，而是指向调用该函数的对象
- 如果你想在函数内部获取函数自身的引用，只能使用函数名或者使用arguments.callee属性(严格模式下不可用)，如果该函数是一个匿名函数,则你只能使用后者。


## 2、函数定义

### 2.1 函数声明

### 2.1.1 用法
```
function name([参数, 参数, ... 参数) { statements }
```

```javascript
function myFunc(theObject) {
  theObject.make = "Toyota";
}

var mycar = {make: "Honda", model: "Accord", year: 1998};
var x, y;

x = mycar.make;     // x获取的值为 "Honda"

myFunc(mycar);
y = mycar.make;     // y获取的值为 "Toyota"
                    // (make属性被函数改变了)

```
### 2.1.2 函数声明提升
- JavaScript 中的函数声明被提升到了函数定义，可以在函数声明之前使用该函数。
- 函数声明同时也创建了一个和函数名相同的变量。因此，以函数声明定义的函数能够在它们被定义的作用域内通过函数名而被访问到

```javascript
hoisted(); // "foo"

function hoisted() {
     console.log("foo");
}

/* equal to*/
var hoisted; 

hoisted = function() {
  console.log("foo");
}
hoisted();
// "foo" 

```


### 2.2 函数表达式

#### 2.2.1 用法

```
let 变量名称 = function 函数名称(参数, 参数, ...参数) {
   statements
};
```
- 函数名称可被省略，此种情况下的函数是匿名函数（anonymous）
- 函数名称只是函数体中的一个本地变量。


```javascript
const square = function(number) { return number * number; };
var x = square(4); // x gets the value 16

const factorial = function fac(n) {return n<2 ? 1 : n*fac(n-1)};
console.log(factorial(3));

```

#### 2.2.2 没有提升

```javascript
 notHoisted(); // TypeError: notHoisted is not a function

var notHoisted = function() {
   console.log('bar');
};
```

### 2.3 函数生成器声明 (function* 声明)
定义一个生成器函数 (generator function)，它返回一个  Generator  对象。

```
function* 函数名(参数,参数, ... 参数) { statements }
```


```javascript
function* generator(i) {
  yield i;
  yield i + 10;
}

const gen = generator(10);

console.log(gen.next().value);
// expected output: 10

console.log(gen.next().value);
// expected output: 20
```

- 生成器函数在执行时能暂停，后面又能从暂停处继续执行。
- 调用一个生成器函数并不会马上执行它里面的语句，而是返回一个这个生成器的 迭代器 （ iterator ）对象。
- 当这个迭代器的 next() 方法被首次（后续）调用时，其内的语句会执行到第一个（后续）出现yield的位置为止，yield 后紧跟迭代器要返回的值。
- 或者如果用的是 yield*（多了个星号），则表示将执行权移交给另一个生成器函数（当前生成器暂停执行）。
- next()方法返回一个对象，这个对象包含两个属性：value 和 done，
- value 属性表示本次 yield 表达式的返回值，
- done 属性为布尔类型，表示生成器后续是否还有 yield 语句，即生成器函数是否已经执行完毕并返回。



### 2.4 函数生成器表达式 (function*表达式)
- function* 表达式和 function* 声明比较相似，并具有几乎相同的语法。
- function* 表达式和function* 声明之间主要区别就是函数名，
- 即在创建匿名函数时，function*表达式可以省略函数名

```javascript
var x = function*(y) {
   yield y * y;
};
```

### 2.5 箭头函数表达式 (=>)

```
(param1, param2, …, paramN) => { statements }
```

- 更简短的函数
- 不绑定this

#### 2.5.1 更短的函数

```
var elements = [
  'Hydrogen',
  'Helium',
  'Lithium',
  'Beryllium'
];

elements.map(function(element) {
  return element.length;
}); // 返回数组：[8, 6, 7, 9]

// 上面的普通函数可以改写成如下的箭头函数
elements.map((element) => {
  return element.length;
}); // [8, 6, 7, 9]

// 当箭头函数只有一个参数时，可以省略参数的圆括号
elements.map(element => {
 return element.length;
}); // [8, 6, 7, 9]

// 当箭头函数的函数体只有一个 `return` 语句时，可以省略 `return` 关键字和方法体的花括号
elements.map(element => element.length); // [8, 6, 7, 9]

// 在这个例子中，因为我们只需要 `length` 属性，所以可以使用参数解构
// 需要注意的是字符串 `"length"` 是我们想要获得的属性的名称，而 `lengthFooBArX` 则只是个变量名，
// 可以替换成任意合法的变量名
elements.map(({ "length": lengthFooBArX }) => lengthFooBArX); // [8, 6, 7, 9]

```
#### 2.5.2 不绑定this

```javascript
function Person() {
  // Person() 构造函数定义 `this`作为它自己的实例.
  this.age = 0;

  setInterval(function growUp() {
    // 在非严格模式, growUp()函数定义 `this`作为全局对象,
    // 与在 Person()构造函数中定义的 `this`并不相同.
    this.age++;
  }, 1000);
}

var p = new Person();


```


```javascript
<!--通过将this值分配给封闭的变量，可以解决this问题-->
function Person() {
  var that = this;
  that.age = 0;

  setInterval(function growUp() {
    // 回调引用的是`that`变量, 其值是预期的对象.
    that.age++;
  }, 1000);
}
```


```javascript
<!--箭头函数不会创建自己的this,它只会从自己的作用域链的上一层继承this-->
function Person(){
  this.age = 0;

  setInterval(() => {
    this.age++; // |this| 正确地指向 p 实例
  }, 1000);
}

var p = new Person();
```

### 2.6 Function构造函数表达式（~~不推荐~~）

```javascript
//结果为true证明没函数都是一个Function对象
(function(){}).constructor === Function //true
```

#### 2.6.1 描述

- 使用 Function 构造函数生成的 Function 对象是在函数创建时解析的。
- 这比你使用函数声明或者函数表达式并在你的代码中调用更为低效，
- 因为使用后者创建的函数是跟其他代码一起解析的。
- 以调用函数的方式调用 Function 的构造函数（而不是使用 new 关键字) 跟以构造函数来调用是一样的。

```javascript
<!--可以正常执行-->
const sum = new Function("a", "b", "return a + b");
console.log(sum(2, 6));

<!--可以正常执行-->
const sum2 = Function.prototype.constructor( "a", "b", "return a + b" );
console.log(sum2(2, 6));
```


```
<!--用法-->
new Function (参数, 参数, ...参数, functionBody字符串)

//例如
const sum = new Function('a', 'b', 'return a + b');
console.log(sum(2, 6));
// expected output: 8

```

#### 2.6.2 实例属性

- Function.caller 获取调用函数的具体对象。（存在兼容性问题）
- Function.length 获取函数的接收参数个数。
- Function.name 获取函数的名称。

#### 2.6.3 实例方法

- Function.prototype.apply() 在一个对象的上下文中应用另一个对象的方法；参数能够以数组形式传入。
- Function.prototype.bind()
    - bind()方法会创建一个新函数,称为绑定函数.
    - 当调用这个绑定函数时,绑定函数会以创建它时传入 bind()方法的第一个参数作为 this,
    - 传入 bind()方法的第二个以及以后的参数加上绑定函数运行时本身的参数按照顺序作为原函数的参数来调用原函数.
- Function.prototype.call() 在一个对象的上下文中应用另一个对象的方法；参数能够以列表形式传入。
- Function.prototype.toString() 获取函数的实现源码的字符串。覆盖了 Object.prototype.toString 方法。

#### 2.6.4 与函数声明、函数表达式区别
- 和函数表达式一样没有函数声明提升。不能在声明前调用
- 不能创建当前环境闭包，只能被创建于全局环境
- 只能访问全局变量和自己的局部变量


```javascript
    const x = 10;
    function createFunction1() {
        const x = 20;
        return new Function("return x;"); // 这里的 x 指向最上面全局作用域内的 x
    }
    function createFunction2() {
        const x = 20;
        function f() {
            return x; // 这里的 x 指向上方本地作用域内的 x
        }
        return f;
    }
    const f1 = createFunction1();
    console.log(f1()); // 10
    const f2 = createFunction2();
    console.log(f2()); // 20
```
#### 2.6.5 尽量避免使用的原因

- 每次构造函数被调用，传递给Function构造函数的函数体字符串都要被解析一次 。
- 虽然函数表达式每次都创建了一个闭包，但函数体不会被重复解析，因此函数表达式仍然要快于"new Function(...)"
- 在通过解析Function构造函数字符串产生的函数里，内嵌的函数表达式和函数声明不会被重复解析。




### 2.7 生成器函数的构造函数（~~不推荐~~）
- GeneratorFunction构造器生成新的生成器函数 对象。
- 在JavaScript中，生成器函数实际上都是GeneratorFunction的实例对象。
- GeneratorFunction并不是一个全局对象。它可以通过下面的代码获取。

```javascript
Object.getPrototypeOf(function*(){}).constructor
```

#### 2.7.1 语法

```
new GeneratorFunction ([arg1[, arg2[, ...argN]],] functionBody字符串)
```

```javascript
var GeneratorFunction = Object.getPrototypeOf(function*(){}).constructor
var g = new GeneratorFunction("a", "yield a * 2");
var iterator = g(10);
console.log(iterator.next().value); // 20

```

#### 2.7.2 尽量避免使用的原因
- 当创建函数时，将使用GeneratorFunction构造函数创建的生成器函数对象进行解析。
- 这比使用function* 表达式 声明生成器函数效率更低，
- 使用GeneratorFunction构造函数创建的生成器函数不会为其创建上下文创建闭包；它们始终在全局范围内创建。
- 当运行它们时，它们只能访问自己的本地变量和全局变量，而不是从GeneratorFunction构造函数调用的范围的变量。

## 3、函数作用域

- 在函数内定义的变量不能在函数之外的任何地方访问，因为变量仅仅在该函数的域的内部有定义。
- 一个函数可以访问其定义所在作用域内的任何变量和函数。例如，定义在全局域中的函数可以访问所有定义在全局域中的变量。
- 在另一个函数中定义的函数也可以访问在其父函数中定义的所有变量和父函数有权访问的任何其他变量。


```javascript
// 下面的变量定义在全局作用域(global scope)中
var num1 = 20,
    num2 = 3,
    name = "Chamahk";

// 本函数定义在全局作用域
function multiply() {
  return num1 * num2;
}

multiply(); // 返回 60

// 嵌套函数的例子
function getScore() {
  var num1 = 2,
      num2 = 3;

  function add() {
    return name + " scored " + (num1 + num2);
  }

  return add();
}

getScore(); // 返回 "Chamahk scored 5"

```

## 4、闭包
- JavaScript 允许函数嵌套，
- 内部函数可以访问定义在外部函数中的所有变量和函数，以及外部函数能访问的所有变量和函数。
- 外部函数却不能够访问定义在内部函数中的变量和函数。这给内部函数的变量提供了一定的安全性。
- 由于内部函数可以访问外部函数的作用域，因此当内部函数生存周期大于外部函数时，外部函数中定义的变量和函数的生存周期将比内部函数执行时间长。
- 当内部函数以某一种方式被任何一个外部函数作用域访问时，一个闭包就产生了。


```javascript
var pet = function(name) {          //外部函数定义了一个变量"name"
  var getName = function() {
    //内部函数可以访问 外部函数定义的"name"
    return name;
  }
  //返回这个内部函数，从而将其暴露在外部函数作用域
  return getName;
};
myPet = pet("Vivie");

myPet();    
```

### 4.1 命名冲突
- 当同一个闭包作用域下两个参数或者变量同名时，就会产生命名冲突。
- 更近的作用域有更高的优先权，所以最近的优先级最高，最远的优先级最低。这就是作用域链。
- 链的第一个元素就是最里面的作用域，最后一个元素便是最外层的作用域。


```javascript
function outside() {
  var x = 5;
  function inside(x) {
    return x * 2;
  }
  return inside;
}

outside()(10); // returns 20 instead of 10
```

## 5.arguments对象

- arguments 是一个对应于传递给函数的参数的类数组对象
- arguments变量只是 ”类数组对象“，并不是一个数组。
- 称其为类数组对象是说它有一个索引编号和length属性。
- 尽管如此，它并不拥有全部的Array对象的操作方法。
- arguments对象是所有（非箭头）函数中都可用的局部变量。

```javascript
function func1(a, b, c) {
  console.log(arguments[0]);
  // expected output: 1

  console.log(arguments[1]);
  // expected output: 2

  console.log(arguments[2]);
  // expected output: 3
}

func1(1, 2, 3);
```

### 5.1 属性
- arguments.callee  : 当前正在执行的函数。
- arguments.caller  : 调用当前执行函数的函数。
- arguments.length  : 传给函数的参数的数目。

## 6. 函数参数

从ECMAScript 6开始，有两个新的类型的参数：默认参数，剩余参数

### 6.1 默认参数

在JavaScript中，函数参数的默认值是 undefined。在过去，用于设定默认参数的一般策略是在函数的主体中测试参数值是否为undefined，如果是则赋予这个参数一个默认值。

```javascript
function multiply(a, b) {
  b = (typeof b !== 'undefined') ?  b : 1;

  return a*b;
}

multiply(5); // 5

```

```javascript
<!--使用默认参数，在函数体的检查就不再需要-->
function multiply(a, b = 1) {
  return a*b;
}

multiply(5); // 5
```

### 6.2 剩余参数

剩余参数语法允许将不确定数量的参数表示为数组。
```javascript
function multiply(multiplier, ...theArgs) {
  return theArgs.map(x => multiplier * x);
}

var arr = multiply(2, 1, 2, 3);
console.log(arr); // [2, 4, 6]
```

> 参考链接
- [Function](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function)
- [函数表达式](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/function)
- [function*](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/function*)
- [箭头函数](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Functions/Arrow_functions)
- [剩余参数](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Functions/rest_parameters)
- [GeneratorFunction](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/GeneratorFunction)
