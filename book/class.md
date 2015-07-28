# Class

## Class基本语法

**（1）概述**

JavaScript语言的传统方法是通过构造函数，定义并生成新对象。下面是一个例子。

```javascript
function Point(x,y){
  this.x = x;
  this.y = y;
}

Point.prototype.toString = function () {
  return '(' + this.x + ', ' + this.y + ')';
}
```

上面这种写法跟传统的面向对象语言（比如C++和Java）差异很大，很容易让新学习这门语言的程序员感到困惑。

ES6提供了更接近传统语言的写法，引入了Class（类）这个概念，作为对象的模板。通过class关键字，可以定义类。基本上，ES6的class可以看作只是一个语法糖，它的绝大部分功能，ES5都可以做到，新的class写法只是让对象原型的写法更加清晰、更像面向对象编程的语法而已。上面的代码用ES6的“类”改写，就是下面这样。

```javascript
//定义类
class Point {

  constructor(x, y) {
    this.x = x;
    this.y = y;
  }

  toString() {
    return '('+this.x+', '+this.y+')';
  }

}
```

上面代码定义了一个“类”，可以看到里面有一个constructor方法，这就是构造方法，而this关键字则代表实例对象。也就是说，ES5的构造函数Point，对应ES6的Point类的构造方法。

Point类除了构造方法，还定义了一个toString方法。注意，定义“类”的方法的时候，前面不需要加上function这个保留字，直接把函数定义放进去了就可以了。

ES6的类，完全可以看作构造函数的另一种写法。

```javascript
class Point{
  // ...
}

typeof Point // "function"
```

上面代码表明，类的数据类型就是函数。

构造函数的prototype属性，在ES6的“类”上面继续存在。事实上，除了constructor方法以外，类的方法都定义在类的prototype属性上面。

```javascript
class Point {
  constructor(){
    // ...
  }

  toString(){
    // ...
  }

  toValue(){
    // ...
  }
}

// 等同于

Point.prototype = {
  toString(){},
  toValue(){}
}
```

由于类的方法（除constructor以外）都定义在prototype对象上面，所以类的新方法可以添加在prototype对象上面。`Object.assign`方法可以很方便地一次向类添加多个方法。

```javascript
class Point {
  constructor(){
    // ...
  }
}

Object.assign(Point.prototype, {
  toString(){},
  toValue(){}
})
```

prototype对象的constructor属性，直接指向“类”的本身，这与ES5的行为是一致的。

```javascript
Point.prototype.constructor === Point // true
```

另外，类的内部所有定义的方法，都是不可枚举的（enumerable）。

```javascript
class Point {
  constructor(x, y) {
    // ...
  }

  toString() {
    // ...
  }
}

Object.keys(Point.prototype)
// []
Object.getOwnPropertyNames(Point.prototype)
// ["constructor","toString"]
```

上面代码中，toString方法是Point类内部定义的方法，它是不可枚举的。这一点与ES5的行为不一致。

```javascript
var Point = function (x, y){
  // ...
}

Point.prototype.toString = function() {
  // ...
}

Object.keys(Point.prototype)
// ["toString"]
Object.getOwnPropertyNames(Point.prototype)
// ["constructor","toString"]
```

上面代码采用ES5的写法，toString方法就是可枚举的。

类的属性名，可以采用表达式。

```javascript
let methodName = "getArea";
class Square{
  constructor(length) {
    // ...
  }

  [methodName]() {
    // ...
  }
}
```

上面代码中，Square类的方法名getArea，是从表达式得到的。

**（2）constructor方法**

constructor方法是类的默认方法，通过new命令生成对象实例时，自动调用该方法。一个类必须有constructor方法，如果没有显式定义，一个空的constructor方法会被默认添加。

```javascript
constructor() {}
```

constructor方法默认返回实例对象（即this），完全可以指定返回另外一个对象。

```javascript
class Foo {
  constructor() {
    return Object.create(null);
  }
}

new Foo() instanceof Foo
// false
```

上面代码中，constructor函数返回一个全新的对象，结果导致实例对象不是Foo类的实例。

**（3）实例对象**

生成实例对象的写法，与ES5完全一样，也是使用new命令。如果忘记加上new，像函数那样调用Class，将会报错。

```javascript
// 报错
var point = Point(2, 3);

// 正确
var point = new Point(2, 3);
```

与ES5一样，实例的属性除非显式定义在其本身（即定义在this对象上），否则都是定义在原型上（即定义在class上）。

```javascript
//定义类
class Point {

  constructor(x, y) {
    this.x = x;
    this.y = y;
  }

  toString() {
    return '('+this.x+', '+this.y+')';
  }

}

var point = new Point(2, 3);

point.toString() // (2, 3)

point.hasOwnProperty('x') // true
point.hasOwnProperty('y') // true
point.hasOwnProperty('toString') // false
point.__proto__.hasOwnProperty('toString') // true
```

上面代码中，x和y都是实例对象point自身的属性（因为定义在this变量上），所以hasOwnProperty方法返回true，而toString是原型对象的属性（因为定义在Point类上），所以hasOwnProperty方法返回false。这些都与ES5的行为保持一致。

与ES5一样，类的所有实例共享一个原型对象。

```javascript
var p1 = new Point(2,3);
var p2 = new Point(3,2);

p1.__proto__ === p2.__proto__
//true
```

上面代码中，p1和p2都是Point的实例，它们的原型都是Point，所以\_\_proto\_\_属性是相等的。

这也意味着，可以通过实例的\_\_proto\_\_属性为Class添加方法。

```javascript
var p1 = new Point(2,3);
var p2 = new Point(3,2);

p1.__proto__.printName = function () { return 'Oops' };

p1.printName() // "Oops"
p2.printName() // "Oops"

var p3 = new Point(4,2);
p3.printName() // "Oops"
```

上面代码在p1的原型上添加了一个printName方法，由于p1的原型就是p2的原型，因此p2也可以调用这个方法。而且，此后新建的实例p3也可以调用这个方法。这意味着，使用实例的\_\_proto\_\_属性改写原型，必须相当谨慎，不推荐使用，因为这会改变Class的原始定义，影响到所有实例。

**（4）name属性**

由于本质上，ES6的Class只是ES5的构造函数的一层包装，所以函数的许多特性都被Class继承，包括name属性。

```javascript
class Point {}
Point.name // "Point"
```

name属性总是返回紧跟在class关键字后面的类名。

**（5）Class表达式**

与函数一样，Class也可以使用表达式的形式定义。

```javascript
const MyClass = class Me {
  getClassName() {
    return Me.name;
  }
};
```

上面代码使用表达式定义了一个类。需要注意的是，这个类的名字是MyClass而不是Me，Me只在Class的内部代码可用，指代当前类。

```javascript
let inst = new MyClass();
inst.getClassName() // Me
Me.name // ReferenceError: Me is not defined
```

上面代码表示，Me只在Class内部有定义。

如果Class内部没用到的话，可以省略Me，也就是可以写成下面的形式。

```javascript
const MyClass = class { /* ... */ };
```

采用Class表达式，可以写出立即执行的Class。

```javascript
let person = new class {
  constructor(name) {
    this.name = name;
  }

  sayName() {
    console.log(this.name);
  }
}("张三");

person.sayName(); // "张三"
```

上面代码中，person是一个立即执行的Class的实例。

**（6）不存在变量提升**

Class不存在变量提升（hoist），这一点与ES5完全不同。

```javascript
new Foo(); // ReferenceError
class Foo {}
```

上面代码中，Foo类使用在前，定义在后，这样会报错，因为ES6不会把变量声明提升到代码头部。这种规定的原因与下文要提到的继承有关，必须保证子类在父类之后定义。

```javascript
{
  let Foo = class {};
  class Bar extends Foo {
  }
}
```

如果存在Class的提升，上面代码将报错，因为let命令也是不提升的。

**（7）严格模式**

类和模块的内部，默认就是严格模式，所以不需要使用`use strict`指定运行模式。只要你的代码写在类或模块之中，就只有严格模式可用。

考虑到未来所有的代码，其实都是运行在模块之中，所以ES6实际上把整个语言升级到了严格模式。

## Class的继承

### 基本用法

Class之间可以通过extends关键字，实现继承，这比ES5的通过修改原型链实现继承，要清晰和方便很多。

```javascript
class ColorPoint extends Point {}
```

上面代码定义了一个ColorPoint类，该类通过extends关键字，继承了Point类的所有属性和方法。但是由于没有部署任何代码，所以这两个类完全一样，等于复制了一个Point类。下面，我们在ColorPoint内部加上代码。

```javascript
class ColorPoint extends Point {

  constructor(x, y, color) {
    super(x, y); // 调用父类的constructor(x, y)
    this.color = color;
  }

  toString() {
    return this.color + ' ' + super.toString(); // 调用父类的toString()
  }

}
```

上面代码中，constructor方法和toString方法之中，都出现了super关键字，它指代父类的实例（即父类的this对象）。

子类必须在constructor方法中调用super方法，否则新建实例时会报错。这是因为子类没有自己的this对象，而是继承父类的this对象，然后对其进行加工。如果不调用super方法，子类就得不到this对象。

```javascript
class Point { /* ... */ }

class ColorPoint extends Point {
  constructor() {
  }
}

let cp = new ColorPoint(); // ReferenceError
```

上面代码中，ColorPoint继承了父类Point，但是它的构造函数没有调用super方法，导致新建实例时报错。

ES5的继承，实质是先创造子类的实例对象this，然后再将父类的方法添加到this上面（`Parent.apply(this)`）。ES6的继承机制完全不同，实质是先创造父类的实例对象this（所以必须先调用super方法），然后再用子类的构造函数修改this。

如果子类没有定义constructor方法，这个方法会被默认添加，代码如下。也就是说，不管有没有显式定义，任何一个子类都有constructor方法。

```javascript
constructor(...args) {
  super(...args);
}
```

另一个需要注意的地方是，在子类的构造函数中，只有调用super之后，才可以使用this关键字，否则会报错。这是因为子类实例的构建，是基于对父类实例加工，只有super方法才能返回父类实例。

```javascript
class Point {
  constructor(x, y) {
    this.x = x;
    this.y = y;
  }
}

class ColorPoint extends Point {
  constructor(x, y, color) {
    this.color = color; // ReferenceError
    super(x, y);
    this.color = color; // 正确
  }
}
```

上面代码中，子类的constructor方法没有调用super之前，就使用this关键字，结果报错，而放在super方法之后就是正确的。

下面是生成子类实例的代码。

```javascript
let cp = new ColorPoint(25, 8, 'green');

cp instanceof ColorPoint // true
cp instanceof Point // true
```

上面代码中，实例对象cp同时是ColorPoint和Point两个类的实例，这与ES5的行为完全一致。

### 类的prototype属性和\_\_proto\_\_属性

在ES5中，每一个对象都有`__proto__`属性，指向对应的构造函数的prototype属性。Class作为构造函数的语法糖，同时有prototype属性和`__proto__`属性，因此同时存在两条继承链。

（1）子类的`__proto__`属性，表示构造函数的继承，总是指向父类。

（2）子类prototype属性的`__proto__`属性，表示方法的继承，总是指向父类的prototype属性。

```javascript
class A {
}

class B extends A {
}

B.__proto__ === A // true
B.prototype.__proto__ === A.prototype // true
```

上面代码中，子类A的`__proto__`属性指向父类B，子类A的prototype属性的__proto__属性指向父类B的prototype属性。

这两条继承链，可以这样理解：作为一个对象，子类（B）的原型（`__proto__属性`）是父类（A）；作为一个构造函数，子类（B）的原型（prototype属性）是父类的实例。

```javascript
B.prototype = new A();
// 等同于
B.prototype.__proto__ = A.prototype;
```

此外，考虑三种特殊情况。第一种特殊情况，子类继承Object类。

```javascript
class A extends Object {
}

A.__proto__ === Object // true
A.prototype.__proto__ === Object.prototype // true
```

这种情况下，A其实就是构造函数Object的复制，A的实例就是Object的实例。

第二种特性情况，不存在任何继承。

```javascript
class A {
}

A.__proto__ === Function.prototype // true
A.prototype.__proto__ === Object.prototype // true
```

这种情况下，A作为一个基类（即不存在任何继承），就是一个普通函数，所以直接继承`Funciton.prototype`。但是，A调用后返回一个空对象（即Object实例），所以`A.prototype.__proto__`指向构造函数（Object）的prototype属性。

第三种特殊情况，子类继承null。

```javascript
class A extends null {
}

A.__proto__ === Function.prototype // true
A.prototype.__proto__ === null // true
```

这种情况与第二种情况非常像。A也是一个普通函数，所以直接继承`Funciton.prototype`。但是，A调用后返回的对象不继承任何方法，所以它的`__proto__`指向`Function.prototype`，即实质上执行了下面的代码。

```javascript
class C extends null {
  constructor() { return Object.create(null); }
}
```

### Object.getPrototypeOf()

Object.getPrototypeOf方法可以用来从子类上获取父类。

```javascript
Object.getPrototypeOf(ColorPoint) === Point
// true
```

### 实例的\_\_proto\_\_属性

父类实例和子类实例的\_\_proto\_\_属性，指向是不一样的。

```javascript
var p1 = new Point(2, 3);
var p2 = new ColorPoint(2, 3, 'red');

p2.__proto__ === p1.__proto // false
p2.__proto__.__proto__ === p1.__proto__ // true
```

通过子类实例的\_\_proto\_\_属性，可以修改父类实例的行为。

```javascript
p2.__proto__.__proto__.printName = function () {
  console.log('Ha');
};

p1.printName() // "Ha"
```

上面代码在ColorPoint的实例p2上向Point类添加方法，结果影响到了Point的实例p1。

### 原生构造函数的继承

原生构造函数是指语言内置的构造函数，通常用来生成数据结构，比如`Array()`。以前，这些原生构造函数是无法继承的，即不能自己定义一个Array的子类。

```javascript
function MyArray() {
  Array.apply(this, arguments);
}

MyArray.prototype = Object.create(Array.prototype, {
  constructor: {
    value: MyArray,
    writable: true,
    configurable: true,
    enumerable: true
  }
});
```

上面代码定义了一个继承Array的MyArray类。但是，这个类的行为与Array完全不一致。

```javascript
var colors = new MyArray();
colors[0] = "red";
colors.length  // 0

colors.length = 0;
colors[0]  // "red"
```

之所以会发生这种情况，是因为原生构造函数无法外部获取，通过`Array.apply()`或者分配给原型对象都不行。ES5是先新建子类的实例对象this，再将父类的属性添加到子类上，由于父类的属性无法获取，导致无法继承原生的构造函数。

ES6允许继承原生构造函数定义子类，因为ES6是先新建父类的实例对象this，然后再用子类的构造函数修饰this，使得父类的所有行为都可以继承。下面是一个继承Array的例子。

```javascript
class MyArray extends Array {
  constructor(...args) {
    super(...args);
  }
}

var arr = new MyArray();
arr[0] = 12;
arr.length // 1

arr.length = 0;
arr[0] // undefined
```

上面代码定义了一个MyArray类，继承了Array构造函数，因此就可以从MyArray生成数组的实例。这意味着，ES6可以自定义原生数据结构（比如Array、String等）的子类，这是ES5无法做到的。

上面这个例子也说明，extends关键字不仅可以用来继承类，还可以用来继承原生的构造函数。下面是一个自定义Error子类的例子。

```javascript
class MyError extends Error {
}

throw new MyError('Something happened!');
```

## class的取值函数（getter）和存值函数（setter）

与ES5一样，在Class内部可以使用get和set关键字，对某个属性设置存值函数和取值函数，拦截该属性的存取行为。

```javascript
class MyClass {
  constructor() {
    // ...
  }
  get prop() {
    return 'getter';
  }
  set prop(value) {
    console.log('setter: '+value);
  }
}

let inst = new MyClass();

inst.prop = 123;
// setter: 123

inst.prop
// 'getter'
```

上面代码中，prop属性有对应的存值函数和取值函数，因此赋值和读取行为都被自定义了。

存值函数和取值函数是设置在属性的descriptor对象上的。

```javascript
class CustomHTMLElement {
  constructor(element) {
    this.element = element;
  }

  get html() {
    return this.element.innerHTML;
  }

  set html(value) {
    this.element.innerHTML = value;
  }
}

var descriptor = Object.getOwnPropertyDescriptor(
  CustomHTMLElement.prototype, "html");
"get" in descriptor  // true
"set" in descriptor  // true
```

上面代码中，存值函数和取值函数是定义在html属性的描述对象上面，这与ES5完全一致。

下面的例子针对所有属性，设置存值函数和取值函数。

```javascript
class Jedi {
  constructor(options = {}) {
    // ...
  }

  set(key, val) {
    this[key] = val;
  }

  get(key) {
    return this[key];
  }
}
```

上面代码中，Jedi实例所有属性的存取，都会通过存值函数和取值函数。

## Class的Generator方法

如果某个方法之前加上星号（*），就表示该方法是一个Generator函数。

```javascript
class Foo {
  constructor(...args) {
    this.args = args;
  }
  * [Symbol.iterator]() {
    for (let arg of this.args) {
      yield arg;
    }
  }
}

for (let x of new Foo('hello', 'world')) {
  console.log(x);
}
// hello
// world
```

上面代码中，Foo类的Symbol.iterator方法前有一个星号，表示该方法是一个Generator函数。Symbol.iterator方法返回一个Foo类的默认遍历器，for...of循环会自动调用这个遍历器。

## Class的静态方法

类相当于实例的原型，所有在类中定义的方法，都会被实例继承。如果在一个方法前，加上static关键字，就表示该方法不会被实例继承，而是直接通过类来调用，这就称为“静态方法”。

```javascript
class Foo {
  static classMethod() {
    return 'hello';
  }
}

Foo.classMethod() // 'hello'

var foo = new Foo();
foo.classMethod()
// TypeError: undefined is not a function
```

上面代码中，Foo类的classMethod方法前有static关键字，表明该方法是一个静态方法，可以直接在Foo类上调用（`Foo.classMethod()`），而不是在Foo类的实例上调用。如果在实例上调用静态方法，会抛出一个错误，表示不存在该方法。

父类的静态方法，可以被子类继承。

```javascript
class Foo {
  static classMethod() {
    return 'hello';
  }
}

class Bar extends Foo {
}

Bar.classMethod(); // 'hello'
```

上面代码中，父类Foo有一个静态方法，子类Bar可以调用这个方法。

静态方法也是可以从super对象上调用的。

```javascript
class Foo {
  static classMethod() {
    return 'hello';
  }
}

class Bar extends Foo {
  static classMethod() {
    return super.classMethod() + ', too';
  }
}

Bar.classMethod();
```

## new.target属性

new是从构造函数生成实例的命令。ES6为new命令引入了一个`new.target`属性，（在构造函数中）返回new命令作用于的那个构造函数。如果构造函数不是通过new命令调用的，`new.target`会返回undefined，因此这个属性可以用来确定构造函数是怎么调用的。

```javascript
function Person(name) {
  if (new.target !== undefined) {
    this.name = name;
  } else {
    throw new Error('必须使用new生成实例');
  }
}

// 另一种写法
function Person(name) {
  if (new.target === Person) {
    this.name = name;
  } else {
    throw new Error('必须使用new生成实例');
  }
}

var person = new Person('张三'); // 正确
var notAPerson = Person.call(person, '张三');  // 报错
```

上面代码确保构造函数只能通过new命令调用。

Class内部调用`new.target`，返回当前Class。

```javascript
class Rectangle {
  constructor(length, width) {
    console.log(new.target === Rectangle);
    this.length = length;
    this.width = width;
  }
}

var obj = new Rectangle(3, 4); // 输出 true
```

需要注意的是，子类继承父类时，`new.target`会返回子类。

```javascript
class Rectangle {
  constructor(length, width) {
    console.log(new.target === Rectangle);
    // ...
  }
}

class Square extends Rectangle {
  constructor(length) {
    super(length, length);
  }
}

var obj = new Square(3); // 输出 false
```

上面代码中，`new.target`会返回子类。

利用这个特点，可以写出不能独立使用、必须继承后才能使用的类。

```javascript
class Shape {
  constructor() {
    if (new.target === Shape) {
      throw new Error('本类不能实例化');
    }
  }
}

class Rectangle extends Shape {
  constructor(length, width) {
    super();
    // ...
  }
}

var x = new Shape();  // 报错
var y = new Rectangle(3, 4);  // 正确
```

上面代码中，Shape类不能被实例化，只能用于继承。

注意，在函数外部，使用`new.target`会报错。

## 修饰器

### 类的修饰

修饰器（Decorator）是一个表达式，用来修改类的行为。这是ES7的一个[提案](https://github.com/wycats/javascript-decorators)，目前Babel转码器已经支持。

修饰器对类的行为的改变，是代码编译时发生的，而不是在运行时。这意味着，修饰器能在编译阶段运行代码。

```javascript
function testable(target) {
  target.isTestable = true;
}

@testable
class MyTestableClass () {}

console.log(MyTestableClass.isTestable) // true
```

上面代码中，`@testable`就是一个修饰器。它修改了MyTestableClass这个类的行为，为它加上了静态属性isTestable。

修饰器函数可以接受三个参数，依次是目标函数、属性名和该属性的描述对象。后两个参数可省略。上面代码中，testable函数的参数target，就是所要修饰的对象。如果希望修饰器的行为，能够根据目标对象的不同而不同，就要在外面再封装一层函数。

```javascript
function testable(isTestable) {
  return function(target) {
    target.isTestable = isTestable;
  }
}

@testable(true) class MyTestableClass () {}
console.log(MyTestableClass.isTestable) // true

@testable(false) class MyClass () {}
console.log(MyClass.isTestable) // false
```

上面代码中，修饰器testable可以接受参数，这就等于可以修改修饰器的行为。

如果想要为类的实例添加方法，可以在修饰器函数中，为目标类的prototype属性添加方法。

```javascript
function testable(target) {
  target.prototype.isTestable = true;
}

@testable
class MyTestableClass () {}

let obj = new MyClass();

console.log(obj.isTestable) // true
```

上面代码中，修饰器函数testable是在目标类的prototype属性添加属性，因此就可以在类的实例上调用添加的属性。

下面是另外一个例子。

```javascript
// mixins.js
export function mixins(...list) {
  return function (target) {
    Object.assign(target.prototype, ...list)
  }
}

// main.js
import { mixins } from './mixins'

const Foo = {
  foo() { console.log('foo') }
}

@mixins(Foo)
class MyClass {}

let obj = new MyClass()

obj.foo() // 'foo'
```

上面代码通过修饰器mixins，可以为类添加指定的方法。

修饰器可以用`Object.assign()`模拟。

```javascript
const Foo = {
  foo() { console.log('foo') }
}

class MyClass {}

Object.assign(MyClass.prototype, Foo);

let obj = new MyClass();
obj.foo() // 'foo'
```

### 方法的修饰

修饰器不仅可以修饰类，还可以修饰类的属性。

```javascript
class Person {
  @readonly
  name() { return `${this.first} ${this.last}` }
}
```

上面代码中，修饰器readonly用来修饰”类“的name方法。

此时，修饰器函数一共可以接受三个参数，第一个参数是所要修饰的目标对象，第二个参数是所要修饰的属性名，第三个参数是该属性的描述对象。

```javascript
readonly(Person.prototype, 'name', descriptor);

function readonly(target, name, descriptor){
  // descriptor对象原来的值如下
  // {
  //   value: specifiedFunction,
  //   enumerable: false,
  //   configurable: true,
  //   writable: true
  // };
  descriptor.writable = false;
  return descriptor;
}

Object.defineProperty(Person.prototype, 'name', descriptor);
```

上面代码说明，修饰器（readonly）会修改属性的描述对象（descriptor），然后被修改的描述对象再用来定义属性。下面是另一个例子。

```javascript
class Person {
  @nonenumerable
  get kidCount() { return this.children.length; }
}

function nonenumerable(target, name, descriptor) {
  descriptor.enumerable = false;
  return descriptor;
}
```

修饰器有注释的作用。

```javascript
@testable
class Person {
  @readonly
  @nonenumerable
  name() { return `${this.first} ${this.last}` }
}
```

从上面代码中，我们一眼就能看出，MyTestableClass类是可测试的，而name方法是只读和不可枚举的。

除了注释，修饰器还能用来类型检查。所以，对于Class来说，这项功能相当有用。从长期来看，它将是JavaScript代码静态分析的重要工具。

### core-decorators.js

[core-decorators.js](https://github.com/jayphelps/core-decorators.js)是一个第三方模块，提供了几个常见的修饰器，通过它可以更好地理解修饰器。

**（1）@autobind**

autobind修饰器使得方法中的this对象，绑定原始对象。

```javascript
import { autobind } from 'core-decorators';

class Person {
  @autobind
  getPerson() {
    return this;
  }
}

let person = new Person();
let getPerson = person.getPerson;

getPerson() === person;
// true
```

**（2）@readonly**

readonly修饰器是的属性或方法不可写。

```javascript
import { readonly } from 'core-decorators';

class Meal {
  @readonly
  entree = 'steak';
}

var dinner = new Meal();
dinner.entree = 'salmon';
// Cannot assign to read only property 'entree' of [object Object]
```

**（3）@override**

override修饰器检查子类的方法，是否正确覆盖了父类的同名方法，如果不正确会报错。

```javascript
import { override } from 'core-decorators';

class Parent {
  speak(first, second) {}
}

class Child extends Parent {
  @override
  speak() {}
  // SyntaxError: Child#speak() does not properly override Parent#speak(first, second)
}

// or

class Child extends Parent {
  @override
  speaks() {}
  // SyntaxError: No descriptor matching Child#speaks() was found on the prototype chain.
  //
  //   Did you mean "speak"?
}
```

**（4）@deprecate (别名@deprecated)**

deprecate或deprecated修饰器在控制台显示一条警告，表示该方法将废除。

```javascript
import { deprecate } from 'core-decorators';

class Person {
  @deprecate
  facepalm() {}

  @deprecate('We stopped facepalming')
  facepalmHard() {}

  @deprecate('We stopped facepalming', { url: 'http://knowyourmeme.com/memes/facepalm' })
  facepalmHarder() {}
}

let person = new Person();

person.facepalm();
// DEPRECATION Person#facepalm: This function will be removed in future versions.

person.facepalmHard();
// DEPRECATION Person#facepalmHard: We stopped facepalming

person.facepalmHarder();
// DEPRECATION Person#facepalmHarder: We stopped facepalming
//
//     See http://knowyourmeme.com/memes/facepalm for more details.
//
```

**（5）@suppressWarnings**

suppressWarnings修饰器抑制decorated修饰器导致的`console.warn()`调用。但是，异步代码出发的调用除外。

```javascript
import { suppressWarnings } from 'core-decorators';

class Person {
  @deprecated
  facepalm() {}

  @suppressWarnings
  facepalmWithoutWarning() {
    this.facepalm();
  }
}

let person = new Person();

person.facepalmWithoutWarning();
// no warning is logged
```

### Mixin

在修饰器的基础上，可以实现Mixin模式。所谓Mixin模式，就是对象继承的一种替代方案，中文译为“混入”（mix in），意为在一个对象之中混入另外一个对象的方法。

请看下面的例子。

```javascript
const Foo = {
  foo() { console.log('foo') }
};

class MyClass {}

Object.assign(MyClass.prototype, Foo);

let obj = new MyClass();
obj.foo() // 'foo'
```

上面代码之中，对象Foo有一个foo方法，通过`Object.assign`方法，可以将foo方法“混入”MyClass类，导致MyClass的实例obj对象都具有foo方法。这就是“混入”模式的一个简单实现。

下面，我们部署一个通用脚本`mixins.js`，将mixin写成一个修饰器。

```javascript
export function mixins(...list) {
  return function (target) {
    Object.assign(target.prototype, ...list);
  };
}
```

然后，就可以使用上面这个修饰器，为类“混入”各种方法。

```javascript
import { mixins } from './mixins'

const Foo = {
  foo() { console.log('foo') }
};

@mixins(Foo)
class MyClass {}

let obj = new MyClass();

obj.foo() // "foo"
```

通过mixins这个修饰器，实现了在MyClass类上面“混入”Foo对象的foo方法。

### Trait

Trait也是一种修饰器，功能与Mixin类型，但是提供更多功能，比如防止同名方法的冲突、排除混入某些方法、为混入的方法起别名等等。

下面采用[traits-decorator](https://github.com/CocktailJS/traits-decorator)这个第三方模块作为例子。这个模块提供的traits修饰器，不仅可以接受对象，还可以接受ES6类作为参数。

```javascript
import {traits } from 'traits-decorator'

class TFoo {
  foo() { console.log('foo') }
}

const TBar = {
  bar() { console.log('bar') }
}

@traits(TFoo, TBar)
class MyClass { }

let obj = new MyClass()
obj.foo() // foo
obj.bar() // bar
```

上面代码中，通过traits修饰器，在MyClass类上面“混入”了TFoo类的foo方法和TBar对象的bar方法。

Trait不允许“混入”同名方法。

```javascript
import {traits } from 'traits-decorator'

class TFoo {
  foo() { console.log('foo') }
}

const TBar = {
  bar() { console.log('bar') },
  foo() { console.log('foo') }
}

@traits(TFoo, TBar)
class MyClass { }
// 报错
// throw new Error('Method named: ' + methodName + ' is defined twice.');
//        ^
// Error: Method named: foo is defined twice.
```

上面代码中，TFoo和TBar都有foo方法，结果traits修饰器报错。

一种解决方法是排除TBar的foo方法。

```javascript
import { traits, excludes } from 'traits-decorator'

class TFoo {
  foo() { console.log('foo') }
}

const TBar = {
  bar() { console.log('bar') },
  foo() { console.log('foo') }
}

@traits(TFoo, TBar::excludes('foo'))
class MyClass { }

let obj = new MyClass()
obj.foo() // foo
obj.bar() // bar
```

上面代码使用绑定运算符（::）在TBar上排除foo方法，混入时就不会报错了。

另一种方法是为TBar的foo方法起一个别名。

```javascript
import { traits, alias } from 'traits-decorator'

class TFoo {
  foo() { console.log('foo') }
}

const TBar = {
  bar() { console.log('bar') },
  foo() { console.log('foo') }
}

@traits(TFoo, TBar::alias({foo: 'aliasFoo'}))
class MyClass { }

let obj = new MyClass()
obj.foo() // foo
obj.aliasFoo() // foo
obj.bar() // bar
```

上面代码为TBar的foo方法起了别名aliasFoo，于是MyClass也可以混入TBar的foo方法了。

alias和excludes方法，可以结合起来使用。

```javascript
@traits(TExample::excludes('foo','bar')::alias({baz:'exampleBaz'}))
class MyClass {}
```

上面代码排除了TExample的foo方法和bar方法，为baz方法起了别名exampleBaz。

as方法则为上面的代码提供了另一种写法。

```javascript
@traits(TExample::as({excludes:['foo', 'bar'], alias: {baz: 'exampleBaz'}}))
class MyClass {}
```

### Babel转码器的支持

目前，Babel转码器已经支持Decorator，命令行的用法如下。

```bash
$ babel --optional es7.decorators
```

脚本中打开的命令如下。

```javascript
babel.transfrom("code", {optional: ["es7.decorators"]})
```

Babel的官方网站提供一个[在线转码器](https://babeljs.io/repl/)，只要勾选Experimental，就能支持Decorator的在线转码。
