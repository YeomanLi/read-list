|                    title                     |    date    |     tags     |
| :------------------------------------------: | :--------: | :----------: |
| Secrets of the JavaScript Ninja - 打卡第二天 | 2019-07-07 | 作用域、闭包 |



# Secrets of the JavaScript Ninja



# Chapter Five

这一章主要讲的是闭包（_closure_）和作用域（_scopes_）。

这张感觉挺难的。。好多都没看明白，各种环境啥的，多回看吧。



# 理解闭包

闭包允许函数访问并操作函数外部的变量。只要变量或函数存在于声明函数时的作用域内，闭包即可使函数能够访问这些变量或函数。

首先看下面的示例：

```javascript
var outerVal = 'outerVal'
var later   //  声明一个空变量，稍后在后面的代码中使用

function outerFun() {
    var innerVal = 'innerVal'
    function innerFun () {
        console.log(`I can see the ${ outerVal }`)
        console.log(`I can see the ${ innerVal }`)
    }
    later = innerFun
}

outerFun()
later()

// I can see the outerVal
// I can see the innerVal
```

在上面的代码中，我们将内部函数 `innerFun` 的引用存储在变量 `later` 上，因为 `later` 在全局作用域内，所以我们可以对它进行调用

接着，我们调用了 `outerFun` 这个函数，创建内部函数 `innerFun` ，并将内部函数赋值给变量 `later` 。

当在外部函数中声明内部函数时，不仅仅定义了函数声明，而且还创建了一个闭包。**该闭包不仅包含了函数的声明，还包含了在函数声明时该作用域中的所有变量。**当最终执行内部函数时，尽管声明时的作用域已经消失了，但是通过闭包，仍然能够访问到原始作用域。

闭包不能过度使用，使用闭包时，所有的信息都会存储在内存中，直到 JavaScript 引擎确保这些信息不再使用（可以安全地进行垃圾回收）或者页面卸载时，才会清理这些信息。



# 使用闭包

## 使用闭包封装私有变量

为啥要有私有变量：当通过其他代码访问这些变量时，我们不希望对象的实现细节对用户造成过度负荷。

_原生 JavaScript 不支持私有变量_。

但是可以使用闭包来实现很接近的、可接受的私有变量：

```javascript
function Baby() {
    var age = 0
    this.getAge = function () {
        return age
    }

    this.grow = function () {
        ++age
    }
}

const baby = new Baby()
baby.grow()
console.log(baby.getAge())
console.log(baby.age)

// 1
// undefined
```

虽然可以通过闭包内部方法获取私有变量的值，但是不能直接访问私有变量。有效地阻止了读私有变量不可控的修改，这与真实的面向对象语言中的私有变量一样。



当我们通过构造函数创建一个新的实例时，新的实例则具有自己私有的 `age` 变量。

```javascript
var anotherBaby = new Baby()
console.log(anotherBaby.getAge())	// 0
```



## 在回调函数中使用闭包

![interval 的回调函数中使用闭包](https://s2.ax1x.com/2019/07/07/ZDKfKI.png)

可以看到，每个动画在处理器的闭包内获得私有变量，并且各个实例之间互相独立，不能访问其他闭包中的变量。



# 通过执行上下文来跟踪代码

两种执行上下文：

1. 全局执行上下文：只有一个，当 JavaScript 程序开始执行时就已经创建了全局上下文。
2. 函数执行上下文：每次调用函数时，都会创建一个新的。

一旦发生函数调用,当前的执行上下文必须停止执行，并创建新的函数执行上下文来执行函数。当函数执行完成后，将函数执行上下文销毁,，并重新回到发生调用时的执行上下文中。所以需
要跟踪执行上下文——正在执行的上下文以及正在等待的上下文。最简单的跟踪方法是使用执行上下文栈(或称为调用栈)。

![执行上下文栈](https://s2.ax1x.com/2019/07/07/ZDM4SJ.png)



# 关键字 `var`

>  当使用关键字 `var` 时，该变量是在距离最近的函数内部或是在全局此法环境中定义的。

```javascript
var globalNinja = 'Yoshi'
function reportActivity () {
    var functionActivity = 'jumping'
    for (var i = 0; i < 3; ++i) {
        var forMessage = globalNinja + " " + functionActivity
        console.log(`current loop counter: ${ i }`)
    }
    if (3 == i && 'Yoshi jumping' === forMessage)
        console.log('Loop variables accessible outside of the loop!')
}
reportActivity()

/*
    current loop counter: 0
    current loop counter: 1
    current loop counter: 2
    Loop variables accessible outside of the loop!
*/
```

![两次循环结束后词法环境的状态](https://s2.ax1x.com/2019/07/07/ZDGPJK.png)



# 注册标识符的过程

>  JavaScript 代码的执行事实上是分两个阶段进行的。

一旦创建了新的词法环境，就会执行第一阶段。在第一阶段，没有执行代码，但是 JavaScript 引擎会访问并注册在当前词法环境中所声明的变量和函数。第一阶段结束后开始执行第二阶段，具体如何执行取决于变量的类型以及环境类型（全局环境、函数环境或块级作用域）。

具体过程如下：

1. 如果是创建一个函数环境，那么创建形参及函数参数的默认值。如果是非函数环境，将跳过此步骤。
2. 如果是创建全局或函数环境,就扫描当前代码进行函数声明（不会扫描其他函数的函数体），但是**不会扫描函数表达式或箭头函数**。对于所找到的函数声明，将创建函数，并绑定到当前环境与函数名相同的标识符上。若该标识符已经存在,那么该标识符的值将被重写。如果是块级作用域，将跳过此步骤。
3. 扫描当前代码进行变量声明。在函数或全局环境中，找到所有当前函数以及其他函数之外通过 `var` 声明的变量，并找到所有在其他函数或代码块之外通过 `let` 或 `const` 定义的变量。在块级环境中，仅查找当前块中通过 `let` 或 `const` 定义的变量。对于所查找到的变量,若该标识符不存在，进行注册并将其初始化为 `undefined` 。若该标识符已经存在,将保留其值。



![具体过程](https://s2.ax1x.com/2019/07/07/ZDYBqJ.png)

## 重载标识符

```javascript
console.log(typeof fun)
var fun = 3
console.log(typeof fun)
function fun () {}
console.log(typeof fun)

// function
// number
// number
```

JavaScript 的这种行为是由标识符注册的结果直接导致的。在处理过程的第2步中，通过函数声明进行定义的函数在代码执行之前对函数进行创建，并赋值给对应的标识符；在第3步，处理变量的声明，那些在当前环境中未声明的变量，将被赋值为 `undefined` 。在示例中，在第2步——注册函数声明时，由于标识符fun已经存在，并未被赋值为 `undefined` 。这就是第1个打印为 `function` 的原因。之后，执行赋值语句 `var fun = 3` ，将数字3赋值给标识符 `fun` 。执行完这个赋值语句之后，`fun` 就不再指向函数了，而是指向数字3。

在程序的实际执行过程中，跳过了函数声明部分，所以函数声明不会影响标识符 `fun` 的值，所以最后打印结果为 `number` 。



## 变量提升

书中稍微带了一下，还是不太明白，暂时搬运过来吧。。

![变量提升](https://s2.ax1x.com/2019/07/08/ZDtvp6.png)



# 总结

- 闭包的主要用途
  - 通过构造函数内的变量以及构造方法来模拟对象的私有属性
  - 处理回调函数，简化代码
- JavaScript 引擎通过执行上下文栈（调用栈）跟踪函数的执行。每次调用函数时，都会创建新的函数执行上下文，并推入调用栈顶端。当函数执行完后，对应的执行上下文将从调用栈中推出。
- JavaScript 引擎通过词法环境跟踪标识符（俗称作用域）。
- 关键字 `var` 定义距离最近的函数级变量或全局变量。