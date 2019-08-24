
# 目录结构
目录结构有两种划分方式：

- 按文件类型（角色）划分  


- 按特性（功能）划分  
特性的粒度大小根据实际情况决定。

不管如何划分都应该避免过多层的嵌套，文件路径最好不要超过4层。

目前国际机票结合使用了这两种划分方式：

common 目录中是一些公用组件，这个目录中按特性划分，即一个组件一个目录。

paga 目录中按角色划分，比如所有 component 在一个目录下，所有 container 在一个目录下，这样存在一个问题，要找某个 component 对应的 container 或者 redux 时不方便，要翻好几层。因此还是按特性划分比较合理。


# 生命周期钩子的使用

除 render 外有一些比较常用的生命周期钩子，还有很少使用或者应该避免使用的生命周期。

我们在代码中发现有一些过度使用生命周期，有些生命周期在未来的版本中会被废弃，不建议使用，它们都有可以替代的方法。

## 常用的生命周期：

### render：

render函数应该是纯函数，不应修改组件状态，也不应该和浏览器交互

如果需要和浏览器交互，可以在 componentDidMount() 完成

当 shouldComponentUpdate() 返回false时，render不会被调用

### constructor：

如果没有state需要初始化，也没有方法需要绑定，那就不需要实现 constructor 方法。

如果实现了constructor 方法，必须在最前面调用 super(props)。

constructor 有两种用途：
1. 初始化 state
2. 绑定事件处理函数

应该避免在构造函数中引入任何副作用或订阅。如果需要这些，在 componentDidMount() 里完成。


### componentDidMount:
- 需要DOM节点的初始化应该放在这里。
- 远端数据请求也应该放在这里。
- 这个方法里也可以设置任何订阅。然后在 componentWillUnmount 中取消订阅。

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

todo

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


# Form 表单校验

## 点击下一步时最外层组件发起对所有 Form 表单的校验

目前有两种实现方式：
- 在最外层获取Form所在的组件的实例，然后调用validateFieldsAndScroll。

todo

缺点是实例要层层传递，最多时要传递6层

- 最外层组件通过一个标识发起一个校验，标识存放在 Redux 中。Form 组件检测到此标识后发起对本组件的校验。

两种方法异曲同工，一种是把实例往上传，一种是把校验标识往下传。最终都是为了调用 validateFieldsAndScroll。


## 表单校验结果被冲

表单校验明明没有通过，但是输入框不显示报错。开发时几个人都遇到这种问题，原因是表单校验后有一次额外的render，校验结果被冲掉了。

目前两种解决方法：
- 找出到底是什么数据变化导致了重新渲染

- 使用 antd 自定义校验

使用自定义校验可以明确的给出当前的校验状态，完全自己控制校验的时机和内容。

缺点是因为存在上层发起的校验，校验要写两遍。

todo



# storage 的使用

使用 sessionStorage，数据结构只有一个层级，避免过多的嵌套。

todo



# Modal 框

## Modal 中 html 的拼接

使用 React.createElement 会显得代码比较臃肿，可以直接使用 JSX



## Modal 回调函数中如何调用 generator 函数

两种方式：
- 使用 co 执行
- 获取 dispatch


# 事件处理函数的绑定

## 四种绑定this方法：
1. 在 constructor 中绑定
```js
constructor(props) {
  super(props);
  this.state = {isToggleOn: true};

  // This binding is necessary to make `this` work in the callback
  this.handleClick = this.handleClick.bind(this);
}

handleClick(e) {
  this.setState(state => ({
    isToggleOn: !state.isToggleOn
  }));
}
```

2. 类字段语法
```js
handleClick = () => {
  console.log('this is:', this);
}
```

3. 在Render中绑定
```js
deleteRow(id, name, e) {
  console.log('it happened');
}
render() {
  return <button onClick={this.deleteRow.bind(this, id, name)}>Click Me</button>;
}
```
*注意，通过 bind 方式向监听函数传参，在类组件中定义的监听函数，事件对象 e 要排在所传递参数的后面。*

bind()方法创建一个新的函数， 当这个新函数被调用时其this置为提供的值，其参数列表前几项置为创建时指定的参数序列。

4. 箭头函数
```js
deleteRow(id, e) {
  console.log('this is:', this);
}

render() {
  return (
    <button onClick={(e) => this.deleteRow(id, e)}>
      Click me
    </button>
  );
}
```
*箭头函数的问题在于每次render都会重新生成一个函数。如果这个回调作为一个prop传递给了子组件，子组件可能会有额外的渲染。*
**所以通常建议使用 constructor 绑定或者使用类字段语法。**


# 继承 Component 即可，强烈建议不要创建自己的基类






