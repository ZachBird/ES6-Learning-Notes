# ES6-Learning-Notes
ECMAScript 6 Primer reading notes

## 暂时先记在这里 ##

## let和const命令 ##

### let ###

1. 声明的变量只在所处代码块中有效
2. 不存在变量提升。方方说实际上是存在的：https://zhuanlan.zhihu.com/p/28140450 不过不影响理解，声明前调用会报错。
3. 暂时性死区（Temporal Dead Zone）
只要块级作用域内**存在**let命令，他所声明的变量就“绑定”（binding）这个区域，不再受外部的影响。

```javascript
if(true){
  // TDZ开始
  tmp = 'abc' // ReferenceError
  console.log(tmp) // ReferenceError
  
  let tmp //TDZ结束
  
  console.log(tmp) //undefined
  tmp = 123
  console.log(tmp) //123
}
```

上面的代码段能很好的理解暂时性死区的“区”的范围：在let命令声明变量tmp之前，都属于变量tmp的“死区”。
然而有些“死区”比较隐蔽，不太容易发现：

```javascript
function bar(x = y, y = 2) {
  return [x,y]
}
bar() //报错
```
报错的原因就是x = y,此时变量y并没有声明，属于“死区”。如果y的默认值是x，就不会报错，因为此时x已经声明：

```javascript
function bar(x = 2, y = x) {
  return [x,y]
}
bar() //[2, 2]
```
ES6规定暂时性死区和不存在变量提升，就是为了避免ES5中采用变量提升导致意外的错误。

总之，暂时性死区的本质是，只要进入当前作用域，所要使用的变量就已经存在，但不可获取，只有等到声明变量的那行代码出现，才可以获取和使用该变量。

4. 不允许重复声明

let不允许在**相同作用域**内重复声明同一个变量。

```javascript
// Error
function () {
  var a = 10
  let a = 1
}

// Error
function () {
  let a = 10 
  let a = 100
}
```

因此不能在函数内部重新声明参数。

```javascript
function (arg) {
  let arg // Error
}

function (arg) {
  { 
    let arg // no Error
  }
}
```

还能帮助你理解所谓的“相同作用域”。

### 块级作用域 ###

1. 为什么要有块级作用域

ES5只有全局和函数作用域，没有块级作用域，这就带来很多不合理的场景：

**Sence 1** 变量覆盖
内层变量覆盖了外层变量，导致意外的结果。

```javascript
var tmp = new Date()
function f(){
	console.log(tmp)
	if (true){
		var tmp = "hello world"
	}
}
f() //undefined
```
原因是变量提升导致内层的tmp变量覆盖了全局的tmp变量。

**Sence 2** 用来计数的循环变量泄露

```javascript
var arr = [1,2,3,4,5]

for (var i = 0; i < arr.length; i++){
    console.log(arr[i])
}

console.log(i) // 5
// 用let声明i，这里会报错，i未定义
```
上面代码中，i只用来控制循环，用var来定义i，循环结束后，它没有消失，而是泄露成了全局变量，污染了变量池，自然会带来这样那样的错误。

2. ES6的块级作用域

let实际上为JavaScript新增了块级作用域。

```javascript
function f1() {
  let n =5
  if (true) {
    let n = 10
  }
  console.log(n) // 5
}
```
上面的代码块的两个块级作用域，都声明了变量n，运行后输出5。这表示外层作用域不受内层作用域的影响。如果使用var定义变量n，最后输出的值就是10。

ES6允许块级作用域任意嵌套。

外层作用域无法读取内层作用域的变量。

```javascript
{{{{
    {let insane = "already madness"}
    console.log(insane) // Error
}}}}
```

内层作用域可以定义外层作用域的同名变量。

```javascript
{{{{
    let insane = "already madness"
    {let insane = "already madness"}
}}}}
```
块级作用域的出现，实际上使得获得广泛应用的立即执行函数表达式（IIFE）不再必要了。

```javascript
// IIFE (Immediately Invoked Function Expression)
(function () {
    var tmp = ...
    ...
}())

// 块级作用域
{
  let tmp = ...
  ...
}

```
先写到这里，后面的http://es6.ruanyifeng.com/#docs/let#块级作用域与函数声明 看这个再补充

核心内容就是：
ES6函数作用域内声明函数类似let，对作用域外无影响。
但是在浏览器中允许有自己的行为方式，所以会产生环境差异，理论上可行的代码，在浏览器环境下有可能报错，所以尽量使用函数表达式而不是函数声明语句。
ES6 的块级作用域允许声明函数的规则，**只在使用大括号的情况下成立**，如果没有使用大括号，就会报错。

```javascript
```
