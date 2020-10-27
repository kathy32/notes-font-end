[参考](https://juejin.im/post/6844903870229905422)
# 目录
#### 1. 是什么
#### 2. 从 h 函数说起
1. 安装 babel 依赖
2. 配置 .babelrc
3. 转译 jsx

#### 3. 渲染虚拟 DOM

#### 4.  算法
1. diff 算法的进化
2. VDOM
- DOM 树的遍历
- VDOM 节点的对比
- 属性的对比
- 子节点的对比
- 更新 DOM
3. cito.js
4. kivi.js
- 算法思想
- 关于 kivi
5. snabbdom

####  总结

---

### 1. 是什么？
虚拟 DOM （Virtual DOM ）这个概念相信大家都不陌生，从 React 到 Vue ，虚拟 DOM 为这两个框架都带来了==跨平台的能力==（React-Native 和 Weex）。因为很多人是在学习 React 的过程中接触到的虚拟 DOM ，所以为先入为主，认为虚拟 DOM 和 JSX 密不可分。其实不然，虚拟 DOM 和 ==JSX== 固然契合，但 JSX 只是虚拟 DOM 的充分不必要条件，Vue 即使使用模版，也能把虚拟 DOM 玩得风生水起，同时也有很多人通过 babel 在 Vue 中使用 JSX。


很多人认为虚拟 DOM 最大的优势是 ==diff== 算法，减少 JavaScript 操作真实 DOM 的带来的性能消耗。虽然这一个虚拟 DOM 带来的一个优势，但并不是全部。虚拟 DOM ==最大的优势在于抽象了原本的渲染过程==，实现了==跨平台==的能力，而不仅仅局限于浏览器的 DOM，可以是安卓和 IOS 的原生组件，可以是近期很火热的==小程序==，也可以是各种GUI。

回到最开始的问题，虚拟 DOM 到底是什么，说简单点，就是一个普通的 ==JavaScript 对象==，包含了 三个属性:
- **tag**、
- **props**、
- **children**

```html
<div id="app">
  <p class="text">hello world!!!</p>
</div>

```
上面的 HTML 转换为虚拟 DOM 如下：

```js
{
  tag: 'div',
  
  props: {
    id: 'app'
  },
  
  chidren: [
    {
      tag: 'p',
      props: {
        className: 'text'
      },
      chidren: [
        'hello world!!!'
      ]
    }
  ]
}

```
该对象就是我们常说的虚拟 DOM 了，因为 DOM 是树形结构，所以使用 JavaScript 对象就能很简单的表示。而原生 DOM 因为浏览器厂商需要实现众多的规范（各种 HTML5 属性、DOM事件），即使创建一个空的 div 也要付出昂贵的代价。

虚拟 DOM 提升性能的点在于 DOM 发生变化的时候，通过 ==diff== 算法比对 JavaScript 原生对象，计算出需要变更的 DOM，然后只对变化的 DOM 进行操作，而==不是更新整个视图==。
那么我们到底该如何将一段 HTML 转换为虚拟 DOM 呢？


### 2. 从 h 函数说起
观察主流的虚拟 DOM 库（snabbdom、virtual-dom），通常都有一个 ==h== 函数，也就是 React 中的 React.createElement，以及 Vue 中的 ==render== 方法中的 ==createElement==，

另外 React 是通过 babel 将 jsx 转换为 h 函数渲染的形式，而 Vue 是使用 ==vue-loader== 将模版转为 h 函数渲染的形式（也可以通过 babel-plugin-transform-vue-jsx 插件在 vue 中使用 jsx，本质还是转换为 h 函数渲染形式）。

我们先使用 babel，将一段 jsx 代码，转换为一段 js 代码：




#### 2.1安装 babel 依赖

```shell
npm i -D @babel/cli @babel/core @babel/plugin-transform-react-jsx

```


#### 2.2 配置 .babelrc

```
{
    "plugins": [
        [
            "@babel/plugin-transform-react-jsx",
            {
                "pragma": "h", // default pragma is React.createElement
            }
        ]
    ]
}

```


#### 2.3 转译 jsx

在目录下新建一个 main.jsx

```js
function getVDOM() {
  return (
    <div id="app">
      <p className="text">hello world!!!</p>
    </div>
  )
}
```

使用如下命令进行转译：

```shell
npx babel main.jsx --out-file main-compiled.js

```
可以看到，最终 HTML 代码会被转译成 h 函数的渲染形式.==h 函数接受是三个参数，分别代表是 DOM 元素的标签名、属性、子节点==，最终返回一个虚拟 DOM 的对象。

```js
function h(tag, props, ...children) {
  return {
    tag,
    props: props || {},
    children: children.flat()
  }
}

```


#### 3. 渲染虚拟 DOM
虽然虚拟 DOM 可以渲染到多个平台，但是这里讲一下在浏览器环境下如何渲染虚拟 DOM。

```js
function render(vdom) {

  // 如果是字符串或者数字，创建一个文本节点
  if (typeof vdom === 'string' || typeof vdom === 'number') {
    return document.createTextNode(vdom)
  }
  
  const { tag, props, children } = vdom
  
  // 创建真实DOM
  const element = document.createElement(tag)
  
  // 设置属性
  setProps(element, props)
  
  // 遍历子节点，并获取创建真实DOM，插入到当前节点
  children
    .map(render)
    .forEach(element.appendChild.bind(element))

  // 虚拟 DOM 中缓存真实 DOM 节点
  vdom.dom = element
  
  // 返回 DOM 节点
  return element
}

function setProps (element, props) {
  Object.entries(props).forEach(([key, value]) => {
    setProp(element, key, value)
  })
}

function setProp (element, key, vlaue) {
  element.setAttribute(
    // className使用class代替
    key === 'className' ? 'class' : key,
    vlaue
  )
}

```
将虚拟 DOM 渲染成真实 DOM 后，只需要插入到对应的根节点即可。

```js
const vdom = <div>hello world!!!</div> // h('div', {}, 'hello world!!!')
const app = document.getElementById('app')
const ele = render(vdom)
app.appendChild(ele)

```
当然在现代化的框架中，一般会有一个组件文件专门用来构造虚拟 DOM，我们模仿 React 使用 class 的方式编写组件，然后渲染到页面中。

```js
class Component {
  vdom = null // 组件的虚拟DOM表示
  $el  = null // 虚拟DOM生成的真实节点

  state = {
    text: 'Initialize the Component'
  }
  
  render() {
    const { text } = this.state
    return (
      <div>{ text }</div>
    )
  }
}

function createElement (app, component) {
  const vdom = component.render()
  component.vdom = vdom
  component.$el = render(vdom) // 将虚拟 DOM 转换为真实 DOM
  app.appendChild(component.$el)
}

const app = document.getElementById('app')
const component = new Component
createElement(app, component)

```




#### 4. diff 算法
1. diff 算法的进化
2. VDOM
- DOM 树的遍历
- VDOM 节点的对比
- 属性的对比
- 子节点的对比
- 更新 DOM
3. cito.js
4. kivi.js
- 算法思想
- 关于 kivi
5. snabbdom

#### 3. 总结

