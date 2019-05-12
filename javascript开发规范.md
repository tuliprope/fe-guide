## **Javascript** 开发规范

### 不要使用没有终点的循环

**❌**
```
for(let i = 0; i < 10; i--){}
for(let i = 10; i > 0; i++){}
```
**✔**
```
for(let i = 0; i < 10; i++){}
```

### 不要在if,while,for语句的条件中出现常量表达式,不要在?:运算符的条件中出现常量表达式
**❌**
```
if(false){}
// typeof 任何值都不为false
while(typeof x){}
let c = 0 ? a : b
```
**✔**
```
if(x === false){}
while(typeof x === 'undefined'){}
let c = x === 0 ? a : b
```
### getter应该有返回值
**❌**
```
let p = {
    get name(){}
}
class p {
    get age(){}
}
Object.defineProperty(p, 'name', {
    get: function(){}
})
```
**✔**
```
let p = {
    get name(){
        return 'jack'
    }
}
class p {
    get age(){
        return 6
    }
}
Object.defineProperty(p, 'name', {
    get: function(){
        return 'Tom'
    }
})
```
### 不要在循环体中使用await
> 在迭代器的每个元素上执行运算是个常见的任务。然而，每次运算都执行 await，意味着该程序并没有充分利用 async/await 并行的好处。
通常，代码应该重构为立即创建所有 promise，然后通过 Promise.all() 访问结果。否则，每个后续的操作将不会执行，直到前一个操作执行完毕。

示例
```
async function getAllData() {
    let result = []
    for (let i = 0 i < 5 i++) {
        //进入下一次for循环前需等待当前请求完成
        let res = await axios.get('test' + i)
        result.push(res.data)
    }
    return result
}

async function getAllData2() {
    let result = []
    for (let i = 0 i < 5 i++) {
        //进入下一次for循环前不需等待当前请求完成
        result.push(axios.get('/test' + i).then(res => res.data))
    }
    return await Promise.all(result)
}

async function test() {
    let start = Date.now()
    await getAllData()
    console.log(`在循环中使用await耗时:${Date.now() - start}ms`)
    start = Date.now()
    await getAllData2()
    console.log(`不在循环中使用await耗时:${Date.now() - start}ms`)
}

test()

/*  console输出:
    在循环中使用await耗时:1016ms
    不在循环中使用await耗时:204ms */
```
### 禁止在对象字面量中出现重复key
> 如果对象字面量中出现多个属性有同样的键可能会到导致意想不到的情况出现。

**❌**
```
let foo = {
    x: 1,
    y: 2,
    x: 1
}
```
**✔**
```
let foo = {
    x: 1,
    y: 2,
    z: 1
}
```
### 禁止在 switch 语句中的 case 子句中出现重复的测试表达式。
**❌**
```
let x = 1

switch(x){
    case 1:
        break
    case 2:
        break
    case 1:   // 重复的case
        break
    default:
        break
}
```
**✔**
```
let x = 1

switch(x){
    case 1:
        break
    case 2:
        break
    case 1:   // 重复的case
        break
    default:
        break
}
```
### 在正则表达式中空字符集[]不能匹配任何字符，它们可能是打字错误。
**❌**
```
/^abc[]/.test('abcdefg') // 结果为 false
'abcdefg'.match(/^abc[]/) // 结果为 null
```
**✔**
```
/^abc[a-z]/.test('abcdefg') // 结果为 true
'abcdefg'.match(/^abc[a-z]/) // 结果为 ['abcd']
```
### 禁止不必要的布尔类型转换
> 在上下文中比如 if 语句的测试表达式的结果已经被强制转化成了一个布尔值，再通过双重否定（!!）或 Boolean 转化是不必要的。例如，这些 if 语句是等价的：

```
if (!!x) {
    // ...
}

if (Boolean(x)) {
    // ...
}

if (x) {
    // ...
}
```
**❌**
```
let foo = !!!bar

let foo = !!bar ? baz : bat

let foo = Boolean(!!bar)

let foo = new Boolean(!!bar)

if (!!foo) {
    // ...
}

if (Boolean(foo)) {
    // ...
}

while (!!foo) {
    // ...
}

do {
    // ...
} while (Boolean(foo))

for ( !!foo ) {
    // ...
}
```
**✔**
```
let foo = !!bar
let foo = Boolean(bar)

function foo() {
    return !!bar
}

let foo = bar ? !!baz : !!bat
```
### 禁止对 function 声明重新赋值 
> JavaScript 函数有两种形式：函数声明 function foo() { ... } 或者函数表达式 let foo = function() { ... } 。虽然 JavaScript 解释器可以容忍对函数声明进行覆盖或重新赋值，但通常这是个错误或会导致问题出现。

**❌**
```
function foo() {}
foo = bar

function foo() {
    foo = bar
}

foo = bar
function foo() {}
```
**✔**
```
let foo = function () {}
foo = bar

function foo(foo) { // `foo`是局部变量
    foo = bar
}

function foo() {
    let foo = bar  // `foo`是局部变量
}
```
### 禁止在 RegExp 构造函数中出现无效的正则表达式 
>在正则表达式字面量中无效的模式在代码解析时会引起 SyntaxError，但是 RegExp 的构造函数中无效的字符串只在代码执行时才会抛出 SyntaxError。

**❌**
```
RegExp('[')

RegExp('.', 'z')

new RegExp('\\')
```
**✔**
```
RegExp('\\[')
RegExp('.','g')
new RegExp('\\\\')
```
### 禁止直接使用 Object.prototypes 的内置属性 
> ECMAScript 5.1 新增了 Object.create，可以通过它创建带有指定的 [[Prototype]] 的对象。Object.create(null) 是一种常见的模式，用来创建键值对对象。当创建的对象有从 Object.prototype 继承来的属性时，可能会导致错误出现。应防止在一个对象中直接调用 Object.prototype 的方法。

**❌**
```
let hasBarProperty = foo.hasOwnProperty('bar')

let isPrototypeOfBar = foo.isPrototypeOf(bar)

let barIsEnumerable = foo.propertyIsEnumerable('bar')
```
**✔**
```
let hasBarProperty = Object.prototype.hasOwnProperty.call(foo, 'bar')

let isPrototypeOfBar = Object.prototype.isPrototypeOf.call(foo, bar)

let barIsEnumerable = {}.propertyIsEnumerable.call(foo, 'bar')
```
### 禁止正则表达式字面量中出现多个空格 
> 正则表达式可以很复杂和难以理解，这就是为什么要保持它们尽可能的简单，以避免出现错误。你在使用正则表达式时最容易出错的是使用了多个空格，例如：

```
let re = /foo   bar/
```

在这个正则表达式中，很难断定想要匹配多少个空格。最好是只使用一个空格，然后指定需要多少个，例如：

``` 
let re = /foo {3}bar/ 
```
现在非常清楚地知道需要匹配 3 个空格。
### 禁用稀疏数组
> 稀疏数组包括很多空位置，经常是由于在数组字面量中使用多个逗号造成的，例如：

```
let items = [,,]
```
在这个例子中，item 数组的 length 为 2，实际上，items[0] 或 items[1]并没有值。数组中只有逗号是有效的，再加上 length 被设置，没有实际的值被设置，这些情况让很多开发者对稀疏数组感到困惑。考虑下面的情况：
```
let colors = [ 'red',, 'blue' ]
```
在这个例子中，colors 数值的 length 是 3。但是否是开发者想让数组中间出现一个空元素？或者只是一个书写错误？

稀疏数组的定义方式造成了很大的困惑，建议避免使用它们，除非你确定它们在你的代码中很有用。
### 禁止在常规字符串中出现模板字面量占位符语法 
> ECMAScript 6 允许程序员使用模板字面量创建包含变量或表达式的字符串，在两个反引号之间书写表达式比如 `${variable}`，而不是使用字符串拼接。在使用模板字面量过程中很容易写错引号，写错成 '${variable}' 而不是在字符串中包含注入的表达式的值。

**❌**
```
'Hello ${name}!'
'Hello ${name}!'
'Time: ${12 * 60 * 60 * 1000}'
```
**✔**
```
`Hello ${name}!`
`Time: ${12 * 60 * 60 * 1000}`
`Hello ${name}`
```
### 禁止在 return、throw、continue 和 break 语句后出现不可达代码
> 因为 return、throw、continue 和 break 语句无条件地退出代码块，其之后的任何语句都不会被执行。不可达语句通常是个错误。

**❌**
```
function foo() {
    return true
    console.log('done')
}

function bar() {
    throw new Error('Oops!')
    console.log('done')
}

while(value) {
    break
    console.log('done')
}

throw new Error('Oops!')
console.log('done')

function baz() {
    if (Math.random() < 0.5) {
        return
    } else {
        throw new Error()
    }
    console.log('done')
}
```
**✔**
```
function foo() {
    return bar()
    function bar() {
        return 1
    }
}
```
### 禁止对关系运算符的左操作数使用否定操作符 
> 开发人员可能会把 -(a + b) 写错成 -a + b 来表示一个负数，也可能会把 !(key in   object) 错写成 !key in object 来测试一个键是否在对象中。类似的情况也有 !obj    instanceof Ctor。
关系运算符有：
> + in 运算符.
> + instanceof 运算符.


**❌**
```
if (!key in object) {
    //由于运算符优先级和类型转换以上代码等价于: (key ? 'false' : 'true') in object
}

if (!obj instanceof Ctor) {
    //因为!obj运算结果为boolean而boolean不是对象因此运算结果始终为false
}
```
**✔**
```
if (!(key in object)) {
    
}

if (!(obj instanceof Ctor)) {
    
}
```
### 要求调用 isNaN()检查 NaN
> 在 JavaScript 中，NaN 是 Number 类型的一个特殊值。它被用来表示非数值，这里的数值是指在 IEEE 浮点数算术标准中定义的双精度64位格式的值。
> 
> 因为在 JavaScript 中 NaN 独特之处在于它不等于任何值，包括它本身，与 NaN 进行比较的结果是令人困惑：
> 
> `NaN !== NaN` 或者 `NaN != NaN` 运算结果为`true`
> 
> 因此，使用 Number.isNaN() 或 全局的 isNaN() 函数来测试一个值是否是 NaN。

**❌**
```
if (foo == NaN) {
    // ...
}

if (foo != NaN) {
    // ...
}
```
**✔**
```
if (isNaN(foo)) {
    // ...
}

if (!isNaN(foo)) {
    // ...
}
```
### 强制数组方法的回调函数中有 return 语句 
> Array有一些方法用来过滤、映射和折叠。如果你忘记了在它们的回调函数中写return语句，这种情况可能是个错误。
> ```
> // 例如将 ['a', 'b', 'c'] 转换成 {a: 1, b: 2, c: 3} 如果忘记return会报错
> let indexMap = myArray.reduce(function(memo, item, index) {
>   memo[item] = index
> }, {}) // Error: cannot set property 'b' of undefined
> ```
> 
> 因此,请在以下数组方法中包含return语句
> + Array.from
> + Array.prototype.every
> + Array.prototype.filter
> + Array.prototype.find
> + Array.prototype.findIndex
> + Array.prototype.map
> + Array.prototype.reduce
> + Array.prototype.reduceRight
> + Array.prototype.some
> + Array.prototype.sort

**❌**
```
let indexMap = myArray.reduce(function(memo, item, index) {
    memo[item] = index
}, {})

let foo = Array.from(nodes, function(node) {
    if (node.tagName === 'DIV') {
        return true
    }
})

let bar = foo.filter(function(x) {
    if (x) {
        return true
    } else {
        return
    }
})
```
**✔**
```
let indexMap = myArray.reduce(function(memo, item, index) {
    memo[item] = index
    return memo
}, {})

let foo = Array.from(nodes, function(node) {
    if (node.tagName === 'DIV') {
        return true
    }
    return false
})

let bar = foo.map(node => node.getAttribute('id'))
```
### 要求 Switch 语句中有 Default 分支
> 考虑到开发人员可能会忘记定义默认分支而导致程序发生错误，所以明确规定定义默认分支是很好的选择。

**❌**
```
switch (a) {
    case 1:
        /* code */
        break
}
```
**✔**
```
switch (a) {
    case 1:
        /* code */
        break

    default:
        /* code */
        break
}

switch (a) {
    case 1:
        /* code */
        break

    // no default
}

switch (a) {
    case 1:
        /* code */
        break

    // No Default
}
```
### 要求使用 === 和 !==
> 使用类型安全的 === 和 !== 操作符代替 == 和 != 操作符是一个很好的实践。
> 
> 这样做的原因是，== 和 != 遵循 [Abstract Equality Comparison Algorithm](https://www.ecma-international.org/ecma-262/5.1/#sec-11.9.3) 作强制转型. 例如，以下语句被认为是 true。
> ```
>     [] == false
>     [] == ![]
>     3 == '03'
> ```
> 如果它们中的任何一个出现在一个看上去无害的语句中，比如 a == b ，那么实际的问题是很难 被发现的。

**❌**
```
if (x == 42) { }

if ('' == text) { }

if (obj.getStuff() != undefined) { }
```
**✔**
```
if (x === 42) { }

if ('' === text) { }
```
### 禁止在 case 或 default 子句中出现词法声明 
> 禁止词法声明 (let、const、function 和 class) 出现在 case或default 子句中。原因是，词法声明在整个 switch 语句块中是可见的，但是它只有在运行到它定义的 case 语句时，才会进行初始化操作。
> 
> 为了保证词法声明语句只在当前 case 语句中有效，将声明子句包裹在块中。

**❌**
```
switch (foo) {
    case 1:
        let x = 1
        break
    case 2:
        const y = 2
        break
    case 3:
        function f() {}
        break
    default:
        class C {}
}
```
**✔**
```
const a = 0

switch (foo) {
    case 1: {
        let x = 1
        break
    }
    case 2: {
        const y = 2
        break
    }
    case 3: {
        function f() {}
        break
    }
    default: {
        class C {}
    }
}
```
### 小数点前面和后面应该有一个数字
> 在 JavaScript 中，浮点值会包含一个小数点，没有要求小数点之前或之后必须有一个数字。例如，以下例子都是有效的 JavaScript 数字：
> ```
> let num1 = .5
> let num2 = 2.
> let num3 = -.7
> ```
> 虽然不是一个语法错误，这种格式的数字使真正的小数和点操作符变的难以区分。由于这个原因，有些人建议应该总是在小数点前面和后面有一个数字，以明确表明是要创建一个小数。

**❌**
```
let num1 = .5
let num2 = 2.
let num3 = -.7
```
**✔**
```
let num1 = 0.5
let num2 = 2.0
let num3 = -0.7
```
### 禁止使用较短的符号实现类型转换
> 在 JavaScript 中，有许多不同的方式进行类型转换。其中有些可能难于阅读和理解。
> 
> 例如：
> ```
> let b = !!foo
> let b = ~foo.indexOf('.')
> let n = +foo
> let n = 1 * foo
> let s = '' + foo
> foo += ``
> ```
> 可以使用下面的代码替换:
> ```
> let b = Boolean(foo)
> let b = foo.indexOf('.') !== -1
> let n = Number(foo)
> let n = Number(foo)
> let s = String(foo)
> foo = String(foo)
> ```

### 禁止使用空解构模式
> 当使用解构赋值时，可能创建了一个不起作用的模式。把空的花括号放在嵌入的对象的解构模式右边时，就会产生这种情况，例如：
> ```
> // 以下代码不会创建任何变量
> let {a: {}} = foo
> ```
> 在以上代码中，没有创建新的变量，因为 a 只是一个辅助位置，而 {} 将包含创建的变量，例如：
> ```
> // 创建变量 b
> let {a: { b }} = foo
> ```
> 在许多情况下，作者本来打算使用一个默认值，却错写成空对象，例如：
> ```
> // 创建一个默认值为{}的变量a
> let {a = {}} = foo
> ```
> 这两种模式直接的区别是微妙的，因为空模式看起来像是一个对象字面量。

**❌**
```
let {} = foo
let [] = foo
let {a: {}} = foo
let {a: []} = foo
function foo({}) {}
function foo([]) {}
function foo({a: {}}) {}
function foo({a: []}) {}
```
**✔**
```
let {a = {}} = foo
let {a = []} = foo
function foo({a = {}}) {}
function foo({a = []}) {}
```
### 禁用不必要的 return await
> 在 async function， return await 是没有用的 。因为 async function 的返回值总是包裹在 Promise.resolve，在 Promise resolve 或 reject 之前，return await 实际上不会做任何事情。这种模式几乎可以肯定是由于程序员不知道 async function 语法的返回值造成的。

**❌**
```
async function foo() {
  return await bar()
}
```
**✔**
```
async function foo() {
  return bar()
}

async function foo() {
  await bar()
  return
}

async function foo() {
  const x = await bar()
  return x
}

async function foo() {
  try {
    // await 是必须的，可以捕获从 bar() 抛出的错误
    return await bar()
  } catch (error) {}
}
```
### 禁止未使用过的变量
> 已声明的变量在代码里未被使用过，就像是由于不完整的重构而导致的遗漏错误。这样的变量增加了代码量，并且混淆读者。

### 派生类中的构造函数必须调用 super(),非派生类的构造函数不能调用 super()

**❌**
```
class A extends B {
    constructor() { }  // ReferenceError.
}
```
**✔**
```
class A {
    constructor() { }
}

class A extends B {
    constructor() {
        super()
    }
}
```
### 禁止重复导入
> 为每个模块使用单一的 import 语句会是代码更加清晰，因为你会看到从该模块导入的所有内容都在同一行。
> 
> 在下面的例子中，行 1 和 行 3 的模块导入是重复的。二者合并会使导入列表更加简洁。
> ```
> import { merge } from 'module'
> import something from 'another-module'
> import { find } from 'module'
> ```

**❌**
```
mport { merge } from 'module'
import something from 'another-module'
import { find } from 'module'
```
**✔**
```
import { merge, find } from 'module'
import something from 'another-module'
```
### 要求使用 let 或 const 而不是 var
> ECMAScript 6 允许程序员使用 let 和 const 关键字在块级作用域而非函数作用域下声明变量。块级作用域在很多其他编程语言中很普遍，能帮助程序员避免错误，例如：
> ```
> var count = people.length
> var enoughFood = count > sandwiches.length
> 
> if (enoughFood) {
>     var count = sandwiches.length // 无意中覆盖了变量count
>     console.log('We have ' + count + ' sandwiches for everyone. Plenty for all!')
> }
> 
> // count的值不再为people.length
> console.log('We have ' + count + ' people and ' + sandwiches.length + 'sandwiches!')
> ```

**❌**
```
var x = 'y'
var CONFIG = {}
```
**✔**
```
let x = 'y'
const CONFIG = {}
```
### 建议使用const
> 如果一个变量不会被重新赋值，最好使用const进行声明。
const 声明告诉读者，“这个变量从不会被重新赋值”，从而减少认知负荷，提高可维护性。

### 建议使用剩余参数代替 arguments
> ES2015 里有剩余参数。我们可以利用这个特性代替变参函数的 arguments 变量。
> arguments 没有 Array.prototype 方法，所以有点不方便。

**❌**
```
function foo() {
    console.log(arguments)
}

function foo(action) {
    let args = Array.prototype.slice.call(arguments, 1)
    action.apply(null, args)
}

function foo(action) {
    let args = [].slice.call(arguments, 1)
    action.apply(null, args)
}
```
**✔**
```
function foo(...args) {
    console.log(args)
}

function foo(action, ...args) {
    action(...args)
}
```
### 建议使用扩展运算符而非.apply()
> 在 ES2015 之前，必须使用 Function.prototype.apply() 调用可变参数函数。
>``` 
> let args = [1, 2, 3, 4]
> Math.max.apply(Math, args)
> ```
> 在 ES2015 中，可以使用扩展运算符调用可变参数函数。
> ```
> let args = [1, 2, 3, 4]
> Math.max(...args)
> ```
