|                    title                     |    date    |       tags       |
| :------------------------------------------: | :--------: | :--------------: |
| Secrets of the JavaScript Ninja - 打卡第四天 | 2019-07-09 | 对象、原型、继承 |



# Secrets of the JavaScript Ninja



# Chapter - 7

这一章主要讲的是面向对象和原型。讲的还是比较简单的，关于对象与原型还是需要好好看看《JavaScript高级程序设计》啊。

<!-- more -->

## 对象构造器与原型

### 原型与实例

- 每一个函数都有一个原型对象
- 每一个函数的原型都具有一个 `constructor` 属性，该属性指向函数本身
- constructor 对象的原型设置为新创建的对象的原型

![示意图](https://s2.ax1x.com/2019/07/09/Z6lCbd.png)

### 属性的优先级

> 实例方法和与之同名的原型方法，会优先使用实例方法

看下面的例子：

```javascript
function Person () {
    this.isMale = false
    this.checkMale = function () {
        return !this.isMale
    }
}

Person.checkMale = function () {
    return this.isMale
}

const person = new Person()
console.log(person.checkMale())
// true
```

原因很简单，在实例中找到了属性，就不需要遍历原型链了。



### 通过原型，一切都可以在运行时修改

修改了函数原型，旧的实例依然引用旧的原型

听起来比较拗口，直接看例子吧：

```javascript
function Ninja () {
    this.swung = true
}
const ninja1 = new Ninja()

Ninja.prototype.swingSword = function () {
    return this.swung
}

console.log(ninja1.swingSword())    //  true

Ninja.prototype = {
    pierce: function () {
        return true
    }
}

console.log(ninja1.swingSword())    // true
console.log(ninja1.pierce())    // 报错！xxx is not a function

const ninja2 = new Ninja()
console.log(ninja2.pierce())    // trues
console.log(ninja2.swingSword())    // 报错！xxx is not a function

```

可以看到，首先我们在实例创建完成后，又在原型上添加了一个方法，发现之前创建好的实例可以使用这个方法。此时的程序状态如图所示：

![状态图 - 1](https://s2.ax1x.com/2019/07/09/Z63EjS.png)

接着，我们使用字面量对象完全重写了 `Ninja` 的原型对象，发现已经构建的对象实例依然引用的是旧的原型。此时的程序状态如图所示：

![状态图 - 2](https://s2.ax1x.com/2019/07/09/Z63l90.png)

最后，我们又使用这个构造函数创建了一个新的对象 `ninja2` ，发现访问不到旧的 `swingSword` 方法，当然，可以访问新的 `pierce` 方法。

此时的程序状态如下：

![状态图 - 3](https://s2.ax1x.com/2019/07/09/Z63B36.png)

总结一下：对象与函数之间的引用关系是在对象创建时建立的。新创建的对象将引用新的原型，只能访问新的方法，原来旧的对象保持着原有的原型，仍然能够访问旧的方法。



## 实现继承

### 重写 `constructor` 属性

看下面的代码：

```javascript
function Person () {}
Person.prototype.dance = function () {}

function Ninja () {}
Ninja.prototype = new Person()
const ninja = new Ninja()
```

![示意图](https://s2.ax1x.com/2019/07/09/Z6JPhR.png)

但是这会引发一个问题：我们已经丢失了 `Ninja` 和 `Ninja` 初始原型之间的关联。

```javascript
console.log(ninja.constructor)  // [Function: Person]
```

原因在于无法查找到 `Ninja` 对象的 `constructor` 属性。回到原型上，原型上也没有 `constructor` 属性，继续在原型链上追溯，在 `Person` 对象的原型上具有指向 `Person` 本身的 `constructor` 属性。



### 解决 `constructor` 属性的问题

很简单，只需在使用 `Object.defineProperty` 方法在 `Ninja.prototype` 对象上添加新的 `constructor` 属性即可。

```javascript
Object.defineProperty(Ninja.prototype, "constructor", {
    enumerable: false,
    value: Ninja,
    writable: true
})
```