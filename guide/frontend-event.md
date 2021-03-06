# 事件监听（前端）

[[toc]]

::: warning

本节内容仅适用于 Blessing Skin v4 或更高版本

:::

## 简介

从 Blessing Skin v4 开始，Blessing Skin 的前端提供了轻量级的事件系统。它提供简单的事件监听和事件触发功能，方便插件以侵入性较低地方式实现一些操作。当然了，您也可以利用这个事件系统，由您来触发一些事件，这对于不同插件之间的相互工作甚至是通信是很有帮助的。

事件系统的所有可用方法都在 `blessing.event` 下（`blessing` 是全局变量）。

## 监听事件

最常见的操作是监听来自 Blessing Skin 的事件，实际上您也可以监听由您自己触发的事件，或者是来自其它插件的事件。不管如何，用法都是一样的，而且很简单。

`blessing.event` 上的 `on` 方法可以让您监听事件。`on` 方法接收两个参数，第一个参数是字符串或 Symbol，表示事件的名称；第二个参数是一个回调函数，当事件被触发时，这个回调函数就会被调用，而这个回调函数可以接收一个参数（也可以没有参数，比如当那个事件触发时没有参数传递，又比如有参数但您不需要），然后事件被触发时，事件的一些数据会被作为参数传递到这个回调参数中。

下面是一个监听 `mounted` 事件的例子。

```javascript
blessing.event.on('mounted', () => {
  // ...
})
```

当 `mounted` 事件被触发时，后面的回调函数就会被执行。

同一事件可以被监听多次。当事件被触发时，同一事件中所有的回调函数都会被执行。

## 触发事件

您也可以使用此事件系统来触发事件。

触发事件应该使用 `blessing.event` 中的 `emit` 方法。`emit` 方法接收两个参数，第一个参数是字符串作为事件名；第二个参数是您想要传递给回调函数的数据（如果不需要传递数据可以直接省略这个参数），推荐您传递 JavaScript 对象，这样做有两个好处：一是允许回调函数修改您传递的数据，以便让插件按照需要去增删改其中的数据，二是方便以后如果需要传递更多的数据，就可以直接向对象中添加属性而无需更改数据类型，从而保持向后兼容性。

举个例子，我们打算要触发一个名为 `eventA` 的事件，并传递一个 `data` 变量过去。那么可以这样写：

```javascript
blessing.event.emit('eventA', data)  // 假设 `data` 变量已定义
```

正如上文所说的，同一事件中所有的回调函数都会被执行。

值得注意的是，要小心如果有多个正在监听的回调函数，小心它们之间对数据的修改会不会有冲突。如果您希望传递一个只读的 JavaScript 对象，那么您可以传递 `Object.freeze(data)` 作为参数，这会将该对象的所有属性设为只读。（注意，`Object.freeze` 只做一层的处理，多层嵌套的属性仍然可以被修改，此时您可能需要递归地调用 `Object.freeze`）

## 所有可用的事件

这里列出了 Blessing Skin 前端中所有定义了的事件。事件触发时，向回调函数传递的参数都是 JavaScript 对象。因此，我们约定，我们使用三级标题表示事件的名称，用四级标题列出该参数上的属性。而每个属性的「类型」均使用 TypeScript / Flow 中的类型语法来表示。

### beforeFetch

::: tip 触发时机

调用 Fetch API 之前。具体用法见 [前端网络请求](fetch.md#事件) 一节。

:::

#### method

类型：`string`

含义：表示当前请求的 HTTP 动词，例如 `GET` 或 `POST` 等。

#### url

类型：`string`

含义：当前请求中的非完整的、相对的 URL。

#### data

类型：`object | FormData`

含义：要发送到后端的数据。

### fetchError

::: tip 触发时机

当调用 Fetch API 发生错误时。如果网络出现故障，或后端返回的 HTTP 状态码不为 2xx，这个事件都会被触发。具体用法见 [前端网络请求](fetch.md#事件) 一节。

:::

注意，这个事件传递给回调函数的参数永远是 `Error` 实例。

#### message

类型：`string`

含义：错误消息

#### response

类型：`Response | undefined`

含义：如果是因为网络故障等由 Fetch API 本身抛出错误时，这个属性不存在。如果是因此后端响应的状态码不为 2xx 时，这个属性存在，为本次请求所返回的对象。关于 `Response` 可阅读 [这里](https://developer.mozilla.org/zh-CN/docs/Web/API/Response)。

### mounted

::: tip 触发时机

当页面上由 Vue.js 渲染的内容已挂载到页面上后。如果页面上不存在由 Vue.js 渲染的内容，此事件不会被触发。

:::

此事件不含任何参数。