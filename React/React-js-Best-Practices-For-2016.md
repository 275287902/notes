
# React.js 最佳实践(2016)

翻译: Jason@Maxleap  
原文链接: https://blog.risingstack.com/react-js-best-practices-for-2016  
译者按：最近React(web/native)依旧如火如荼,相信大家都跃跃欲试,我们团队也开始在React领域有所尝试.  
2016年应该是 React 逐渐走向成熟的一年,让我们一起来看看国外的开发者们都总结了哪些"最佳实践".  
=============以下为译文==============  
2015年 React 在全世界都有很多关于新的更新和开发者大会的讨论.关于去年的重要事件,请参考 React in 2015.  
那么,2016年最有趣的问题来了:我们应该如何开发一个 App, 有什么推荐的库?  
作为一名长期使用 React.js 的开发者,我对此问题有自己的答案和最佳实践,但你可能不一定完全同意.我对你的想法和观点很有兴趣,请留言以便讨论.


![](https://risingstack-blog.s3.amazonaws.com/2016/Jan/react_best_practices-1453211146748.png)

如果你刚刚开始接触React.js,可以查看我们的 [React.js 教程](https://blog.risingstack.com/the-react-way-getting-started-tutorial/),或者 Pete Hunt 的 [React howto](https://github.com/petehunt/react-howto).


# 数据处理
在 React.js 应用中处理数据非常简单,但也充满挑战.    
这是因为你可以使用多种方式将属性数据传递给 React 组件,从而构建出渲染树,但你应该怎样更新视图却不是显而易见的.  
2015一开始便诞生了很多不同 Flux 库,随后涌现出出更多具有更强功能和更加响应式解决方案.  
让我们一起来看看：  

## Flux
根据我们的经验,Flux 经常被过度使用,(就是大家总是在不需要它的时候仍然用了它).  
Flux 提供了一种非常清晰的方式来存储和更新**App 全局 state**(译者注：对应 react 中的 state),并在需要的时候触发渲染.   
Flux 在管理App的全局状态时很有用,比如：管理已登录用户状态,路由状态,或者是活跃账号状态,但若是用来管理临时数据或者本地数据,立刻就会变得很痛苦.  
我们不推荐使用 Flux 来管理路由相关的数据,比如 /items/:itemId.获取路由数据并存储在组件的 state 之中.这种情况下,它会在组件销毁时一起被销毁.  
如果你想了解更多关于 Flux 的信息,建议阅读 [The Evolution of Flux Frameworks](https://medium.com/@dan_abramov/the-evolution-of-flux-frameworks-6c16ad26bb31#.90lamiv5l).

## 使用 Redux
Redux 是一个 JavaScript App的可预测 state 容器.   
如果你觉得需要 Flux 或者相似的解决方案,你应该了解一下 redux,并学习 [Dan Abramov](https://twitter.com/dan_abramov) 的 [Getting started with redux](https://egghead.io/series/getting-started-with-redux),这能够迅速提高你的开发技能.   
Redux 延续并改进了 Flux 的思想,学习了 Elm ,避开了 Flux 的复杂度(译者注：[Elm](http://elm-lang.org) 是一门函数式编程语言).  

### 扁平化 state
API 经常会返回嵌套的资源.这在 Flux 或基于 Redux 的架构中处理起来会非常困难.我们推荐使用 [normalizr](https://github.com/gaearon/normalizr) 这类库将数据进行扁平化处理,尽可能地扁平化state.  
像这样：  
```
const data = normalize(response, arrayOf(schema.user))
state = _.merge(state, data.entities)
```  
(我们使用[isomorphic-fetch](https://www.npmjs.com/package/isomorphic-fetch)与API进行通信)  

### 使用 immutable state
共享的可变性 state 是罪恶的根源. - Pete Hunt, React.js Conf 2015  
![](https://risingstack-blog.s3.amazonaws.com/2016/Jan/immutable_logo_for_react_js_best_practices-1453211749818.png)  
[不可变对象](https://en.wikipedia.org/wiki/Immutable_object)是指在创建后不可再被修改的对象.   
不可变对象可以让我们免于痛苦,并且通过引用级的比对检查来**提升渲染性能**.比如在 ```shouldComponentUpdate``` 中：  
```
shouldComponentUpdate(nexProps) {
 // 不进行对象的深度对比
 return this.props.immutableFoo !== nexProps.immutableFoo
}
```

### 如何在JavaScript中实现不可变?
本办法是小心的写代码,示例代码如下,你需要在单元测试中通过 [deep-freeze-node](https://www.npmjs.com/package/deep-freeze-node) 来反复验证.
```
return {  
  ...state,
  foo
}
 
return arr1.concat(arr2)
```
相信我,这是最明显的例子了.  
更简单也更自然的方式是使用 Immutable.js.
```
import { fromJS } from 'immutable'

const state = fromJS({ bar: 'biz' })  
const newState = foo.set('bar', 'baz') 
```
Immutable.js 非常之快,背后理念也非常美妙.哪怕你并不准备使用它,我也推荐阅读这个由 [Lee Byron](https://twitter.com/leeb) 所制作的视频 [Immutable Data and React](https://www.youtube.com/watch?v=I7IdS-PbEgI).它非常深刻的讲解了 Immutable.js 的工作原理. 

### Observables and Reactive 解决方案
如果你不喜欢 Flux/Redux 或者只是想要更加   reactive,不用失望!还有很多数据处理的方案供你选择,这里有一个可能是你想要的库的简单列表：
* cycle.js(“一个更清晰简洁的函数式 reactive JavaScript 框架”)
* rx-flux(“Flux 架构与 Rxjs 的结合”)
* redux-rx(“Redux的 Rxjs 工具集”)
* mobservable(“可预测的数据,reactive的功能,简洁的代码”)


# 路由
几乎所有 App 都有路由功能.如果你在浏览器中使用 React.js,你将会在挑选库的时候遇到选择性问题.    
我们的选择是react-router, 来自优秀的 rackt 社区.Racket 给 React.js 爱好者们带来了很多高质量资源.   
要使用 react-router,请查看它的文档.但更重要的是：如果你使用Flux/Redux,我们推荐你将路由 state 与 store 或全局 state 保持同步.    
同步的路由 state 会帮助你控制 Flux/Redux Actions 的路由行为,并能在组件中读取路由状态和参数.   
Redux 用户可以通过 [redux-simple-router](https://github.com/rackt/redux-simple-router) 这个库轻松实现它.

## 代码分割,惰性加载
只有一小部分 ```webpack``` 用户知道 App 代码可以分割成多个 JavaScript 块. 
```
require.ensure([], () => {  
  const Profile = require('./Profile.js')
  this.setState({
    currentComponent: Profile
  })
})
```
这对于大型应用十分有用,每次部署之后用户浏览器不用下载那些```很少会使用到的代码```,比如Profile页面. 
更多代码块将导致更多 HTTP 请求 - 但是使用 HTTP/2 多路复用就没有问题.  
结合 [chunk hashing](https://christianalfoni.github.io/react-webpack-cookbook/Optimizing-caching.html),可以在代码更新之后优化缓存命中率.   
下个版本的 react-router 将会对代码分隔做更多支持.  
对于 react-router 的未来规划,可以去查看博客 [Ryan Florence](https://twitter.com/ryanflorence): [Welcome to Future of Web Application Delivery](https://medium.com/@ryanflorence/welcome-to-future-of-web-application-delivery-9750b7564d9f#.vuf3e1nqi).  

# 组件
很多人都在抱怨JSX,但首先要知道,它在 React 中是可选的.   
JSX 在最后都会通过 Babel 被编译成 JavaScript.你可以直接编写 JavaScript 来替代 JSX,但是在处理 HTML 的时候使用 JSX 会感觉更加自然.  
特别是对于不懂技术的人来说,他们只可以理解和修改必要的部分.  

JSX 是一种与 XML 类似的 JavaScript 语法扩展.你可以通过一个简单的 JSX 语法转换器来转换它.— [JSX in depth](https://facebook.github.io/react/docs/jsx-in-depth.html)  

如果你想了解更多 JSX 的内容,查看文章 [JSX Looks Like An Abomination – But it’s Good for You](https://medium.com/javascript-scene/jsx-looks-like-an-abomination-1c1ec351a918#.ca28nvee6)  



## 使用 Classes
React 与 ES2015 的 Class 语法搭配的很好. 
```
class HelloMessage extends React.Component {  
  render() {
    return <div>Hello {this.props.name}</div>
  }
}
```
相对于mixins,我们更喜欢[高阶组件](http://jamesknelson.com/structuring-react-applications-higher-order-components/),所以保留 createClass 更像是一个语法问题,而不是技术问题. 我们认为使用 createClass  或者 React.Component 只是选择不同而已,没有对错之分.
##属性类型 
如果你仍然没有检查 熟悉类型,那么你应该从2016年开始做起,这将为你节省大量的时间,相信我.
```
MyComponent.propTypes = {  
  isLoading: PropTypes.bool.isRequired,
  items: ImmutablePropTypes.listOf(
    ImmutablePropTypes.contains({
      name: PropTypes.string.isRequired,
    })
  ).isRequired
}
```
当然,也可以使用 [react-immutable-proptypes](https://www.npmjs.com/package/react-immutable-proptypes) 验证 Immutable.js 所编写的属性.

# 高阶组件
当前 mixins 将死,而且在 ES6 的 Class 不再支持 mixins,我们应当寻找新方案.
## 什么是高阶组件？
```
PassData({ foo: 'bar' })(MyComponent)
```
简单来讲,从由原始组件创造一个新的组件并且扩展它的行为.你可以在多种场景来使用它,比如鉴权：```requireAuth({ role: 'admin' })(MyComponent)```(检查用户权限,如果未登录就跳转),或者将组件与 Flux/Redux 的 store 连通.   
在 RisingStack,我们也喜欢将数据拉取和控制类的逻辑分离到高阶组件中,以尽可能地保持 view 层的简单. 

# 测试
保证测试的高代码覆盖率是开发周期中的重要一环.幸运的是,React.js 社区有很多这样的库来帮助我们.
## 组件测试
AirBnb 的 [enzyme](https://github.com/airbnb/enzyme) 是我们最喜爱的组件测试库之一.使用它的浅渲染特性可以对组件的逻辑和渲染结果进行测试,非常神奇.它现在还不能替代selenium测试,但是将前端测试提升到了一个新高度.
```
it('simulates click events', () => {  
  const onButtonClick = sinon.spy()
  const wrapper = shallow(
    <Foo onButtonClick={onButtonClick} />
  )
  wrapper.find('button').simulate('click')
  expect(onButtonClick.calledOnce).to.be.true
})
```
看起来非常简洁,不是吗？  
你使用 chai 作为测试断言库嘛？相信你会喜欢 [chai-enyzime](https://github.com/producthunt/chai-enzyme) 的!
## Redux测试
**测试 reducer** 非常简单,它响应新到来的 actions 然后将原来的 state 转换为新的 state：

```
it('should set token', () => {  
  const nextState = reducer(undefined, {
    type: USER_SET_TOKEN,
    token: 'my-token'
  })
 
  // immutable.js state output
  expect(nextState.toJS()).to.be.eql({
    token: 'my-token'
  })
})
```
**测试 actions** 也很简单,但是异步 actions 就不太一样了.对于测试异步的 actions 来说,我们推荐使用  [redux-mock-store](https://www.npmjs.com/package/redux-mock-store),非常有帮助. 
```
it('should dispatch action', (done) => {  
  const getState = {}
  const action = { type: 'ADD_TODO' }
  const expectedActions = [action]

  const store = mockStore(getState, expectedActions, done)
  store.dispatch(action)
})
```
关于更深入的 [redux测试](http://rackt.org/redux/docs/recipes/WritingTests.html) ,请参考官方文档. 


# 使用 npm
虽然 React.js 并不依赖代码打包工具就可以工作得很好,但我们还是推荐使用 Webpack 或者 Browserify 来发挥 npm 的能力.Npm 有很多 React.js 的包,可以帮助你优雅地管理依赖.   
(请不要忘记复用你自己的组件,这是优化代码的绝佳方式.)  

## Bundle 大小
这本身不是一个 React 相关的问题,但是大多数人都在打包他们的 React 应用,所以我有必要在这里提一下.   
当你打包源代码的时候,要时刻警惕打包后文件的大小.想要**将其控制在最小体积**,你需要思考如何如何 require/import 依赖.   
查看下面的代码片段,这两种方式可以对输出大小会产生重大影响：
```
import { concat, sortBy, map, sample } from 'lodash'
 
// vs.
import concat from 'lodash/concat';  
import sortBy from 'lodash/sortBy';  
import map from 'lodash/map';  
import sample from 'lodash/sample';  
```
查看[Reduce Your bundle.js File Size By Doing This One Thing](https://lacke.mn/reduce-your-bundle-js-file-size/),以获取更多信息.   
我们也喜欢将代码分离到至少 vendors.js 和 app.js 两个文件,因为 vendors 相对于我们的代码库来说更新频率低很多.   
对输出文件进行 hash 命名(WebPack中的chunk hash),并使用长缓存,我们可以显著地减少访问用户需要下载的代码.结合代码懒加载,优化效果非常显著.  
如果你还不太熟悉 Webpack,可以查看这本优秀的 [React webpack cookbook](https://christianalfoni.github.io/react-webpack-cookbook). 
## 组件级别的 hot reload
如果你曾使用过hot reload编写单页面应用,当你在处理某些与状态相关的事情时,可能你就会明白当你在编辑器中点击保存,整个页面就重新加载了是多么令人讨厌.你需要逐步点击操作到刚才的环节,然后在这样的重复中奔溃.   
通过 React,在重载组件的同时保持组件状态已经成为可能,从此不再痛苦!  
关于如何搭建hot reload,可参考 [react-transform-boilerplate](https://github.com/gaearon/react-transform-boilerplate)  
# 使用ES2015
前面有提到过,我们在 React.js 组件中使用 JSX,然后使用 Babel.js 进行编译.  
 Babel 的能力远不止这些,它也可以让我们现在就可以给浏览器编写 ES6/ES2015 代码.在RisingStack,我们在服务器端和客户端都使用了ES2015的特性,ES2015已经可以在最新的LTS Node.js版本中使用了. 
# 代码检查
或许你已经给你的 JavaScript 代码制定了代码规范,但是你知道也有用于 React 的代码规范了吗？我们建议你选择一个代码规范,然后照着它说的来做.
在 RisingStack,我们也将 linters 强制运行在 CI 系统上,```git push``` 亦然.可以试试 ```pre-push``` 或者 ```pre-commit```. 
我们使用标准的 JavaScript 代码风格,并使用 [eslint-plugin-react](https://www.npmjs.com/package/eslint-plugin-react) 来检查React.js代码. 
(是的,我们已经不再使用分号了)
# GraphQL和Relay
相对而言 GraphQL 和 Relay 还属于新技术,在 RisingStack,我们还没有在产品环境中使用它们,但保持关注. 
我们写过一个 Relay 的 MongoDB ORM 库,叫做 graffiti,可以使用你已有的 mongoose models 来创建 GraphQL server. 
如果你想要学习这些新技术,我们建议你可以找来玩一玩. 
# 尽情享用这些 React.js 的最佳实践
有些优秀的技术和库其实跟React都几乎没关系,但要关注社区的其他人都在做些什么.2015这一年,React社区被 [Elm 架构](https://github.com/evancz/elm-architecture-tutorial/)启发了很多.    
如果你知道其它在2016年必要的 React.js 工具,请在下面给我们留言!


