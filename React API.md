# React API

## React.Component

继承 Component 即可，**强烈建议不要创建自己的基类**

## 组件的生命周期

Mounting 阶段的声明周期：

    constructor()
    static getDerivedStateFromProps()
    render()
    componentDidMount()

Updating 阶段的声明周期：

    static getDerivedStateFromProps()
    shouldComponentUpdate()
    render()
    getSnapshotBeforeUpdate()
    componentDidUpdate()

### render：

render函数应该是纯函数，不应修改组件状态，也不应该和浏览器交互

如果需要和浏览器交互，可以在 componentDidMount() 完成

当 shouldComponentUpdate() 返回false时，render不会被调用

### constructor：

如果没有state需要初始化，也没有方法需要绑定，那就不需要实现 constructor 方法。

constructor 中必须在最前面调用 super(props)

constructor 有两种用途：
1. 初始化 state
2. 绑定事件处理函数

避免在构造函数中引入任何副作用或订阅。如果需要这些，在 componentDidMount() 里完成


### componentDidMount:
需要DOM节点的初始化应该放在这里。
远端数据请求也应该放在这里。
这个方法里也可以设置任何订阅。然后在componentWillUnmount中取消订阅。

在此方法中调用 setState()，会触发一次额外的渲染，但它会在浏览器更新屏幕之前发生。这就保证了render会调用两次，但是用户不会看到中间状态。但是要谨慎使用此模式，因为它经常导致性能问题。在大多数情况下，可以在constructor中初始化状态。当需要在呈现的东西取决于 DOM 节点的大小或位置时，比如模态框或提示框之类的，可以这样调用。

### componentDidUpdate:
这里可以操作DOM。

可以对比当前 props 和 之前的 props 来决定是否发起网络请求。

在这个方法中调用setState() 一定要加上条件，否则会导致死循环。

如果想把某些 props 映射到 state，可以考虑直接使用 props。

当 shouldComponentUpdate() 返回 false 时，componentDidUpdate() 不会被调用

### componentWillUnmount
在此方法中清除定时器、取消网络请求、取消订阅


## 较少或者避免使用的生命周期方法

### shouldComponentUpdate
可以考虑使用内置的 PureComponent ，而不是手动重写 shouldComponentUpdate()

PureComponent 执行 props 和 state 的浅比较，减少了跳过必要更新的概率。

shouldComponentUpdate 返回 false 不会阻止子组件在 state 变化时重新渲染。

***我们不建议在 shouldComponentUpdate 中做深层的相等检查或者使用JSON.stringify()，这是非常低效的，会损害性能。***

shouldComponentUpdate() 返回 false 后，UNSAFE_componentWillUpdate()、render()和component.Update()不会被调用。

未来 shouldComponentUpdate() 可能会被视为一个提示，并且返回false仍然可能导致组件的重新渲染。

### static getDerivedStateFromProps()
getDerivedStateFromProps 在 render 之前调用。

如果想执行一个副作用，比如数据获取或者动画，使用 componentDidUpdate 代替

如果想在 prop 改变时重新计算某些数据，可以使用 memoization 代替

此方法在每次渲染时都会被激发。


### UNSAFE_componentWillMount()
在render前调用。

避免在此方法中引入副作用，如果有的话可以使用 componentDidMount() 代替。


### UNSAFE_componentWillReceiveProps(nextProps)

如果是用于在 prop 变化时重新计算某些数据，可以使用memoization代替

如果是用于产生某些副作用，用 componentDidUpdate 代替

如果是用于当 props 变化时重置某些state，用 fully controlled 代替

调用this.setState()通常不会触发 UNSAFE_componentWillReceiveProps


### UNSAFE_componentWillUpdate(nextProps, nextState)

不要在此方法中调用 this.setState()，或者分发一个redux action

此方法可以被 componentDidUpdate 代替


## 其他API
### setState

setState(updater[, callback])

React不保证state的变化会被立即应用

setState 可能会批量执行或者推迟执行

setState() 调用后立即读取 this.state ，可能不会得到最新的值。可以在setState 使用回调函数解决这个问题。


## React.PureComponent

React.PureComponent 实现了 shouldComponentUpdate()，用的是 prop 和 state 的浅比较

当你的props 和 state 都比较简单时，才用PureComponent，或者当你知道深层的数据结构已经变化时使用forceUpdate() ，或者使用immutable对象。

此外，PureComponent 的 shouldComponentUpdate 会跳过整个子树的 prop 更新，要确保子组件也是"pure"的



## ReactDOM

### hydrate
ReactDOM.hydrate(element, container[, callback])

用于服务端渲染

### findDOMNode()
ReactDOM.findDOMNode(component)

获取原生的浏览器DOM元素。多数情况下可以用ref代替


## DOM 元素
- 在React中，所有的DOM properties 和 attributes 都应该是驼峰的。  
比如，tabindex 应该写成 tabIndex。aria-* 和 data-* 是例外，仍然是小写的

- 在 React 和 HTML 中 Attributes 的不同：  
比如 checked 用于构建受控组件，defaultCheck 用于设置初始值


- dangerouslySetInnerHTML  
就像在DOM中使用innerHTML
```js
function MyComponent() {
  return <div dangerouslySetInnerHTML={{__html: 'First &middot; Second'}} />;
}
```

- htmlFor  
用于表示HTML中的for

- style  
style属性接受一个JS对象，对象中的属性是驼峰的  
通常不推荐使用样式属性作为样式的主要方法。style 最常用于在渲染时动态计算出的样式。
```js
const divStyle = {
  color: 'blue',
  backgroundImage: 'url(' + imgUrl + ')',
};

function HelloWorldComponent() {
  return <div style={divStyle}>Hello World!</div>;
}
```
在内联样式中，Read会在数字属性后自动加上“px”。如果你使用的单位不是"px"，将值加上单位写成字符串的形式。
```js
<div style={{ height: '10%' }}>
  Hello World!
</div>
```

## SyntheticEvent 合成事件

在事件处理函数中return false 不能阻止时间冒泡，e.stopPropagation() 或者 e.preventDefault() 应该被手动触发。

事件池：合成事件会被放到事件池中，我们不能异步获取event

事件处理函数会在冒泡阶段(bubbling phase)触发。

要为捕获阶段(capture phase)注册事件处理器，把 Capture 加到事件名后面。比如事件名不要写成onClick，而是使用 onClickCapture 在捕获阶段处理单击事件。

