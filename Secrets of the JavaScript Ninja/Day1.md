|                   title                    |        date         |     tags     |
| :----------------------------------------: | :-----------------: | :----------: |
| Secrets of the JavaScript Ninja-打卡第一天 | 2019-07-06 | 理解函数调用 |



# Secrets of the JavaScript Ninja

# Chapter Four

这一章主要是介绍了两个函数的参数 `this` 和 `arguments` 。两者都会被**隐式（_implicit_）**地传递给函数，并且可以在函数体内像显式声明的参数一样被正常访问。
	
其中 `this` 表示被调用函数的上下文（_context_），**是 JavaScript 面向对象的重要组成成分**，而 `arguments` 表示函数调用过程中传递的所有参数。



# arguments

1. 虽然我们可以像数组一样，使用下标（_notation_）访问 `arguments` 里面的元素，但是 **`arguments` 并不是数组**，所以 `arguments` 并不能使用诸如 `map` 、`forEach` 、`sort` 等数组方法。

   

2. `arguments` 相当于函数参数（_parameters_）的别名（_alias_），当我们在函数体内改变 `arguments` 的元素时，相对应的函数参数也会发生改变，同样，改变函数参数，也会影响到对应的 `arguments` 。

   

3. 我们可以使用 ES5 引入的严格模式（_strict mode_）来规避 `arguments` 的这种特性。



# 函数的调用方式

## 作为函数被直接调用

在这种情况下，函数上下文（_function context_）也就是 `this` 的取值有两种可能结果：

1. 非严格模式下，`this` 为全局上下文（_the window object_）。
2. 严格模式下，`this` 为 `undefined` 。



## 作为方法被调用

略



## 作为构造函数被调用

使用 `new` 关键字调用函数会触发以下几个动作：

1. 创建一个空对象
2. 该对象作为 `this` 参数传递给构造函数，从而成为构造函数的上下文
3. 新构造的对象作为 `new` 运算符的返回值



示例图如下：

![new运算符](https://s2.ax1x.com/2019/07/06/Z001Wq.png)

举个例子：

```javascript
function Ninja () {
    this.skulk = function () {
        return this
    }
}

var ninja1 = new Ninja()
var ninja2 = new Ninja()

ninja1.skulk() == ninja1	//	true
ninja2.skulk() == ninja2	// true
```



我们刚才提到过，构造函数的目的是初始化新创建的对象，并且新构造的对象会作为构造函数的调用结果（通过 `new` 运算符）返回。

> 那么，当构造函数本身有返回值时会是什么结果？



这就要分成两种情况了：



### 返回原始值的构造函数

```javascript
function Ninja () {
	this.skulk = function () {
        return true
    }
    return 1		//	构造函数返回一个确定的原始类型值，即数字 1
}

Ninja() === 1		// true

var ninja = new Ninja()
typeof ninja === 'object'		// true
```

测试结果表明，通过 `new` 运算符调用构造函数，原始类型的返回值 1 被忽略了。



### 显式返回对象值的构造函数

```javascript
var puppet = {
    rules: false
}

function Emperor () {
    this.rules = true
    return puppet;		//	尽管初始化了传入的 this 对象，返回该全局对象
}

var emperor = new Emperor()

emperor === puppet		// true
emperor.rules			// false
```

这里设置了一个模棱两可的情况：新生成的对象会传递给构造函数作为函数上下文 `this` ，同时被初始化。但是当我们显式地返回一个完全不同的 `puppet` 对象时，哪个对象会最终作为构造函数的返回值呢？

上面的实例代码已经给出了答案：`puppet` 对象最终作为构造函数调用的返回值，而且在构造函数中对函数上下文的操作都是无效的。最终返回的将是 `puppet` 。



### 总结

- 如果构造函数返回一个对象，则该对象将作为整个表达式的值返回，而传入构造函数的 `this` 将被丢弃。
- 但是，如果构造函数返回的是非对象类型，则忽略返回值，返回新的对象。



## 使用 `call` 和 `apply` 方法调用

### 使用 `call` 和 `apply` 方法来设置函数上下文

```javascript
function juggle () {
    var result = 0
    for (let i = 0; i < arguments.length; ++i) {
        result += arguments[i]
    }
    this.result = result
}

var ninja1 = {}
var ninja2 = {}

juggle.apply(ninja1, [1, 2, 3, 4])
juggle.call(ninja2, 5, 6, 7, 8)

ninja1.result		// 10
ninja2.result		// 26
```

![手动设置函数上下文，产生函数上下文（this）和arguments](https://s2.ax1x.com/2019/07/06/Z0cpcR.png)



### 两者区别

`call` 和 `apply` 的区别仅仅在于前者是一次列出调用参数，而后者是使用参数数组。如图所示：

![call 和 apply 的区别](https://s2.ax1x.com/2019/07/06/Z06qBV.png)



### 强制指定回调函数的函数上下文

简单实现一个 `forEach` ：

```javascript
function forEach(list, callback) {
    for (let i = 0; i < list.length; ++i)
        callback.call(list[i], i)
}

const sutdent = [
    { name: 'Yeoman' },
    { name: 'Lym' },
    { name: 'Yeoman_Li' }
]

forEach(sutdent, function (index) {
    console.log(this.name === sutdent[index].name)
})

// true
// true
// true
```



# 解决函数上下文的问题

## 使用箭头函数解决

> 箭头函数没有自己的 `this` 值，箭头函数的 `this` 与声明所在的上下文的相同。

具体示例就不列出了，基本操作，略



## 使用 `bind` 方法

所有函数均可访问 `bind` 方法，可以创建并返回一个新函数，并绑定在传入的对象上。不管如何调用该函数，`this` 均被设置为对象本身。

看下面的例子：

```javascript
<button id="test">Click Me!</button><script>
	var button = {
		clicked: false,
		click: function(){
			this.clicked = true;
			assert(button.clicked,"The button has been clicked");
		}
    };
	var elem = document.getElementById("test");
	elem.addEventListener("click", button.click.bind(button));
/*使用bi
nd函数创建新函数,绑定到button对象上*/

	var boundFunction = button.click.bind(button);
	assert(boundFunction != button.click,"Calling bind creates a completly new function");
</script>
```

从示例代码中的最后一句断言可以看出,调用bind方法不会修改原始函数,而是创建了一个全新的函数。



# 小结

1. 当调用函数时，除了传入在函数定义中显式声明的参数之外，同时还传入了两个隐式参数：`arguments` 和 `this` 。

   - `arguments` 参数是传入函数的所有参数集合。具有 `length` 属性，表示传入参数的个数，通过 `arguments` 参数还可以获取那些与函数形参不匹配的参数，修改 `arguments` 对象会修改函数实参，可以通过严格模式来规避。
   - `this` 表示函数上下文，即与函数调用相关联的对象。函数的定义方式和调用方式决定了 `this` 的取值。

   

2. 函数的调用方式有 4 种：

   - 作为函数调用
   - 作为方法调用
   - 作为构造函数调用
   - 通过 `apply` 和 `call` 方法调用

   

3. 函数的调用方式影响 `this` 的值：

   - 作为函数调用，非严格模式下，`this` 指向全局 `window` 对象；严格模式下，`this` 指向 `undefined` 。
   - 作为方法调用，`this` 指向调用的对象。
   - 作为构造函数调用，`this` 指向新创建的对象。
   - 通过 `apply` 和 `call` 调用，`this` 指向第一个参数。

   

4. 箭头函数没有单独的 `this` 值，`this` 在箭头函数创建时确定。

5. 所有函数均可使用 `bind` 方法，创建新函数，并绑定到 `bind` 方法传入的参数上。被绑定的函数和原函数具有一致的行为。
