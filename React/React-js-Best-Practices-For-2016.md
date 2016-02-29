Jason@Maxleap  
译者按：最近React(web/native)依旧如火如荼,相信大家都跃跃欲试,我们团队也开始在React领域有所尝试.  
2016年应该是 React 逐渐走向成熟的一年,让我们一起来看看国外的开发者们都总结了哪些"最佳实践".  
=============以下为译文==============  
2015年 React 在全世界都有很多关于新的更新和开发者大会的讨论.关于去年的重要事件,请参考 React in 2015.  
那么,2016年最有趣的问题来了:我们应该如何开发一个 App, 有什么推荐的库?  
作为一名长期使用 React.js 的开发者,我对此问题有自己的答案和最佳实践,但你可能不一定完全同意.我对你的想法和观点很有兴趣,请留言以便讨论.


![](https://risingstack-blog.s3.amazonaws.com/2016/Jan/react_best_practices-1453211146748.png)

如果你刚刚开始接触React.js，可以查看我们的 [React.js 教程](https://blog.risingstack.com/the-react-way-getting-started-tutorial/)，或者 Pete Hunt 的 [React howto](https://github.com/petehunt/react-howto).


# 数据处理
在 React.js 应用中处理数据非常简单，但也充满挑战.    
这是因为你可以使用多种方式将属性数据传递给 React 组件，从而构建出渲染树,但你应该怎样更新视图却不是显而易见的.  
2015一开始便诞生了很多不同 Flux 库，随后涌现出出更多具有更强功能和更加响应式解决方案.  
让我们一起来看看：  

## Flux
根据我们的经验，Flux 经常被过度使用，（就是大家总是在不需要它的时候仍然用了它）.  
Flux 提供了一种非常清晰的方式来存储和更新**App 全局 state**（译者注：对应 react 中的 state），并在需要的时候触发渲染。  
Flux 在管理App的全局状态时很有用，比如：管理已登录用户状态，路由状态，或者是活跃账号状态，但若是用来管理临时数据或者本地数据，立刻就会变得很痛苦。  
我们不推荐使用 Flux 来管理路由相关的数据，比如 /items/:itemId。获取路由数据并存储在组件的 state 之中。这种情况下，它会在组件销毁时一起被销毁。
如果你想了解更多关于 Flux 的信息，建议阅读 [The Evolution of Flux Frameworks](https://medium.com/@dan_abramov/the-evolution-of-flux-frameworks-6c16ad26bb31#.90lamiv5l)。

## 使用 Redux
Redux 是一个 JavaScript App的可预测 state 容器。  
如果你觉得需要 Flux 或者相似的解决方案，你应该了解一下 redux，并学习 [Dan Abramov](https://twitter.com/dan_abramov) 的 [Getting started with redux](https://egghead.io/series/getting-started-with-redux)，这能够迅速提高你的开发技能。  
Redux 延续并改进了 Flux 的思想，学习了 Elm ，避开了 Flux 的复杂度（译者注：[Elm](http://elm-lang.org) 是一门函数式编程语言）.  

### 扁平化 state
API 经常会返回嵌套的资源。这在 Flux 或基于 Redux 的架构中处理起来会非常困难。我们推荐使用 [normalizr](https://github.com/gaearon/normalizr) 这类库将数据进行扁平化处理，尽可能地扁平化state.  
像这样：  
```
const data = normalize(response, arrayOf(schema.user))
state = _.merge(state, data.entities)
```  
（我们使用[isomorphic-fetch](https://www.npmjs.com/package/isomorphic-fetch)与API进行通信）  

### 使用 immutable state
共享的可变性 state 是罪恶的根源. —— Pete Hunt, React.js Conf 2015  
![](https://risingstack-blog.s3.amazonaws.com/2016/Jan/immutable_logo_for_react_js_best_practices-1453211749818.png)  
[不可变对象](https://en.wikipedia.org/wiki/Immutable_object)是指在创建后不可再被修改的对象。  
不可变对象可以让我们免于痛苦，并且通过引用级的比对检查来**提升渲染性能**。比如在 ```shouldComponentUpdate``` 中：  
```
shouldComponentUpdate(nexProps) {
 // 不进行对象的深度对比
 return this.props.immutableFoo !== nexProps.immutableFoo
}
```

### 如何在JavaScript中实现不可变?
本办法是小心的写代码,示例代码如下，你需要在单元测试中通过 [deep-freeze-node](https://www.npmjs.com/package/deep-freeze-node) 来反复验证。
```
return {  
  ...state,
  foo
}
 
return arr1.concat(arr2)
```
相信我，这是最明显的例子了.  
更简单也更自然的方式是使用 Immutable.js.
```
import { fromJS } from 'immutable'

const state = fromJS({ bar: 'biz' })  
const newState = foo.set('bar', 'baz') 
```
Immutable.js 非常之快，背后理念也非常美妙.哪怕你并不准备使用它，我也推荐阅读这个由 [Lee Byron](https://twitter.com/leeb) 所制作的视频 [Immutable Data and React](https://www.youtube.com/watch?v=I7IdS-PbEgI)。它非常深刻的讲解了 Immutable.js 的工作原理。  
