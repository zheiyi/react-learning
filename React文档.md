# React 文档

React 是一个声明式的，高效且灵活的用来构建用户界面的框架。

## React 相关工具链推荐

1.  Create React App  
    构建单页应用

2.  Next.js  
    静态和服务端渲染应用

3.  Gatsby  
    创建静态网站

4.  Razzle  
    服务端渲染框架

## React 理念

- 可以自顶向下(从层级最高的组件开始)或者自底向上(从层级最低的组件开始)构建应用。在较为简单的例子中，通常自顶向下更容易，而在较大的项目中，自底向上会更容易并且在你构建的时候有利于编写测试。

- 将组件切分为更小的组件。

- 从组件自身的角度来命名 props，而不是根据使用组件的上下文命名。

- props 是只读的，组件不应该直接修改它。

## 生命周期钩子

### componentDidMount

    组件输出到 DOM 后执行

## State

React 可能将多个 setState() 调用合并成一个调用来提高性能。

如果 setState 中需要用到当前的 state，那么 setState 应该接受一个函数作为参数，而不是一个对象。此函数将接收先前的 state 作为第一个参数，第二个参数是实时的 props。

```javascript
this.setState((state, props) => ({
  counter: state.counter + props.increment,
}));
```

React 中的单向数据流指的是**组件间的数据自顶向下流动**。任何状态始终由某些特定组件所有，并且从该状态导出的任何数据或 UI 只能影响树中下方的组件。

## 事件处理

React 事件使用驼峰命名，而不是像 DOM 中的全小写。

在 JSX 语法中，应该传入一个函数作为事件处理函数，而不是一个字符串(DOM 元素的写法)。

return false 不能阻止默认行为，必须显式调用 preventDefault。
- 在 jQuery 中使用 return false 时，相当于同时使用 event.preventDefault 和 event.stopPropagation ，它会阻止冒泡也会阻止默认行为。  
- 使用原生 js 时，return false 只会阻止默认行为（例如\<a\>标签的默认行为就是跳转到指定页面）。

```js
handleClick(e) {
  e.preventDefault();
  console.log('The link was clicked.');
}

render() {
  return (
    <a href="#" onClick={handleClick}>
      Click me
    </a>
  );
}
```
e 是一个合成事件(Synthetic Event)。React 根据 W3C spec 来定义这些合成事件。

### 四种绑定this方法：
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
*箭头函数的问题在于每次render都会重新生成一个函数。如果这个回调作为一个prop传递给了子组件，子组件可能会有额外的渲染。
所以通常建议使用 constructor 绑定或者使用类字段语法。*

## 条件渲染
- 简便的与运算符 &&
```js
return (
  <div>
    <h1>Hello!</h1>
    {unreadMessages.length > 0 &&
      <h2>
        You have {unreadMessages.length} unread messages.
      </h2>
    }
  </div>
);
```
false 会被 React 忽略并跳过

- 三元运算符
```js
render() {
  const isLoggedIn = this.state.isLoggedIn;
  return (
    <div>
      The user is <b>{isLoggedIn ? 'currently' : 'not'}</b> logged in.
    </div>
  );
}
```

- 防止组件渲染  
render 方法返回 null 可以隐藏组件，但是不会影响组件的生命周期，比如componentDidUpdate 仍会被调用。


## 列表

渲染多个组件
```js
const numbers = [1, 2, 3, 4, 5];
const listItems = numbers.map((number) =>
  <li>{number}</li>
);
```

Keys帮助react识别哪些项有新增移除或改变
```js
const todoItems = todos.map((todo, index) =>
  // Only do this if items have no stable IDs
  <li key={index}>
    {todo.text}
  </li>
);
```
在map()中的元素需要加上key。  
把 index 作为 key 是万不得已的办法。



## JSX
JSX从本质上说是React.createElement(component, props, ...children)的语法糖。  
书写JSX代码的时，React必须在作用域内。  
自定义的组件必须大写开头，否则React会把它当成HTML标签。  
JS 表达式应该用{}包裹。

字符串常量可以使用以下两种方法传递：
```js
<MyComponent message="hello world" />
<MyComponent message={'hello world'} />
```

prop如果没有值，则默认为true。以下两个表达式相等
```js
<MyTextBox autocomplete />
<MyTextBox autocomplete={true} />
```

... 扩展操作符
```js
const props = {firstName: 'Ben', lastName: 'Hector'};
return <Greeting {...props} />;
```

JSX中的Children  
两个标签之间的内容会被当成一个特殊prop传递：props.children  
children的形式有：

1. 字符串常量
```js
<MyComponent>Hello world!</MyComponent>
```
JSX会移除头部和尾部的空格，也会移除空行。

2. React组件返回元素数组
```js
render() {
  // No need to wrap list items in an extra element!
  return [
    // Don't forget the keys :)
    <li key="A">First item</li>,
    <li key="B">Second item</li>,
    <li key="C">Third item</li>,
  ];
}
```

3. JS表达式作为Children
```js
<MyComponent>foo</MyComponent>
<MyComponent>{'foo'}</MyComponent>
```

4. 函数作为Children

5. false, null, undefined, 和 true，是合法的children，但是不会渲染
```js
// 错误写法
<div>
  {props.messages.length &&
    <MessageList messages={props.messages} />
  }
</div>
```
*props.messages是空数组时，会显示 0，所以 && 之前的表达式一定要是boolean的*



## 表单Forms
受控组件  
在HTML中，像input textarea select这样的表单组件保存他们自己的状态，并根据用户输入更新。
在React中，这些状态保存在state中，并用setState()更新。由React控制其值的表单元素称为“受控组件”。


## PropTypes 类型检查
```js
import PropTypes from 'prop-types';

class Greeting extends React.Component {
  render() {
    return (
      <h1>Hello, {this.props.name}</h1>
    );
  }
}

Greeting.propTypes = {
  name: PropTypes.string
};
```

## 静态类型检查
推荐使用 Flow 或 TS 代替 PropTypes 
### Flow
Flow 允许用特殊类型语法注释变量、函数和组件，并尽早捕获错误  
步骤：  
1. npm install --save-dev flow-bin
2. package.json中添加flow
3. npm run flow init
4. npm run flow

Flow 只会检查带有这种注释的文件 // @flow

### TypeScript
步骤：  
1. 添加TS依赖
2. 配置TS编译选项
3. 使用正确的文件扩展名
4. 添加在用的库的定义

库的类型声明：
1. 库本身绑定了声明文件。检查库中是否有 index.d.ts 文件。
2. 使用 DefinitelyTyped 。DefinitelyTyped 是一个仓库，它存放着没有绑定声明文件的库的类型声明。
3. 对于既没有绑定声明，又不存在于 DefinitelyTyped 中的库，我们可以本地声明。在根目录创建一个 declarations.d.ts 文件：
```js
declare module 'querystring' {
  export function stringify(val: object): string
  export function parse(val: string): object
}
```

在 create-react-app 中使用 TypeScript  
可以使用第三方库 react-scripts-ts 或 typescript-react-starter.

## Refs 和 DOM

Refs提供了一种访问DOM节点的方法。  
少数几个好的使用refs的场景：
- 处理焦点、文本选择或媒体控制。
- 触发强制动画。
- 集成第三方 DOM 库。

不要滥用Refs，多数情况下可以用"state提升"来解决问题


### 创建Refs

1. createRef
```js
class MyComponent extends React.Component {
  constructor(props) {
    super(props);
    this.myRef = React.createRef();
  }
  render() {
    return <div ref={this.myRef} />;
  }
}
```
获取Ref
```js
const node = this.myRef.current;
```

HTML元素：通过current属性获得DOM元素。

    是否点击了面板外面
    this.toggleContainer.current.contains(event.target)

自定义类组件：current中是组件的实例。

函数式组件上不能使用Ref，因为他们没有实例。

ref 的更新发生在componentDidMount 或 componentDidUpdate生命周期之前。

2. Callback Refs
```js
class CustomTextInput extends React.Component {
  constructor(props) {
    super(props);

    this.textInput = null;

    this.setTextInputRef = element => {
      this.textInput = element;
    };
  }

  render() {
    // Use the `ref` callback to store a reference to the text input DOM
    // element in an instance field (for example, this.textInput).
    return (
      <div>
        <input
          type="text"
          ref={this.setTextInputRef}
        />
      </div>
    );
  }
}
```
当组件挂载后，React会传递DOM元素给ref callback，当组件卸载时，会传递null。  
Refs会在componentDidMount 或 componentDidUpdate 之前更新。  
*Ref Callback 不要写成内联函数。*


### 将DOM REF暴露给父组件
在很少的情况下，可能希望在父组件中访问一个孩子的DOM节点。

可能用于触发子节点的focus或者测量子节点的大小或位置。

推荐使用 Ref Forwarding。

如果你完全不能控制子组件，你最后的选择是使用 finddomnode()，但是不推荐使用此方法。


## Forwarding Refs
通常来说不需要获取组件内部的 DOM 元素的 ref ，这样可以避免组件间依赖彼此的 DOM 结构。

但对于像 Button 或 TextInput 这种处处用到的小组件，有时会不可避免的需要获取他们的 DOM 节点，比如操作focus，选中等。

```js
const FancyButton = React.forwardRef((props, ref) => (
  <button ref={ref} className="FancyButton">
    {props.children}
  </button>
));

// You can now get a ref directly to the DOM button:
const ref = React.createRef();
<FancyButton ref={ref}>Click me!</FancyButton>;
```

在常规的函数式或类组件中，不存在 ref 参数，ref 不是一个 prop，ref 也不能通过 props 获取。


## 非受控组件
受控组件：表单数据由组件控制  
非受控组件：表单数据由DOM自身控制  
非受控组件中使用defaultValue或defaultChecked来设置初始值

## 性能优化
1. 使用生产版本  
在生产模式下，React开发者工具的背景是黑色的，在开发模式下是红色的
2. 使用 uglify-js-brunch 插件
3. 使用 Browserify
4. rollup rollup-plugin-commonjs rollup-plugin-replace rollup-plugin-uglify 
5. DevTools Performance
6. 长列表：使用react-window react-virtualized 
7. 避免Reconciliation(一致)：  
   当组件的props或者state改变时，React通过比较新返回的元素和先前呈现的元素来确定是否需要实际的DOM更新。  
   继承React.PureComponent，相当于实现了shouldComponentUpdate()，将当前和之前的props和state进行浅比较。  
   如果shouldComponentUpdate返回false，React不会调用render，否则React会比较render返回的元素和之前的是否相同。


## 协调（Reconciliation）
### key的选用
可以选择已有的唯一ID作为key，比如
```js
<li key={item.id}>{item.name}</li>
```
如果没有id，可以hash部分要展示的内容生成key。  
key值只需要在兄弟节点中唯一，不必全局唯一。  
数组中的索引也可以作为key值，但是如果列表会重新排序，可能会出现速度慢和组件状态混乱的问题。


## Context

在典型的React应用中，数据是自顶向下通过props传递的，这对于某些特定类的props来说比较麻烦。

Context提供了一种在组件间共享数据的方法，避免了数据在组件树之间的逐级递。

Context可以看做全局共享数据，比如用户信息，主题，语言等。

Context允许我们将一个值传递到组件树中，而不必显式地通过每个组件。

Context可能会使组件的复用更加困难，所以不要过度使用。

```js
<Provider value={/* some value */}>
<Consumer>
  {value => /* render something based on the context value */}
</Consumer>
```
当 Provider 提供的值发生变化时，所有的后代 Consumer 会重新渲染。 

Provider 向 Consumers 的数据传输不受 shouldComponentUpdate 影响。

使用 Object.is 来比较新值是否不同于旧值。

将 context 的值当做 prop 传递，就可以在生命周期中获取
```js
<ThemeContext.Consumer>
  {theme => <Button {...props} theme={theme} />}
</ThemeContext.Consumer>
```

如果某个 context 在多处用到，可以考虑使用高阶组件

```js
const ThemeContext = React.createContext('light');

// This function takes a component...
export function withTheme(Component) {
  // ...and returns another component...
  return function ThemedComponent(props) {
    // ... and renders the wrapped component with the context theme!
    // Notice that we pass through any additional props as well
    return (
      <ThemeContext.Consumer>
        {theme => <Component {...props} theme={theme} />}
      </ThemeContext.Consumer>
    );
  };
}

function Button({theme, ...rest}) {
  return <button className={theme} {...rest} />;
}

const ThemedButton = withTheme(Button);
```


## Fragments
片段用来包裹多个子元素，不用像\<div\>那样添加额外的节点。

两种使用片段的方式：
1. <></>
2. <React.Fragment/>

<></> 是 <React.Fragment/> 的语法糖

Fragments 可以添加 key

```js
class Columns extends React.Component {
  render() {
    return (
      <>
        <td>Hello</td>
        <td>World</td>
      </>
    );
  }
}
```

通过使用一个数组让render()返回多个元素
```js
render() {
  return [
    <li key="A">First item</li>,
    <li key="B">Second item</li>,
    <li key="C">Third item</li>,
  ];
}
```

## Portals(入口)

通常 render 返回的元素会作为最近的父节点的子节点挂载到 DOM 上。

使用 Portals 可以将其插入到 DOM 节点的不同位置。
```js
render() {
  // React does *not* create a new div. It renders the children into `domNode`.
  // `domNode` is any valid DOM node, regardless of its location in the DOM.
  return ReactDOM.createPortal(
    this.props.children,
    domNode
  );
}
```
portals 多用于dialogs, hovercards, tooltips等。

使用父组件捕获从 portals 冒泡的事件可以使开发更灵活。

## 错误边界
如果一个组件定义了 componentDidCatch 生命周期，它会成为一个错误边界。

componentDidCatch 就像 JS 中的catch {}

一般来说错误边界组件只在应用中声明一次
```js
componentDidCatch(error, info) {
  logComponentStackToMyService(info.componentStack);
}
```
添加错误边界可以在出错时提供更好的用户体验

## 高阶组件

高阶组件是一个函数，它接收一个组件并返回一个新组件。

```js
function logProps(WrappedComponent) {
  return class extends React.Component {
    componentWillReceiveProps(nextProps) {
      console.log('Current props: ', this.props);
      console.log('Next props: ', nextProps);
    }
    render() {
      // Filter out extra props that are specific to this HOC and shouldn't be
      // passed through
      const { extraProp, ...passThroughProps } = this.props;
    
      // Inject props into the wrapped component. These are usually state values or
      // instance methods.
      const injectedProp = someStateOrInstanceMethod;
    
      // Pass props to wrapped component
      return (
        <WrappedComponent
          injectedProp={injectedProp}
          {...passThroughProps}
        />
      );
    }
  }
}
```

Hoc 将原始组件封装在容器组件中。Hoc 是一个具有零副作用的纯函数。

HOC最常见的签名：
```js
// React Redux's `connect`
const ConnectedComment = connect(commentSelector, commentActions)(CommentList);
```
connect 是一个返回高阶组件的高阶函数


## 和其他库集成

### jQuery
在 componentDidMount 获取 root DOM 的 ref，传递给 jQuery 插件。

为了避免React在mounting后操作DOM，我们可以在render()中返回一个空的\<div \/\>，这个\<div \/\>没有子元素，所以React不会去更新它。

如果你想在 React 流之外改变 DOM，重要的一点是要确保 React 接触不到这些 DOM 节点。

```js
class SomePlugin extends React.Component {
  componentDidMount() {
    this.$el = $(this.el);
    this.$el.somePlugin();
  }

  componentWillUnmount() {
    this.$el.somePlugin('destroy');
  }

  render() {
    return <div ref={el => this.el = el} />;
  }
}
```

## Render Props
组件中 render 作为一个 prop，赋予一个返回 React 元素的函数
```js
<DataProvider render={data => (
  <h1>Hello {data.target}</h1>
)}/>
```
render props 不单指有一个名为 render 的 prop，任何一个用来告诉组件渲染什么内容的 prop，在技术上都是 render props



## 严格模式
严格模式只在开发模式生效，不会影响生产。

启用严格模式：
```js
import React from 'react';

function ExampleApplication() {
  return (
    <div>
      <Header />
      <React.StrictMode>
        <div>
          <ComponentOne />
          <ComponentTwo />
        </div>
      </React.StrictMode>
      <Footer />
    </div>
  );
}
```
严格模式的作用：
1. 识别不安全声明周期的使用

    某些遗留的生命周期方法在异步React程序中是不安全的。

2. 使用遗留的string ref API时警告

    不要再使用 string ref API ，推荐使用 callback ref 或者 createRef。callback ref 相对 createRef 更灵活一些。

3. 检测意外副作用

    React工作分为两个阶段：

    render阶段决定需要对DOM进行什么改变。在此阶段，React 调用 render ，然后将结果与先前的渲染进行比较。  

    提交阶段指 React 应用这些更改。  
    React 会在这个阶段调用 componentDidMount 和 componentDidUpdate。  
    提交阶段通常非常快，render阶段可能很慢。  
    异步模式中，将render的工作分解为多个部分，暂停并恢复工作，以避免阻塞浏览器。这意味着 React 在提交之前可以不止一次地调用 render 阶段的生命周期。

    render 阶段的生命周期包括：
    ```
    constructor
    componentWillMount
    componentWillReceiveProps
    componentWillUpdate
    getDerivedStateFromProps
    shouldComponentUpdate
    render
    setState updater functions (the first argument)
    ```
    因为上述方法可能不止一次地被调用，所以它们不应该包含副作用。

4. 检测遗留的context API


## 可达性 Accessibility
保证你的web应用可以只使用键盘就能操作











