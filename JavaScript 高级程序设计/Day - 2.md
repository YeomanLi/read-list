|                title                 |    date    |  tags   |
| :----------------------------------: | :--------: | :-----: |
| JavaScript 高级程序设计 - 打卡第二天 | 2019-07-23 | BOM、H5 |

# JavaScript 高级程序设计

# Chapter - 10

这一章主要讲的是 DOM 相关。

<!-- more -->

## 节点层次

### Node 类型

每个节点都有一个 `childNodes` 属性，其中保存着一个 `NodeList` 对象。 `NodeList` 是一种类数组
对象，用于保存一组有序的节点，可以通过位置来访问这些节点。

```javascript
var firstChild = someNode.childNodes[0]
var secondChild = someNOde.childNodes.item(1)
```

`NodeList` 是类数组对象，可以转换为真正的数组，书中给出的方法：

```javascript
var arrayOfNodes = Array.prototype().slice.call(someNode.childNodes, 0)
```

![节点层次关系](https://s2.ax1x.com/2019/07/22/eP20l6.png)

#### 操作节点

如果 `appendChild` 传入的节点已经是文档的一部分了，那么结果就是将节点从原来的位置转移到新的位置。

如果 `insertBefore` 的第二个参数（参照节点）是 `null` ，那么就是和 `appendChild` 一样的效果。

`replaceChild` 、`removeChild` 就不说了。

#### 其他方法

`cloneNode` 用于创建调用此方法的节点一个完全相同的副本，接收一个参数，`true` 表示深复制，`false` 表示浅复制。

深复制除了复制当前节点之外，还会复制整个子节点树。

注意：**不会复制添加到 DOM 节点中的 JavaScript 属性，例如事件处理程序等**。



### Document 类型

`getElementById` ID 必须与页面中元素的 id 特性 (attribute) 严格匹配，包括大小写。

如果页面中存在多个 `id` 相同的元素，那么 `document.getElementById()` 会返回第一次出现的元素。



### Element 类型

`nodeType` 的值为 1 。

#### 获取特性

获取元素的类名，可以通过直接访问对象的属性、`getAttribute()` 来获取。但是前者使用的是 `element.className` ，后者使用的是 `element.getAttribute(class)` 。

此外，`getAttribute` 还可以获取元素自定义的属性。

特性的名称是不区分大小写的，即 "ID" 和 "id" 代表的都是同一个特性。另外也要注意，根
据 HTML5 规范，自定义特性应该加上 data- 前缀以便验证。（Vue 项目通过浏览器开发者工具可以看到 ）

两类特殊的特性，通过属性的值和 `getAttribute` 来访问得到不同的结果：

1. `style` ，通过属性的值访问返回一个对象，而通过 `getAttribute` 访问得到的是 CSS 文本。
2. `onclick` 这样的事件处理程序，访问 `onclick` 属性时，返回的是一个 JavaScript 函数，如果没有则返回 `null` ，而通过 `getAttribute` 访问时返回相应代码的字符串。

#### 元素的子节点

元素类型也可以使用 `getElementsByTagName()` 方法。

比如，当我们想访问指定 `ul` 下的 `li` ：

```javascript
var ul = document.getElementById('myList')
var liList = ul.getElementsByTagName('li')
```

不过，这样**得出的是直接后代子元素**。



### Attr 类型

`nodeType` 的值为 2 。

```javascript
var attr = document.createAttribute("align")
attr.value = 'left'
element.setAttributeNode(attr)

console.log(element.getAttribute("align"))
console.log(element.getAttributeNode["align"].value)
console.log(element.attributes["align"].value)
```



## DOM 操作技术

### 动态脚本

```javascript
var script = document.createElement('script')
script.type = "text/javascript"
script.src = 'xxx.js'
document.body.appendChild(script)
```



### 使用 NodeList

NodeList 是动态的，每当文档结构发生变化时，都会得到更新。从本质上说，所有 NodeList 对象都是在访问 DOM 文档时实时运行的查询。例如，下面代码就会无限循环运行：

```javascript
var divs = document.getElementsByTagName("div"),
    div,
    i

for (i = 0; i < divs.length; ++i) {
    div = document.createElement("div")
    document.body.appendChild("div")
}
```

尽量减少访问 NodeList 的次数。因为每次访问 NodeList ，都会运行一次基于文档的查询。



# Chapter - 11

这一章主要讲的是新的 Selectors API 以及 HTML5 DOM 扩展。

<!-- more -->

## 选择符 API

`querySelector` 返回模式匹配的**第一个**元素，如果没找到匹配的元素，返回 `null` 。

`querySelectorAll` 返回的也是 NodeList 对象，不管找到多少匹配的对象。如果没有找到匹配的对象，那么 NodeList 就是空的。

具体来说，`querySelectorAll` 返回的值实际上是带有==**所有属性和方法**==的NodeList ，而其底层实现则类似于一组元素的**快照**，而非不断对文档进行搜索的动态查询。这样实现可以避免使用 NodeList 对象通常会引起的大多数性能问题。



## HTML 5

### 与类相关的扩充

新增了一种操作类名的方式，可以让操作更简单也更安全，那就是为所有元素添加 classList 属性。这个 classList 属性是新集合类型 DOMTokenList 的实例。这个类型除了 `length` 属性和 `item()` 方法，还定义了如下方法：

- add
- contains
- remove
- toggle：如果存在，那么删除，不存在就添加。



### 焦点管理

```javascript
var btn = document.getElementById('btn')
btn.focus()
console.log(document.activeElement === btn)	//	true
console.log(document.hasFocus())	//	true
```

默认情况下，文档刚刚加载完成时，`document.activeElement` 保存的是 `document.body` 元素的引用。文档加载期间，`document.acitveELement` 的值为 `null` 。

通过检测文档是否获得了焦点，可以知道用户是不是正在于页面交互。



### HTMLDocument 的变化

新增了 `readyState` 属性：

- loading：正在加载文档
- complete：已经加载完文档



### 字符集属性

HTML5 规定可以为元素添加非标准的属性，但要添加前缀 data- ，目的是为元素提供与渲染无关的
信息，或者提供语义信息。这些属性可以任意添加、随便命名，只要以 data- 开头即可。

```html
<div id="myDiv" data-appId="12345" data-myname="Nicholas"></div>
```

```javascript
div = document.getELementById('myDiv')
var appId = div.dataset.appId
div.dataset.appId = 23456
```



### 插入标记

`innerHTML` 、`outerHTML` 。



# Chapter - 12
这一章主要讲了 DOM2 和 DOM3 的变化、操作样式的 DOM API 、DOM 的遍历与范围。

<!-- more -->

## 样式

### 元素大小

#### 偏移量

包括元素在屏幕上占用的所有可见的空间。元素的可见大小由其高度、宽度决定，包括所有内边距、滚动条大小和边框大小（不包括外边距）。

![偏移量](https://s2.ax1x.com/2019/07/23/eFRtHO.png)

`offsetTop` 和 `offsetLeft` 属性与包含元素有关，包含元素的引用保存在 `offsetParent` 属性中。`offsetParent` 属性不一定与 `parentNode` 值相等。例如，`<td>` 元素的 `offsetParent` 是作为其祖先元素的 `<table>` 元素，因为 `<table>` 是在 DOM 层次中距 `<td>` 最近的一个具有大小的元素。

#### 客户区大小

元素内容及其内边距所占据的空间大小。

![客户区大小](https://s2.ax1x.com/2019/07/23/eF4qyR.png)

不包含滚动条的空间。

#### 滚动大小

- scrollHeight：在没有滚动条的情况下，元素内容的总高度
- scrollWidth：在没有滚动条的情况下，元素内容的总宽度
- scrollLeft：被隐藏的内容区域左侧的像素数
- scrollTop：被隐藏在内容区域上方的像素数

![滚动大小](https://s2.ax1x.com/2019/07/23/eFoTfS.png)

#### 确定元素的大小

通过 `getBounding()` 方法，返回一个矩形对象，包含 4 个属性：left、top、right、bottom 。