# ES6时代的总结

## 七种数据类型
Undefined、Null、Boolean、String、Number、Object、Symbol

## 字符串的正则方法
- String.prototype.match 调用 RegExp.prototype[Symbol.match]
- String.prototype.replace 调用 RegExp.prototype[Symbol.replace]
- String.prototype.search 调用 RegExp.prototype[Symbol.search]
- String.prototype.split 调用 RegExp.prototype[Symbol.split]

## 整数范围
JavaScript能够准确表示的整数范围在`-2^53`到`2^53`之间（不含两个端点），超过这个范围，无法精确表示这个值。

## 数值存储
JavaScript的整数使用32位二进制形式表示



## 数组中的`NaN`
- `Array.prototype.indexOf()`不支持`NaN`
- `Array.prototype.findIndex()`可以借助`Object.is()`支持`NaN`
- `Array.prototype.includes()`支持`NaN`

```js
[NaN].indexOf(NaN)
// -1

[NaN].findIndex(y => Object.is(NaN, y))
// 0
```
上面代码中，`indexOf`方法无法识别数组的`NaN`成员，但是`findIndex`方法可以借助`Object.is`方法做到。


## 数组中的空位
ES5的API对空位的处理，已经很不一致了，大多数情况下会忽略空位。
- `forEach()`, `filter()`, `every()`, `some()`都会跳过空位。
- `map()`会跳过空位，但会保留这个值
- `join()`和`toString()`会将空位视为`undefined`，而`undefined`和`null`会被处理成空字符串。

ES6的API则是明确将空位转为`undefined`。

## 包装对象
只有字符串的包装对象会产生可枚举属性
```js
Object(true) // {[[PrimitiveValue]]: true}
Object(10)  //  {[[PrimitiveValue]]: 10}
Object('abc') // {0: "a", 1: "b", 2: "c", length: 3, [[PrimitiveValue]]: "abc"}
```


## 克隆对象
拷贝到一个空对象，就得到了原始对象的克隆
```js
function clone(origin) {
  return Object.assign({}, origin);
}

```

不过，采用这种方法克隆，只能克隆原始对象自身的值，不能克隆它继承的值。如果想要保持继承链，可以采用下面的代码。

```js
function clone(origin) {
  let originProto = Object.getPrototypeOf(origin);
  return Object.assign(Object.create(originProto), origin);
}
```

## 属性的可枚举性
`Object.getOwnPropertyDescriptor(obj:Object, property:String)`方法可以获取某个属性的描述对象。

`obj.propertyIsEnumerable(prop)` 可以获取指定的属性名是否是当前对象的可枚举的**自身属性**

### 不可枚举的作用
ES5有三个操作会忽略`enumerable`为`false`的属性。
- `for...in`循环：只遍历对象自身的和继承的可枚举的属性
- `Object.keys()`：返回对象自身的所有可枚举的属性的键名
- `JSON.stringify()`：只串行化对象自身的可枚举的属性

ES6新增了一个操作会忽略`enumerable`为`false`的属性
- `Object.assign()`

### 所有Class的原型方法都是不可枚举的

## 对象属性的遍历方法
ES6共有5种方法遍历对象的属性
1. `for...in`
循环遍历对象**自身和继承**的**可枚举**属性，不含`Symbol`属性

1. `Object.keys(obj)`
返回数组，值是属性的键名，包含**非继承**的**可枚举**的属性，不含`Symbol`属性

1. `Object.getOwnPropertyNames(obj)`
返回数组，值是属性的键名，包含**非继承**的属性，**包含可枚举和不可枚举属性**，不含`Symbol`属性

1. `Object.getOwnPropertySymbols(obj)`
返回数组，值是属性的键名，只包含**非继承**的Symbol属性，没有常规属性

1. `Reflect.ownKeys(obj)`
返回数组，值是属性的键名，包含**非继承**的所有属性，包含`Symbol`属性，也不管是否可枚举

|                              | 属性名类型    | 自身 | 继承 | 可枚举 | 不可枚举 |
| ---------------------------- | ------------- | ---- | ---- | ------ | -------- |
| for...in                     | String        |  Y   |  Y   |   Y    |    N     |
| Object.keys/values/entries   | String        |  Y   |  N   |   Y    |    N     |
| Object.getOwnPropertyNames   | String        |  Y   |  N   |   Y    |    Y     |
| Object.getOwnPropertySymbols | Symbol        |  Y   |  N   |   Y    |    Y     |
| Reflect.ownKeys              | String&Symbol |  Y   |  N   |   Y    |    Y     |


以上的5种方法遍历对象的属性，都遵守同样的属性遍历的次序规则：
- 首先遍历所有属性名为数值的属性，按照数字排序。
- 其次遍历所有属性名为字符串的属性，按照生成时间排序。
- 最后遍历所有属性名为Symbol值的属性，按照生成时间排序。


## 识别32位Unicode字符的方法
### 1. 利用Spread扩展运算符
```
'x\uD83D\uDE80y'.length // 4

function length(str) {
  return [...str].length;
}

length('x\uD83D\uDE80y') // 3
```
### 2. for...of

## 数组去重
方法一：
```js
[...new Set(array)]
```
方法二
```js
function dedupe(array) {
  return Array.from(new Set(array));
}

dedupe([1, 1, 2, 3]) // [1, 2, 3]
```


## 部署了Iterator接口的数据结构 [Symbol.iterator]
1. Set
2. Map
3. Array
4. 某些Array-Like  如`arguments` `DOM NodeList`
5. String包装对象
6. Generator

### Iterator接口的作用
1. 解构赋值
对`array`和`Set`解构赋值的时候，会调用`Symbol.iterator`

1. 扩展运算符
`...`默认调用iterator接口，可以将任何实现了iterator接口的数据结构转为数组形式

1. `yield*`
默认调用iterator接口

1. `for...of`
1. `Array.from()`
1. `Map()`,`Set()`,`WeakMap()`,`WeakSet()`
在构造相应实例的时候，都可以接受一个实现了iterator接口的数据结构

1. `Promise.all()`
1. `Promise.race()`


## ECMAScript的原生构造函数
1. Boolean()
1. Number()
1. String()
1. Array()
1. Date()
1. Function()
1. RegExp()
1. Error()
1. Object()

----

## 严格模式
ES5引入了严格模式, 需要在头部用`"use strict"`显示启用, 

ES6中, 类和模块自动启用严格模式, 所以基本等于ES6升级到了严格模式


严格模式有以下限制:
- 变量必须声明后再使用
- 函数的参数不能有同名属性，否则报错
- 不能使用`with`语句
- 不能对只读属性赋值，否则报错
- 不能使用前缀`0`表示八进制数，否则报错
- 不能删除不可删除的属性，否则报错
- 不能删除变量`delete prop`，会报错，只能删除属性`delete global[prop]`
- `eval`不会在它的外层作用域引入变量
- `eval`和`arguments`不能被重新赋值
- `arguments`不会自动反映函数参数的变化
- 不能使用`arguments.callee`
- 不能使用`arguments.caller`
- 禁止`this`指向全局对象, 顶层`this`是`undefined`
- 不能使用`fn.caller`和`fn.arguments`获取函数调用的堆栈
- 增加了保留字（比如`protected`、`static`和`interface`）
