---
title: "es基础"
date: 2024-04-11 11:33:47
categories: "前端"
tags: ["es6"]
summary: "es6的一些基础知识"
---

[代码](https://github.com/Luo25177/Web.git)

## 简介

es 与 js 的关系就是，前者是后者的规范，后者是前者的一种实现

## 环境配置

### nodejs

nodejs 的下载安装与一般的软件无异，从官网找到并且下载安装就好

### npm

需要使用镜像下载： [镜像地址](https://npmmirror.com/)

直接在终端执行下列命令就可完成安装

```shell
npm install -g cnpm --registry=https://npmmirror.com/
```

## 语法

### let关键字

特点

- 声明的变量只在当前的代码块中有效
- 不满足变量提升，也就是必须在声明之后使用变量，不然会报错
- 不允许重复声明同一个变量


```es6
{
  let a = 0;
}
a = 1; // 报错
```

### const关键字

特点

- 声明之后变量不可改变
- 声明时必须初始化
- 不满足变量提升，也就是必须在声明之后使用变量，不然会报错
- 只在当前的代码块中有效
- 不允许重复声明同一个变量

```es6
const a = 1;
a = 2; // 报错
```

### 变量解构赋值

解构可以用于对象，也可以很方便的将现有的对象的方法赋值到某个变量，对象的顺序没有要求，但是必须与属性同名才能得到正确的值

```JavaScript
var {age, name} = {name:"1", age:10};
console.log(name, age);
```

### 字符串扩展

- 在 es6 中增加了字符串对 `unicode` 字符码的支持，可以在代码中使用 `\uxxx` 来表示一个字符
  - `unicode` 码即统一码，是计算机科学领域里的一项业界标准，包括字符集，编码方案等， `unicode` 是为了解决传统的字符编码方案的局限而产生的，为每种语言的每个字符都设定了统一并且唯一的二进制编码
- 利用 `for...of` 语句实现字符串的遍历
- 模板字符串，是增强版的字符串，使用反引号 ` 标记，可以当作普通字符串使用，也可用来定义多行字符串，或者在字符串中嵌入变量
- `include()` 表示字符串中是否包含参数字符串，返回布尔值，支持第二个参数，表示开始搜索的位置，默认为0
- `srartsWith()` 返回布尔值，表示字符串是否以参数字符串开头，同上
- `endsWith()` 返回布尔值，表示字符串是否以参数字符串结尾，同上
- `repeat()` 返回一个由源字符串重复 `n` 次的字符串
- `padStart()` 如果不够指定长度，在头部补全
- `padEnd()` 如果不够指定长度，在尾部补全
- `trimStart()` 消除字符串头部的空格，返回新字符串，不改变源字符串
- `trimEnd()` 消除字符串头部的空格，同上
- `at()` 返回指定位置的字符，支持负索引

### 数组扩展

- 扩展运算符，三个点，将一个数组转为用逗号分割的参数序列
  - 将数组转为函数的参数，不需要再使用 `apply` 方法了
  - 作为合并数组的新写法
- `Array.from()` 用于将类数组转为真正的数组，常见类数组有三种
  - arguments
  - 元素集合
  - 类似数组的对象，但是书写要求较高，有严格的标准
- `Array.of()` 将一组数值转为数组

### 对象扩展

- es6 允许再大括号内，直接写入变量和函数，作为对象的属性和方法，用于函数的返回值
- 属性名表达式，允许字面量定义对象，用表达式作为对象的属性名，就是把标达式放在括号内
- 对象的扩展运算符，三个点，相当于将对象展开

### 函数扩展

- 箭头函数，es6 允许使用箭头 `=>` 定义函数
  - 箭头函数如果不需要参数或者需要多个参数，就使用一个圆括号代表参数部分
  - 如果箭头函数的代码块部分多于一条语句，就使用大括号将它们括起来，使用 `return` 返回
  - 箭头函数的作用就是简化回调函数
  - 对于普通函数，内部的 `this` 指向函数运行时所在的对象，但是对于箭头函数，内部的 `this` 就是定义时上层作用域中的 `this` ，箭头函数没有自己的 `this` 只能调用上层的 `this`

### Set

- es6 提供了新的数据结构 `Set` ，类似于数组，但是成员的数值是唯一的，没有重复的值
- `Set` 本身是一个构造函数，用来生成 `Set` 数据结构
- 通过 `add` 方法加入成员，而且 `Set` 不会添加重复的值
- 可以接收一个数组作为参数
- 可以用作数组去除重复项的方法
- 字符串去除重复字符
- 加入值时，数据类型不会发生转换，所以字符和数字是会分别的
- `size` 返回实例的成员总数
- `add()` 添加元素
- `delete()` 删除某个值，返回一个布尔值，表示删除操作是否成功
- `has()` 是否有某个元素
- `clear()` 清除所有元素

### promise

是异步编程的一种解决办法， `Promise` 是一个对象，从它可以获取异步操作的消息

有三种 `promise` 状态，除了异步操作的结果，任何其他操作都无法改变这个状态，只有从 `pending` 变为 `fulfilled` 和从 `pending` 变为 `rejected` 的状态改变。只要处于 `fulfilled` 和 `rejected` ，状态就不会再变了即 `resolved`

- `pending` 进行中
- `fulfilled` 已成功
- `rejected` 已失败

**缺点**

- 无法取消 `Promise` ，一旦新建它就会立即执行，无法中途取消
- 如果不设置回调函数， `Promise` 内部抛出的错误，不会反应到外部
- 当处于 `pending` 状态时，无法得知目前进展到哪一个阶段（刚刚开始还是即将完成）

**then方法**

- 接收两个函数作为参数，第一个参数是 `Promise` 执行成功时的回调，第二个参数是 `Promise` 执行失败时的回调，两个函数只会有一个被调用
- 当事件队列的当前运行完成之前，回调函数永不会被调用，
- `then` 方法可以返回值作为下一个链式反应的值
- 当忘记返回值时，对应链条被打破，导致之后的链式反应与当前链式反应同时进行
- 大多数浏览器中不能终止的 `Promise` 链里的 `rejection` ，建议后面都跟上 `.catch(error => console.log(error));`

### Async函数

使得异步操作更方便，可以把异步操作变为同步操作，但是需要使用 `await` 关键字来等待异步操作完成，以此来实现同步

### 类的使用

`js` 中，生成实例对象的传统方法是通过构造函数来实现，在 `es6` 中引入了 `class` ，通过这个关键字可以定义类

- 实例化需要使用 `new` 来实现
- 类不存在提升，也就是只能在声明之后使用

**实例的属性**

是指类的对象实例可以调用的属性，一般就是在构造函数中所声明并且定义的属性

**静态方法**

所有在类中定义的方法都会被实例继承，但是对于 `static` 方法不会被实例继承，但是可以直接调用，而实例中却没有此方法，如果静态方法中包含 `this` 那这个就是指这个类，而不是实例

**静态属性**

就是 `class` 本身的属性，也就是 `Class.propname` ，这个可以在类之外定义，但是不会被实例继承

### module语法

`js` 没有将一个大的程序拆分成一些相互有依赖关系的小文件，再用简单的方式拼接起来，但是在 `es6` 中引入了 `module` 方法来实现此功能

先通过 `export` 指令显示指定来自外部的代码，再通过 `import` 命令输入

```es6
export var hello = "hello";
import{hello} form "./hello.js";
```

测试需要采用 `Nodejs` 的方式测试 `module` 语法

### Generator函数

`Generator` 函数有两个区分于普通函数的部分

- 在 `function` 之后，函数名之前有个 `*`
- 函数内部有 `yield` 表达式，用 `yield` 来定义函数内部的状态

```es6
function *func() {
  console.log("one");
  yield '1';
  console.log("two");
  return '2';
}
```

**执行机制**

调用 `Generator` 函数和调用普通函数一样，在函数名后加上 `()` 即可，但是该函数不会像普通函数一样立即执行，而是返回一个指向内部状态的指针，所以要调用遍历器对象 `iterator` 的 `next` 方法，指针就会从函数头部或者上一次停下来的地方开始执行，类似于 `python` 中的生成器函数，每调用一次都会执行到下一个 `yield` 或者 `return`

```es6
func.next(); // 'one' value='1'
func.next(); // 'two' value='2'
func.next(); // value=undefined
```

需要注意的是，如果获取 `yield` 的值，那每一步 `yield` 之后，执行下一次这个参数会变成 `undefined` ，但是如果使用 `next` 传参的话，这个数据就会被附上值，总之有点复杂，可以直接看运行结果

```es6
function *func() {
  console.log("one");
  var x = yield '1';
  console.log("x = " + x);
  console.log("two");
  return '2';
}
var f = func();
console.log(f.next()); 
console.log(f.next());// x = undefined
console.log(f.next());

var f1 = func();
console.log(f1.next(10)); 
console.log(f1.next(20)); // x = 20
```

除了使用 `next` 遍历该函数，还可以使用 `for...of` 来实现遍历

```es6
var f3 = func();
for (var item of f3) {
  item;
}
```

**return方法**

返回给定值，并且结束遍历 `Generator` 函数

提供参数时返回该参数，不提供参数时，返回 `undefined`

**错误处理**

对于下列 `Generator` 函数

```es6
var func2 = function* () {
  try {
    yield;
  } catch (e) {
    console.log('catch inner', e);
  }
};
var f2 = func2();
f2.next;
try {
  f2.throw('a');
  f2.throw('b');
} catch (e) {
  console.log('catch outside', e);
}
```

遍历器对象会抛出两个错误，第一个被 `Generator` 函数内部捕获，另一个因为函数体内部的 `catch` 函数已经执行过了，不会再捕获这个错误，所以这个错误就抛出 `Generator` 函数体，被函数体外的 `catch` 捕获

**yield*表达式**

`yield*` 表达式表示 `yield` 返回一个遍历器对象，用于在 `Generator` 函数内部，调用另一个 `Generator` 函数

```es6
function* callee() {
  console.log('callee: ' + (yield));
}
function* caller() {
  while (true) {
    yield* callee();
  }
}
```

### Reflext与Proxy

- `Proxy` 可以对目标对象的读取、函数调用等操作进行拦截，然后进行操作处理。它不直接操作对象，而是像代理模式，通过对象的代理对象进行操作，在进行这些操作时，可以添加一些需要的额外操作。 
- `Reflect` 可以用于获取目标对象的行为，它与 `Object` 类似，但是更易读，为操作对象提供了一种更优雅的方式。它的方法与 `Proxy` 是对应的
- `Reflect` 对象的方法与 `Proxy` 对象的方法是一一对应的。所以 `Proxy` 对象的方法可以通过调用 `Reflect` 对象的方法获取默认行为，然后进行额外操作

### Proxy

一个 `Proxy` 对象由两个部分组成： `target` 和 `handler` 。在通过 `Proxy` 构造函数生成实例对象时，需要提供这两个参数。 `target` 即目标对象， `handler` 是一个对象，声明了代理 `target` 的指定行为

```es6
let target = {
  name: 'Tom',
  age: 24
}
let handler = {
  get: function(target, key) {
    console.log('getting '+key);
    return target[key]; // 不是target.key
  },
  set: function(target, key, value) {
    console.log('setting '+key);
    target[key] = value;
  }
}
let proxy = new Proxy(target, handler)
proxy.name
proxy.age = 25 
proxy.sex = "man"
```

- `target` 可以为空对象
- `handler` 对象也可以为空，相当于不设置拦截操作，直接访问目标对象
- 通过构造函数新建实例时其实是对目标对象进行了浅拷贝，因此目标对象与代理对象会互相影响

**实例方法**

- `get(target, propKey, receiver)` 用于 `target` 对象上的 `propKey` 的读取操作，而且 `get` 方法可以继承
- `set(target, propKey, value, receiver)` 用于拦截 `target` 对象上的 `propKey` 的赋值操作，如果目标对象自身的某个属性不可写且不可配置，那 `set` 方法将不起作用，第四个参数 `receiver` 表示原始操作行为所在对象，一般是 `Proxy` 实例本身，严格模式下， `set` 代理如果没有返回 `true` ，就会报错
- `apply(target, ctx, args)` 用于拦截函数的调用， `call` 和 `reply` 操作，而 `target` 表示目标对象， `ctx` 表示目标对象上下文， `args` 表示目标对象的参数数组
- `has(target, propKey)` 用于拦截 `HasProperty` 操作，即在判断 `target` 对象是否存在 `propKey` 属性时，会被这个方法拦截。此方法不判断一个属性是对象自身的属性，还是继承的属性，此方法不拦截 `for...in` 循环
- `construct(target, args)` 此方法用于拦截 `new` 指令，返回值必须为对象
- `deleteProperty(target, propKey)` 用于拦截 `delete` 操作，如果这个方法抛出错误或者返回 `false` ， `propKey` 属性就无法被 `delete` 命令删除
- `defineProperty(target, propKey, propDesc)` 用于拦截 `Object.definePro` 若目标对象不可扩展，增加目标对象上不存在的属性会报错；若属性不可写或不可配置，则不能改变这些属性
- `getOwnPropertyDescriptor(target, propKey)` 用于拦截 `Object.getOwnPropertyD()` 返回值为属性描述对象或者 `undefined`
- `getPrototypeOf(target)` 主要用于拦截获取对象原型的操作，返回值必须是对象或者 `null` ，否则报错。另外，如果目标对象不可扩展（non-extensible）， `getPrototypeOf` 方法必须返回目标对象的原型对象，主要包括
  - `Object.prototype._proto_`
  - `Object.prototype.isPrototypeOf()`
  - `Object.getPrototypeOf()`
  - `Reflect.getPrototypeOf()`
  - `instanceof`
- `isExtensible(target)` 用于拦截 `Object.isExtensible` 操作，该方法只能返回布尔值，否则返回值会被自动转为布尔值，它的返回值必须与目标对象的isExtensible属性保持一致，否则会抛出错误
- `ownKeys(target)` 用于拦截对象自身属性的读取操作，方法返回的数组成员，只能是字符串或 Symbol 值，否则会报错。若目标对象中含有不可配置的属性，则必须将这些属性在结果中返回，否则就会报错。若目标对象不可扩展，则必须全部返回且只能返回目标对象包含的所有属性，不能包含不存在的属性，否则也会报错。主要包括
  - `Object.getOwnPropertyNames()`
  - `Object.getOwnPropertySymbols()`
  - `Object.keys()`
  - `or...in`
- `preventExtensions(target)` 拦截 `Object.preventExtensions` 操作。该方法必须返回一个布尔值，否则会自动转为布尔值
- `setPrototypeOf()` 主要用来拦截 `Object.setPrototypeOf` 方法。返回值必须为布尔值，否则会被自动转为布尔值。若目标对象不可扩展， `setPrototypeOf` 方法不得改变目标对象的原型
- `Proxy.revocable()` 用于返回一个可取消的 `Proxy` 实例

### Reflect

es6 中将 `Object` 的一些明显属于语言内部的方法移植到了 `Reflect` 对象上（当前某些方法会同时存在于 `Object` 和 `Reflect` 对象上），未来的新方法会只部署在 `Reflect` 对象上。 `Reflect` 对象对某些方法的返回结果进行了修改，使其更合理。 `Reflect` 对象使用函数的方式实现了 `Object` 的命令式操作

**静态方法**

- `Reflect.get(target, name, receiver)` 查找并返回 `target` 对象的 `name` 属性
- `Reflect.set(target, name, value, receiver)` 将 `target` 的 `name` 属性设置为 `value` 。返回值为布尔值， `true` 表示修改成功，`false` 表示失败。当 `target` 为不存在的对象时，会报错
- `Reflect.has(obj, name)` 是 `name in obj` 指令的函数化，用于查找 `name` 属性在 `obj` 对象中是否存在。返回值为布尔值。如果 `obj` 不是对象则会报错 `TypeError`
- `Reflect.deleteProperty(obj, property)` 是 `delete obj[property]` 的函数化，用于删除 `obj` 对象的 `property` 属性，返回值为布尔值。如果 `obj` 不是对象则会报错 `TypeError`
- `Reflect.construct(obj, args)` 等同于 `new target(...args)`
- `Reflect.getPrototypeOf(obj)` 用于读取 `obj` 的 `_proto_` 属性。在 `obj` 不是对象时不会像 `Object` 一样把 `obj` 转为对象，而是会报错
- `Reflect.setPrototypeOf(obj, newProto)` 用于设置目标对象的 `prototype`
- `Reflect.apply(func, thisArg, args)` 等同于 `Function.prototype.apply.call(func, thisArg, args)` 。 `func` 表示目标函数， `thisArg` 表示目标函数绑定的 `this` 对象， `args` 表示目标函数调用时传入的参数列表，可以是数组或类似数组的对象。若目标函数无法调用，会抛出 `TypeError`
- `Reflect.defineProperty(target, propertyKey, attributes)` 用于为目标对象定义属性。如果 `target` 不是对象，会抛出错误。
- `Reflect.getOwnPropertyDescriptor(target, propertyKey)` 用于得到 `target` 对象的 `propertyKey` 属性的描述对象。在 `target` 不是对象时，会抛出错误表示参数非法，不会将非对象转换为对象
- `Reflect.isExtensible(target)` 用于判断 `target` 对象是否可扩展。返回值为布尔值。如果 `target` 参数不是对象，会抛出错误
- `Reflect.preventExtensions(target)` 用于让 `target` 对象变为不可扩展。如果 `target` 参数不是对象，会抛出错误
- `Reflect.ownKeys(target)` 用于返回 `target` 对象的所有属性，等同于 `Object.getOwnPropertyNames` 与 `Object.getOwnPropertySymbols` 之和
