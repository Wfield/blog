1. hooks 不能在循环、条件判断、组件内的函数中使用（useEffects 与生命周期类似）；只能在函数组件下和自定义 hooks 中使用；
2. 可以声明多个 state，state 只在组件首次渲染的时候被创建。在下一次重新渲染时，useState 返回给我们当前的 state（React 会在重复渲染时保留这个 state）；
3. state 在函数退出后可以保留下来
> 这是一种在函数调用时保存变量的方式 —— useState 是一种新方法，它与 class 里面的 this.state 提供的功能完全相同。一般来说，在函数退出后变量就会”消失”，而 state 中的变量会被 React 保留
4. hooks 的 state 是完全替换以前的 state，而不是合并；
5. 每次重新渲染，都会生成新的 effect，替换掉之前的
6. 当 React 渲染组件时，会保存已使用的 effect，并在更新完 DOM 后执行它
7. effect 的清除( return )阶段在每次重新渲染时都会执行，而不是只在卸载组件的时候执行一次

## TODO
- 需要学习的 Hooks
1. useWhatChanged
2. [useWhyDidYouUpdate](https://usehooks.com/useWhyDidYouUpdate/)
3. why-did-you-render

- 函数退出与组件卸载是不是一回事？
> **React 何时清除 effect？** React 会在组件卸载的时候执行清除操作。正如之前学到的，effect 在每次渲染的时候都会执行。这就是为什么 React *会*在执行当前 effect 之前对上一个 effect 进行清除。我们稍后将讨论[为什么这将助于避免 bug](https://zh-hans.reactjs.org/docs/hooks-effect.html#explanation-why-effects-run-on-each-update)以及[如何在遇到性能问题时跳过此行为](https://zh-hans.reactjs.org/docs/hooks-effect.html#tip-optimizing-performance-by-skipping-effects)。
