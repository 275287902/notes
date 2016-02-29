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
在 React.js 应用中处理数据非常简单，但也充满挑战。  
这是因为你可以使用多种方式将属性数据传递给 React 组件，从而构建出渲染树,但你应该怎样更新视图却不是显而易见的。
