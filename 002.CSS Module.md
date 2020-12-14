CSS Modules 能最大化地结合现有 CSS 生态和 JS 模块化能力，API 简洁到几乎零学习成本。发布时依旧编译出单独的 JS 和 CSS。它并不依赖于 React，只要你使用 Webpack，可以在 Vue/Angular/jQuery 中使用。

CSS Modules 内部通过 ICSS 来解决样式导入和导出这两个问题。分别对应 :import 和 :export 两个新增的伪类。
```
:import("path/to/dep.css") {
  localAlias: keyFromDep;
  /* ... */
}
:export {
  exportedKey: exportedValue;
  /* ... */
}
```
但直接使用这两个关键字编程太麻烦，实际项目中很少会直接使用它们，我们需要的是用 JS 来管理 CSS 的能力。结合 Webpack 的 css-loader 后，就可以在 CSS 中定义样式，在 JS 中导入。

启用 CSS Modules

// webpack.config.js
css?modules&localIdentName=[name]__[local]-[hash:base64:5]
加上 modules 即为启用，localIdentName 是设置生成样式的命名规则。
也可以这样设置：

         test: /\.less$/,
         use: [
                'style-loader',
                {
                    loader: 'css-loader',
                    options: {
                        modules: true,
                        localIdentName: '[name]__[local]-[hash:base64:5]',
                    },
                },
               ],
        },
样式文件Button.css：
```
.normal { /* normal 相关的所有样式 */ }
.disabled { /* disabled 相关的所有样式 */ }
/* components/Button.js */
import styles from './Button.css';

console.log(styles);

buttonElem.outerHTML = `<button class=${styles.normal}>Submit</button>`
```
生成的 HTML 是

`<button class="button--normal-abc53">Submit</button>`
注意到 button--normal-abc53 是 CSS Modules 按照 localIdentName 自动生成的 class 名。其中的 abc53 是按照给定算法生成的序列码。经过这样混淆处理后，class 名基本就是唯一的，大大降低了项目中样式覆盖的几率。同时在生产环境下修改规则，生成更短的class名，可以提高CSS的压缩率。

上例中 console 打印的结果是：
```
Object {
  normal: 'button--normal-abc53',
  disabled: 'button--disabled-def884',
}
```
CSS Modules 对 CSS 中的 class 名都做了处理，使用对象来保存原 class 和混淆后 class 的对应关系。

通过这些简单的处理，CSS Modules 实现了以下几点：

所有样式都是 local 的，解决了命名冲突和全局污染问题
class 名生成规则配置灵活，可以此来压缩 class 名
只需引用组件的 JS 就能搞定组件所有的 JS 和 CSS
依然是 CSS，几乎 0 学习成本
样式默认局部
使用了 CSS Modules 后，就相当于给每个 class 名外加加了一个 :local，以此来实现样式的局部化，如果你想切换到全局模式，使用对应的 :global。
```
.normal {
  color: green;
}

/* 以上与下面等价 */
:local(.normal) {
  color: green; 
}

/* 定义全局样式 */
:global(.btn) {
  color: red;
}

/* 定义多个全局样式 */
:global {
  .link {
    color: green;
  }
  .box {
    color: yellow;
  }
}
Compose 来组合样式
对于样式复用，CSS Modules 只提供了唯一的方式来处理：composes 组合

/* components/Button.css */
.base { /* 所有通用的样式 */ }

.normal {
  composes: base;
  /* normal 其它样式 */
}

.disabled {
  composes: base;
  /* disabled 其它样式 */
}
```
```
import styles from './Button.css';

buttonElem.outerHTML = `<button class=${styles.normal}>Submit</button>`
```
生成的 HTML 变为

`<button class="button--base-daf62 button--normal-abc53">Submit</button>`
由于在 .normal 中 composes 了 .base，编译后会 normal 会变成两个 class。

composes 还可以组合外部文件中的样式。
```
/* settings.css */
.primary-color {
  color: #f40;
}

/* components/Button.css */
.base { /* 所有通用的样式 */ }

.primary {
  composes: base;
  composes: primary-color from './settings.css';
  /* primary 其它样式 */
}
```
对于大多数项目，有了 composes 后已经不再需要 Sass/Less/PostCSS。但如果你想用的话，由于 composes 不是标准的 CSS 语法，编译时会报错。就只能使用预处理器自己的语法来做样式复用了。

class 命名技巧
CSS Modules 的命名规范是从 BEM 扩展而来。

clipboard.png

BEM 把样式名分为 3 个级别，分别是：

Block：对应模块名，如 Dialog
Element：对应模块中的节点名 Confirm Button
Modifier：对应节点相关的状态，如 disabled、highlight
综上，BEM 最终得到的 class 名为 dialog__confirm-button--highlight。使用双符号 __ 和 -- 是为了和区块内单词间的分隔符区分开来。虽然看起来有点奇怪，但 BEM 被非常多的大型项目和团队采用。我们实践下来也很认可这种命名方法。

CSS Modules 中 CSS 文件名恰好对应 Block 名，只需要再考虑 Element 和 Modifier。BEM 对应到 CSS Modules 的做法是：
```
/* .dialog.css */
.ConfirmButton--disabled {
}
```
你也可以不遵循完整的命名规范，使用 camelCase 的写法把 Block 和 Modifier 放到一起：
```
/* .dialog.css */
.disabledConfirmButton {
}
```
模块化命名实践:MBC 【仅供参考参考】
M:module 模块（组件）名
B:block 元素块的功能说明
C:custom 自定义内容
如何实现CSS，JS变量共享
注：CSS Modules 中没有变量的概念，这里的 CSS 变量指的是 Sass 中的变量。
上面提到的 :export 关键字可以把 CSS 中的 变量输出到 JS 中。下面演示如何在 JS 中读取 Sass 变量：
```
/* config.scss */
$primary-color: #f40;

:export {
  primaryColor: $primary-color;
}
/* app.js */
import style from 'config.scss';

// 会输出 #F40
console.log(style.primaryColor);
```
CSS Modules 使用技巧
CSS Modules 是对现有的 CSS 做减法。为了追求简单可控，作者建议遵循如下原则：

不使用选择器，只使用 class 名来定义样式
不层叠多个 class，只使用一个 class 把所有样式定义好
所有样式通过 composes 组合来实现复用
不嵌套
上面两条原则相当于削弱了样式中最灵活的部分，初使用者很难接受。第一条实践起来难度不大，但第二条如果模块状态过多时，class 数量将成倍上升。
一定要知道，上面之所以称为建议，是因为 CSS Modules 并不强制你一定要这么做。听起来有些矛盾，由于多数 CSS 项目存在深厚的历史遗留问题，过多的限制就意味着增加迁移成本和与外部合作的成本。初期使用中肯定需要一些折衷。幸运的是，CSS Modules 这点做的很好：

如果我对一个元素使用多个 class 呢？

没问题，样式照样生效。
如何我在一个 style 文件中使用同名 class 呢？

没问题，这些同名 class 编译后虽然可能是随机码，但仍是同名的。
如果我在 style 文件中使用伪类，标签选择器等呢？

没问题，所有这些选择器将不被转换，原封不动的出现在编译后的 css 中。也就是说 CSS Modules 只会转换 class 名和 id 选择器名相关的样式。
但注意，上面 3 个“如果”尽量不要发生。

CSS Modules 结合 React 实践
首先，在 CSS loader中开启CSS Module：
```
  {
        loader: 'css-loader',
        options: {
          modules: true,
          localIdentName: '[local]',
        },
  },
```
在 className 处直接使用 css 中 class 名即可。
```
/* dialog.css */
.root {}
.confirm {}
.disabledConfirm {}
import classNames from 'classnames';
import styles from './dialog.css';

export default class Dialog extends React.Component {
  render() {
    const cx = classNames({
      [styles.confirm]: !this.state.disabled,
      [styles.disabledConfirm]: this.state.disabled
    });

    return <div className={styles.root}>
      <a className={cx}>Confirm</a>
      ...
    </div>
  }
}
```
注意，一般把组件最外层节点对应的 class 名称为 root。这里使用了 classnames 库来操作 class 名。
如果你不想频繁的输入 styles.xx，可以试一下 react-css-modules，它通过高阶函数的形式来避免重复输入 styles.xx。

React 中使用CSS Module,可以设置
5.几点注意

1.多个class的情况

// 可以使用字符串拼接
`className = {style.oneclass+' '+style.twoclass}`

//可以使用es6的字符串模板
`className = {`${style['calculator']} ${style['calculator']}`} `
2.如果class使用的是连字符可以使用数组方式style['box-text']

3.一个class是父组件传下来的
如果一个class是父组件传下来的，在父组件已经使用style转变过了，在子组件中就不需要再style转变一次，例子如下:
```
//父组件render中
 <CalculatorKey className={style['key-0']}   onPress={() => this.inputDigit(0)}>0</CalculatorKey>

//CalculatorKey 组件render中
 const { onPress, className, ...props } = this.props;
        return (
            <PointTarget onPoint={onPress}>
                <button className={`${style['calculator-key']} ${className}`} {...props}/>
            </PointTarget>
        )
子组件CalculatorKey接收到的className已经在父组件中style转变过了
```

4.显示undefined
如果一个class你没有在样式文件中定义，那么最后显示的就是undefined，而不是一个style过后的没有任何样式的class, 这点很奇怪。

6.从Vue 来看CSS 管理方案的发展
```
/* HTML */
<template>
  <h1 @click="clickHandler">{{ msg }}</h1>
</template>

/* script */
<script>
module.exports = {
  data: function() {
    return {
      msg: 'Hello, world!'
    }
  },
  methods:{
    clickHandler(){
      alert('hi');
    }
  }
}
</script>

/* scoped CSS */
<style scoped>
h1 {
  color: red;
  font-size: 46px;
}
</style>
```
在Vue 元件档，透过上面这样的方式提供了一个template (HTML)、script 以及带有scoped 的style 样式，也仍然可以保有过去HTML、CSS 与JS 分离的开发体验。但本质上仍是all-in-JS 的变种语法糖。

值得一提的是，当style标签加上scoped属性，那么在Vue元件档经过编译后，会自动在元件上打上一个随机的属性 data-v-hash，再透过CSS Attribute Selector的特性来做到CSS scope的切分，使得即便是在不同元件档里的h1也能有CSS样式不互相干扰的效果。当然开发起来，比起JSX、或是inline-style等的方式，这种折衷的作法更漂亮。
