|                    title                     |    date    |               tags               |
| :------------------------------------------: | :--------: | :------------------------------: |
| Secrets of the JavaScript Ninja - 打卡第三天 | 2019-07-08 | 生成器、Promise 对象、模拟 async |



# Secrets of the JavaScript Ninja



# Chapter Six

# 使用生成器函数

调用生成器后，就会创建一个迭代器（_iterator_），使用迭代器的 `next` 方法向生成器请求一个值，用于控制生成器。

`next` 函数调用之后，生成器就开始执行代码，当代码执行到 `yield` 关键字时，就会生成一个中间结果（生成值序列中的一项），然后返回一个封装了结果值和指示完成的指示器的**对象**。

**每当生成一个当前值后，生成器就会非阻塞地挂起执行，耐心等待下一次值请求的到达。**



## 实际用途举例

### 使用生成器生成 ID 序列

```javascript
function * idGenerator () {
    var id = 0
    while (true) {
        yield ++id
    }
}

const idIterator = idGenerator()
const ninja1 = { id: idIterator.next().value }
const ninja2 = { id: idIterator.next().value }
const ninja3 = { id: idIterator.next().value }

console.log(ninja1.id)
console.log(ninja2.id)
console.log(ninja3.id)

// 1
// 2
// 3
```

Q：为什么代码不会无限 `while` 循环下去？

A：当生成器遇到了一个 `yield` 语句，它就会一直挂起执行直到下次调用 `next` 方法，所以只有每次调用一次 `next` 方法，`while` 循环才会迭代一次并返回下一个 ID 值。



### 使用迭代器遍历 DOM 树

```javascript
function * DomTraversal (el) {
    yield el
    el = el.firstElementChild
    while (el) {
        yield * DomTraversal(el)
        el = el.nextElementSibling
    }
}
const subTree = document.getElementById('subTree')
for (let el of DomTraversal(subTree))
    console.log(el)
```



## 探索生成器内部构成

生成器的 4 个状态：

- 挂起开始——创建一个生成器后，它最先以这种状态开始。其中的任何代码都未执行。
- 执行——生成器对应的迭代器调用了 `next` 方法，并且当前存在可执行的代码时，生成器都会转移到这个状态。
- 挂起让渡——生成器遇到 `yield` 表达式，创建并返回对象后，会再次挂起执行。生成器在这个状态暂停并等待继续执行。
- 完成——生成器执行期间，如果代码执行到 `return` 语句或者全部代码执行完毕，生成器就会进入这个状态。



![生成器状态转换示意图](https://s2.ax1x.com/2019/07/08/ZsZPxI.png)

### 生成器的执行上下文

太多了，直接看书吧🤦	P 190



# 使用 `Promise`

各种基本操作，略



# 把生成器和 `promise` 相结合

> 这一部分尤为精彩！可以说是目前看的几章中最为精彩的一章了！读起来有种酣畅淋漓的快感，由于篇幅太长，这里仅仅是简单摘录、总结一些内容，强烈建议这一部分仔仔细细回看个5、 6、 7、 8遍！

P 210



这部分基本就是使用生成器和 `promise` 结合起来，实现了简单的 `async` 和 `await` 。

主要思想其实就是利用生成器中让渡后会挂起执行而不会发生阻塞的特性。将异步任务放入一个生成器中，然后执行生成器函数。在该生成器执行的过程中，每执行到一个异步任务，就创建一个`promise` 用于代表该异步任务的执行结果。

当然，我们无法得知 `promise` 何时“兑现”（或者会不会兑现），所以在生成器执行的时候，我们会将执行权让渡给生成器，从而不会导致阻塞。过了一会儿，当承诺被兑现，我们会继续通过迭代器的 `next` 方法执行生成器。

看下面的实现代码：

```javascript
function async (generator) {
    var iterator = generator()
    function handle (iteratorResult) {
        if (iteratorResult.done)    return
        const iteratorValue = iteratorResult.value
        if (iteratorValue instanceof Promise) {
            iteratorValue.then(res => handle(iterator.next(res)))
                         .catch(err => iterator.throw(err))
        }
    }
    try {
        handle(iterator.next())
    } catch (err) { iterator.throw(err) }
}

// 使用
async(function * () {
    try {
        const ninjas = yield getJSON("data/ninjas.json")
        const missions = yield getJSON("ninjas[0].missionsUrl")
        const missionDescription = yield getJSON("missions[0].detailsUrl")
    } catch (err) {
        // do something
    }
})
```

结合了很多之前的内容：

- 函数是第一类对象——我们向 `async` 函数中传入了一个函数作为参数。
- 生成器函数——用它的特性来挂起和恢复执行。
- promise——帮我们处理异步代码。
- 回调函数——在 `promise` 对象上注册成功和失败的回调函数。
- 箭头函数——箭头函数的简洁适合用在回调函数上。
- 闭包——在我们控制生成器的过程中，迭代器在 `async` 函数内被创建，随之我们在 `promise` 的回调函数内通过闭包来获取该迭代器。



# 总结

- 生成器是一种不会在同时输出所有值序列的函数，而是基于每次的请求生成值。
- 不同于标准函数，生成器可以挂起和恢复它们的执行状态。当生成器生成了一个值后，它将会在不阻塞主线程的基础上挂起执行，随后静静地等待下次请求。
- 我们可以通过 `yield` 关键字来生成一个值并挂起生成器的执行。如果我们想让渡到另一个生成器中，可以使用 `yield*` 操作符。
- 我们能够通过 `next` 函数向生成器中传入值。