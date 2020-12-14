## Ref
### 使用场景
- 管理焦点，文本选择或媒体播放。
- 触发强制动画。
- 集成第三方 DOM 库。
### 创建 Refs
Refs 是使用 React.createRef() 创建的，并通过 ref 属性附加到 React 元素。* 在构造组件时 *，通常将 Refs 分配给实例属性，以便可以在整个组件中引用它们。

为 DOM 元素添加 ref：
```
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
为 class 组件添加 Ref：
```
// 功能：模拟组件挂载之后立即被点击的操作
class CustomTextInput extends React.Component {
  constructor(props) {
    super(props);
    // 创建一个 ref 来存储 textInput 的 DOM 元素
    this.textInput = React.createRef();
    this.focusTextInput = this.focusTextInput.bind(this);
  }

  focusTextInput() {
    // 直接使用原生 API 使 text 输入框获得焦点
    // 注意：我们通过 "current" 来访问 DOM 节点
    this.textInput.current.focus();
  }

  render() {
    // 告诉 React 我们想把 <input> ref 关联到
    // 构造器里创建的 `textInput` 上
    return (
      <div>
        <input
          type="text"
          ref={this.textInput} />

        <input
          type="button"
          value="Focus the text input"
          onClick={this.focusTextInput}
        />
      </div>
    );
  }
}



class AutoFocusTextInput extends React.Component {
  constructor(props) {
    super(props);
    this.textInput = React.createRef();
  }

  componentDidMount() {
    this.textInput.current.focusTextInput();
  }

  render() {
    return (
      <CustomTextInput ref={this.textInput} />
    );
  }
}
```
### 访问 Refs
```
const node = this.myRef.current;
```
-----
## Ref 转发
### 使用场景
高度复用组件，组件倾向于像 DOM 组件一样被使用（访问 DOM 节点对管理焦点，选中或动画来说是不可避免的。）
### 作用
组件可以像暴露自己的 ref 一样暴露子组件的 ref。
### 原理
使用 ```React.forwardRef((props, ref) => (...))``` 生成的组件，将 ```const transRef = React.createRef()```生成的 ref（即 transRef） 传递给组件内的 DOM 元素。然后通过 transRef.current 获得 DOM 元素的引用（即此DOM元素的 ref）；

### NOTE
	1. 第二个参数 ref 只在使用 React.forwardRef 定义组件时存在。常规函数和 class 组件不接收 ref 参数，且 props 中也不存在 ref。
	2. Ref 转发不仅限于 DOM 组件，你也可以转发 refs 到 class 组件实例中。

### 例子：
```
const FancyButton = React.forwardRef((props, ref) => (
  <button ref={ref} className="FancyButton">
    {props.children}
  </button>
));

// 你可以直接通过 ref.current 获取 DOM button 的 ref：
const ref = React.createRef();
<FancyButton ref={ref}>Click me!</FancyButton>;
```
