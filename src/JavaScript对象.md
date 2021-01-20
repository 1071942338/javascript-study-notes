## 1、概述
- 一个对象就是一系列属性的集合
- 一个属性包含一个名和一个值
- 一个属性的值可以是函数，该属性也被称为方法

## 2、对象和属性
- 一个 javascript 对象有很多属性。
- 一个对象的属性可以被解释成一个附加到对象上的变量。
- 对象的属性和普通的 javascript 变量基本没什么区别，仅仅是属性属于某个对象。
- 属性定义了对象的特征
- 可以通过点符号或者方括号访问或者设置一个对象的属性
- 对象的名字(可以是普通的变量)和属性的名字都是大小写敏感
- 对象中未赋值的属性的值为undefined，而不是null


```javascript
var myCar = new Object();
myCar.make = "Ford";
myCar.model = "Mustang";
myCar.year = 1969;

myCar["make"] = "Ford";
myCar["model"] = "Mustang";
myCar["year"] = 1969;
```

## 3、枚举一个对象的所有属性

```javascript
    class Parent {
    constructor(name) {
        this.name = name;
        this.height = name;
    }
    say() {}
}
class Child extends Parent {
    constructor(name, age) {
        super(name);
        this.age = age;
    }
    say() {}
}
Child.prototype.sex = "1";
const child = new Child("child", 10);
console.log(child);
```

### 3.1 ES5 之前 getOwnPropertyNames

```javascript
   <!-- ES6之前 -->
function listAllProperties(o, isEnumerable = false) {
    //获取所有属性，包含原型链
    let objectToInspect;
    let result = [];
    for (
        objectToInspect = o;
        objectToInspect !== null;
        objectToInspect = Object.getPrototypeOf(objectToInspect)
    ) {
        result = result.concat(Object.getOwnPropertyNames(objectToInspect));
    }
    //数组元素去重
    const newRes = [];
    result.forEach((item) => {
        if (newRes.indexOf(item) === -1) {
            if (isEnumerable === true) {
                //保留不可枚举属性
                newRes.push(item);
            } else {
                //不保留不可枚举属性
                o.propertyIsEnumerable(item) && newRes.push(item);
            }
        }
    });
    return newRes;
}
console.log("ES5之前:", listAllProperties(child, true));
    
    <!--//结果-->
<!--["name", "height", "age", "constructor", "say", "sex", "__defineGetter__", "__defineSetter__", "hasOwnProperty", "__lookupGetter__", "__lookupSetter__", "isPrototypeOf", "propertyIsEnumerable", "toString", "valueOf", "__proto__", "toLocaleString"]-->
```

### 3.2 for...in
该方法依次访问一个对象及其原型链中所有可枚举的属性。

```javascript
    const keys = [];
    for (let key in child) {
        keys.push(key);
    }
    console.log("for...in:", keys);
    <!--结果：["name", "height", "age", "sex"]-->
```

### 3.3 Object.keys(o)
该方法返回对象 o 自身包含（不包括原型中）的所有可枚举属性的名称的数组。
```javascript
    console.log("Object.keys:", Object.keys(child));
    <!--结果：["name", "height", "age"]-->
```
### 3.4 Object.getOwnPropertyNames(o)
该方法返回对象 o 自身包含（不包括原型中）的所有属性(无论是否可枚举)的名称的数组。
```javascript
     console.log(
        "Object.getOwnPropertyNames:",
        Object.getOwnPropertyNames(child)
    );
    <!--结果：["name", "height", "age"]-->
```

## 4、创建新对象

### 4.1 Object 构造函数

```javascript
    <!--Object构造函数-->
    const obj1 = new Object();
    obj1.name = "zs";
    obj1.say = function () {
        console.log("obj1:", this.name);
    };
    obj1.say();
    console.log("obj1:", obj1);
```

### 4.2 Object.create()
创建一个新对象，使用现有的对象来提供新创建的对象的`__proto__`
```javascript
    <!--Object.create函数-->
    // const obj2 = Object.create(obj1);
    // const obj2 = Object.create(Object.prototype);
    const obj2 = Object.create({});
    obj2.name = "zs";
    obj2.say = function () {
        console.log("obj2:", this.name);
    };
    obj2.say();
    console.log("obj2:", obj2);
```
### 4.3 构造函数
两步来创建对象：
- 通过创建一个构造函数来定义对象的类型。首字母大写是非常普遍而且很恰当的惯用法。
- 通过 new 创建对象实例。


```javascript
    <!--构造函数-->
    function Person(name) {
        this.name = name;
        this.say = function () {
            console.log("obj3:", this.name);
        };
    }

    const obj3 = new Person("zs");
    obj3.name = "zs";
    obj3.say = function () {
        console.log("obj3:", this.name);
    };
    obj3.say();
    console.log("obj3:", obj3);
```
### 4.4 字面量

```javascript
    <!--字面量-->
    const obj4 = {};
    obj4.name = "zs";
    obj4.say = function () {
        console.log("obj4:", this.name);
    };
    obj4.say();
    console.log("obj4:", obj4);
```
### 4.5 class 创建

```javascript
    <!--class-->
    class People {
        constructor(name) {
            this.name = name;
        }
        say() {
            console.log("obj5:", this.name);
        }
    }
    const obj5 = new People("zs");
    obj5.name = "zs";
    obj5.say = function () {
        console.log("obj5:", this.name);
    };
    obj5.say();
    console.log("obj5:", obj5);
```

## 5、原型
- JavaScript 常被描述为一种基于原型的语言
- 每个对象拥有一个原型对象，是隐藏属性
- 对象以其原型为模板、从原型继承方法和属性。
- 原型对象也可能拥有原型，并从中继承方法和属性，一层一层、以此类推。
- 这种关系常被称为原型链 (prototype chain)，它解释了为何一个对象会拥有定义在其他对象中的属性和方法。

### 5.1 函数对象获取原型
通过 prototype 属性访问原型。该属性定义在 Function 对象上。没有找到相关文档，但是可以通过以下验证

```javascript
Function.hasOwnProperty("prototype")
//结果为 true
```

- 每个 JavaScript 函数实际上都是一个 Function 对象实例
- 构造函数也是函数


```javascript
function doSomething(){}
console.log( doSomething.prototype );
//var doSomething = function(){};
//console.log( doSomething.prototype );

```
输出结果为

```
{
    constructor: ƒ doSomething(),
    __proto__: {
        constructor: ƒ Object(),
        hasOwnProperty: ƒ hasOwnProperty(),
        isPrototypeOf: ƒ isPrototypeOf(),
        propertyIsEnumerable: ƒ propertyIsEnumerable(),
        toLocaleString: ƒ toLocaleString(),
        toString: ƒ toString(),
        valueOf: ƒ valueOf()
    }
}
```

### 5.2 实例对象获取原型

- `__proto__`属性，该方法从 Object.prototype 继承而来
- Object.getPrototypeOf(obj)方法
- obj.constructor.prototype,其中 constructor 方法从 Object.prototype 继承而来


```javascript
function People(){};
const p = new People();
console.log(p.__proto__)
console.log(Object.getPrototypeOf(p))
console.log(p.constructor.prototype)

<!-- 结果-->
<!--constructor: ƒ People()-->
<!--__proto__:-->
<!--constructor: ƒ Object()-->
<!--hasOwnProperty: ƒ hasOwnProperty()-->
<!--isPrototypeOf: ƒ isPrototypeOf()-->
<!--propertyIsEnumerable: ƒ propertyIsEnumerable()-->
<!--toLocaleString: ƒ toLocaleString()-->
<!--toString: ƒ toString()-->
<!--valueOf: ƒ valueOf()-->
<!--__defineGetter__: ƒ __defineGetter__()-->
<!--__defineSetter__: ƒ __defineSetter__()-->
<!--__lookupGetter__: ƒ __lookupGetter__()-->
<!--__lookupSetter__: ƒ __lookupSetter__()-->
<!--get __proto__: ƒ __proto__()-->
<!--set __proto__: ƒ __proto__()-->


console.log(p.__proto__ === p.constructor.prototype)
console.log(p.__proto__ === Object.getPrototypeOf(p))
console.log(p.constructor.prototype === Object.getPrototypeOf(p))
<!-- 结果-->
<!--true->

```

## 6、继承
- JavaScript没有类，全是 ++对象++
- 对象的原型叫 ++对象原型++
- 通过对象创建的叫 ++对象实例++
- 对象实例的原型是 ++对象原型++ 不是 ++对象++
- 对象实例的初始属性来自 ++复制、初始化对象属性++ 和 ++继承对象原型/链属性++

### 6.1 继承方式1:call

```javascript
 //定义父类
    function People1(name) {
        //定义父类属性，每个实例对象值都一样，定义在对象上
        this.name = name;
    }
    //定义父类方法，每个实例对象执行方法一样，定义在原型上
    People1.prototype.say = function () {};

    //定义子类
    //1、实现属性拷贝和赋值
    function Student1(name, age) {
        //使用this调用父类构造函数，生成对象
        People1.call(this, name);
        //子类属性初始化赋值
        this.age = age;
    }
    //2、实现原型链指向
    /*
     * //不能直接指向父类原型，因为还要修改原型constructor指向，会破坏父类的原型链
     * Student1.prototype = People1.prototype;
     * */
    /*
     * //父类构造函数需要初始化参数，作为原型来讲还需要提供不必要的参数
     * Student1.prototype = new People1("");
     * */
    // 重新生成一个新的原型对象，防止子类父类原型链互相污染
    Student1.prototype = Object.create(People1.prototype);
    Student1.prototype.constructor = Student1;

    const people = new People1("ww");
    console.log("people:", people);
    console.log(
        "people.constructor.prototype:",
        people.constructor.prototype
    );
    People1.prototype.run2 = function () {};
    Student1.prototype.run3 = function () {};
    const student = new Student1("zs", 20);
    console.log("student:", student);
    console.log(
        "student.constructor.prototype:",
        student.constructor.prototype
    );
```


### 6.2 继承方式1:extends

```javascript
  class People2 {
        constructor(name) {
            this.name = name;
        }
        say = function () {};
    }
    class Student2 extends People2 {
        constructor(name, age) {
            super(name);
            this.age = age;
        }
    }
    const people2 = new People2("ww");
    console.log("people2:", people2);
    console.log(
        "people2.constructor.prototype:",
        people2.constructor.prototype
    );
    People2.prototype.run2 = function () {};
    Student2.prototype.run3 = function () {};
    const student2 = new Student2("zs", 20);
    console.log("student2:", student2);
    console.log(
        "student2.constructor.prototype:",
        student2.constructor.prototype
    );
```


## 7、 Function 和 Object

-  Object.protype 是顶级原型对象

```
 //Object对象
    console.log("Object:", Object);
    console.log("-Object函数属性---");
    //Object函数原型
    console.log("Object.prototype:", Object.prototype);
    //Object函数原型的原型： 值为null
    console.log(
        "Object.prototype.__proto__:",
        Object.prototype.__proto__
    );
    console.log("-Object对象属性---");
    //Object对象-原型
    console.log("Object.__proto__:", Object.__proto__);
    //Object对象-原型-原型
    console.log(
        "Object.__proto__.__proto__:",
        Object.__proto__.__proto__
    );
    //Object对象-原型-原型 等于 Object.prototype
    console.log(
        "Object.__proto__.__proto__:",
        Object.__proto__.__proto__ === Object.prototype
    );
    //Object对象-原型-原型-原型 ： 值为null
    console.log(
        "Object.__proto__.__proto__.__proto__:",
        Object.__proto__.__proto__.__proto__
    );

    console.log("-Object构造函数属性---");
    //Object对象-构造函数:Function
    console.log("Object.constructor:", Object.constructor);

    console.log("-Function构造函数属性---");
    //Function函数-原型
    console.log("Function.prototype:", Function.prototype);
    //Function函数-原型-原型
    console.log(
        "Function.prototype.__proto__:",
        Function.prototype.__proto__
    );
    //Function函数-原型-原型 等于 Object.prototype 等于 Object.__proto__.__proto__
    console.log(
        "Function.prototype.__proto__:",
        Function.prototype.__proto__ === Object.prototype
    );
    console.log("-Function对象属性---");
    //Function对象-原型
    console.log("Function.__proto__:", Function.__proto__);
    //Function对象-原型 等于 Function.prototype
    console.log(
        "Function.__proto__:",
        Function.__proto__ === Function.prototype
    );

    //Function对象-原型-原型
    console.log(
        "Function.__proto__.__proto__:",
        Function.__proto__.__proto__
    );
    //Function对象-原型-原型 等于Function.prototype.__proto__ 等于 Object.prototype 等于 Object.__proto__.__proto__
    console.log(
        "Function.__proto__.__proto__:",
        Function.__proto__.__proto__ === Function.prototype.__proto__
    );

    console.log("-Function构造函数属性---");
    console.log("Function.constructor:", Function.constructor);
    console.log(
        "Function.constructor:",
        Function.constructor.prototype
    );
    console.log(
        "Function.constructor:",
        Function.constructor.prototype === Function.__proto__
    );
```
- [代码链接-创建对象](https://github.com/hao-kuai/javascript-study-notes/blob/main/src/%E5%88%9B%E5%BB%BA%E6%96%B0%E5%AF%B9%E8%B1%A1.html)
- [代码链接-继承](https://github.com/hao-kuai/javascript-study-notes/blob/main/src/%E7%BB%A7%E6%89%BF.html)

>参考链接
- [使用对象](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Guide/Working_with_Objects)
- [propertyIsEnumerable](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/propertyIsEnumerable)
- [Object.create()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/create)
- [Object](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object)
- [propertyIsEnumerable](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/propertyIsEnumerable)
- [对象原型](https://developer.mozilla.org/zh-CN/docs/Learn/JavaScript/Objects/Object_prototypes)
- [JavaScript 中的继承](https://developer.mozilla.org/zh-CN/docs/Learn/JavaScript/Objects)
- [Function.prototype](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Function/prototype)
- [Function](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Function#%E5%B1%9E%E6%80%A7%E5%92%8C%E6%96%B9%E6%B3%95)
