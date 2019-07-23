|                title                 |    date    |  tags   |
| :----------------------------------: | :--------: | :-----: |
| JavaScript 高级程序设计 - 打卡第一天 | 2019-07-20 | BOM、H5 |

# JavaScript 高级程序设计

# Chapter - 8

这一章主要讲的是 BOM 对象。

<!-- more -->



## window 对象

### 全局作用域

由于 `window` 对象同时扮演着 ECMAScript 中 `Global` 对象的角色，因此，**所有在全局作用域中声明的变量、函数都会变成 `window` 对象的属性和方法。**

==定义全局变量不能通过 `delete` 操作符删除，而直接在 `window` 对象上定义的属性可以。==

看下面的例子：

```javascript
var age = 29
window.color = 'red'

delete window.age	//	在 IE < 9 时抛出错误,在其他所有浏览器中都返回 false
delete window.color	// 在 IE < 9 时抛出错误,在其他所有浏览器中都返回 true
```

原因：使用 `var` 语句添加的属性有一个名为 `[[Configurable]]` 的特性，这个特性的值被设置为 `false` ，因此这样定义的属性不可以通过 `delete` 操作符删除。



尝试访问未声明的变量会抛出错误，但是通过查询 `window` 对象，可以知道某个可能未声明的变量是否存在，例如：

```javascript
var newVal = oldVal	//	抛出错误，因为 oldVal 并未定义
var newVal = window.oldVal	//	不会抛出错误，因为这是一次属性查询，值为 undefined
```



## location 对象

除了 `href` 之外，修改 `location` 对象的其他属性也可以改变当前加载的页面。如：hash、hostname、pathname、port、search等。

`replace` 和以上这些的区别在于：当通过上述任何一种方法修改 URL 之后，浏览器的历史记录就会生成一条新的记录，因此用户可以后退到之前的页面；而 `replace` 方法不会生成新的历史记录，不可以返回之前的页面。



## history 对象

几个常见的 API ：go、forward、backward。

`go` 方法可以传递一个字符串参数，此时浏览器会跳转到历史记录中包含该字符串的第一个位置——可能后退，也可能前进，具体看哪个位置最近。如果历史记录中不包含该字符串，那么浏览器什么也不做。



# Chapter - 16

这一章主要讲的有关 HTML 5 。拖动、媒体元素、历史状态管理等。

<!-- more -->



## 原生拖放

### 拖放事件

拖动元素的时候，会依次触发以下事件：

- dragstart
- drag
- dragend

在元素被拖动期间会持续触发 drag 事件。



当某一个元素被拖动到一个有效的放置元素**内**时，下列事件会依次发生：

- dragenter
- dragover
- dragleave（当被拖动元素没有放下就离开目的地元素时触发） 或 drop

被拖动的元素还在放置目标的范围内移动时，就会持续触发 dragover 事件，如果元素被放到了放置目标中，则会触发 drop 事件而不是 dragleave 事件。



### dropEffect 和 effectAllowed

通过 dropEffect 属性可以知道被拖动的元素能够执行哪种放置行为，有以下 4 种可能的值：

- none：不能把拖动的元素放在这里，只是除了文本框之外所有元素的值
- move：应该把拖动的元素移动到放置目标
- copy：应该把拖动的元素复制到放置目标
- link：表示放置目标会打开拖动的元素（但拖动的元素必须是一个链接，有 URL ）

要使用 dropEffect 属性，必须在 ondragenter 事件处理程序中针对放置目标来设置它。

dropEffect 属性只有搭配 effectAllowed 属性才有用。 effectAllowed 属性表示允许拖动元
素的哪种 dropEffect , effectAllowed 属性可能的值如下：

- uninitialized：没有给被拖动的元素设置任何放置行为
- none：被拖动的元素不能有任何行为
- copy：只允许 copy
- link：只允许 link
- move：只允许 move
- copyLink：允许 copy 和 link
- copyMove：...
- linkMove：...
- all：允许任意 dropEffect



## 历史状态管理

通过 `hashchange` 事件，可以知道 URL 的参数什么时候发生了变化，即什么时候该有所反应。而通过状态管理 API ，能够在不加载新页面的情况下改变浏览器的 URL 。为此，需要使用 `history.pushState()` 方法。

接收三个参数：状态对象、新状态的标题、可选的相对 URL 。

```javascript
history.pushState({name: 'Yeoman'}, 'new hash', '/index')
```

因为 `pushState()` 会创建新的历史状态,所以你会发现“后退”按钮也能使用了。

![补充说明](https://s2.ax1x.com/2019/07/23/eFb8B9.png)