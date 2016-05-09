
本文讲的是一种在 React Web 和 React Native 共享应用程序逻辑,但在每个平台保持独立的渲染特性的方法。示例应用程序可以在 [这里](https://github.com/kauffecup/react-native-web-hello-world) 上找到.

## 应用

<img src="https://raw.githubusercontent.com/huangciyin/notes/master/ReactNative/images/mobile.gif" alt="native" style="width: 200px;">
<img src="https://raw.githubusercontent.com/huangciyin/notes/master/ReactNative/images/mobile.gif" alt="native" style="width: 700px;">

这个应用程序本身是一个非常简单的 Hello World 应用程序。它不仅会显示"Hello World"，而且当您点击它时,颜色会从红色改变成蓝色!哇!

## 动机
不管是 Web 还是 Mobile, React 都是很棒的. 所以为什么在你的两个实现之间分享代码?

React Native 并不是为了实现"一次编写，到处运行"的框架。Facebook 一直称之为"一次学习，到处编写"的框架-这就是说你需要为你的平台进行定制。即便这样，你仍然可以应用程序之间共享大量的逻辑。

在这篇文章我将讨论如何吸取这两种思想的优点,达到一个融合共通的状态。在坚持每个平台定制呈现方式的同时，我们会共享所有应用程序逻辑。

我们会假设你已经具备以下知识: [React](https://facebook.github.io/react/), [React Native](https://facebook.github.io/react-native/) 和 [Redux](http://redux.js.org/).

## 初始安装 + 目录概述
首先我们需要初始化我们的项目。我们要按照取决于 [Facebook 的入门指南](https://facebook.github.io/react-native/docs/getting-started.html) 中的步骤 ︰

```
$ npm install -g react-native-cli
$ react-native init ReactNativeWebHelloWorld
```

现在，我们的目录看起来是这样:

```
ReactNativeWebHelloWorld
|-- android
|-- ios
|-- node_modules
|-- .flowconfig
|-- .gitignore
|-- .watchmanconfig
|-- index.android.js
|-- index.ios.js
+-- package.json
```
这包含我们需要为我们的 iOS 和 Android 应用程序的所有文件。现在，我们创建下列目录和文件来配置和运行我们的 Web 应用程序 ︰

```
ReactNativeWebHelloWorld
+-- web
    |-- public
    |   +-- index.html
    +-- webpack
        |-- web.dev.config.js
        +-- web.prod.config.js
```
[index.html](https://github.com/kauffecup/react-native-web-hello-world/blob/master/web/public/index.html)， [web.dev.config.js](https://github.com/kauffecup/react-native-web-hello-world/blob/master/web/webpack/web.dev.config.js) 和 [web.prod.config.js](https://github.com/kauffecup/react-native-web-hello-world/blob/master/web/webpack/web.dev.config.js) 的内容都可以在 [GitHub](https://github.com/kauffecup/react-native-web-hello-world) 上找到 - 我们会更晚些时候分析他们 (但如果你现在就想点击他们,那就点吧!).

当我们改好目录结构后，接着安装依赖项:

```
$ npm install --save babel babel-polyfill ...
$ npm install --save-dev autoprefixer babel-core ...
```
依赖项的完整列表，请查阅 package.json.

最后，我们为应用程序初始化的所有文件。我们将会做一个相当通用的 React/Redux 应用程序 ︰

```
ReactNativeWebHelloWorld
+-- app
    |-- actions
    |-- constants
    |-- reducers
    |-- store
    |-- native
    |   |-- components
    |   |-- containers
    |   +-- style
    +-- web
        |-- components
        |-- containers
        +-- style
```
这时候就已经相当清晰了。我们为不同的应用程序有三个不同的入口点 ︰ ```index.ios.js```、 ```index.android.js```和```app/web/index.js```。IOS 和 Android 的入口文件从```app/native```加载组件和容器， web 入口文件从```app/web```加载组件和容器。这给我们带来我们了:

## 应用程序代码结构
我不会去分析所有的文件，但要指出一些 Web 和 Native 的关键差异。

让我们看看应用程序入口文件,```index.ios.js```看起来是这样:

```
import React, { Component, AppRegistry } from 'react-native';
import Root           from './app/native/containers/Root';
import configureStore from './app/store/configureStore.prod.js';

const store = configureStore();

class ReactNativeHelloWorld extends Component {
  render() {
    return (
      <Root store={store} />
    );
  }
}

AppRegistry.registerComponent('ReactNativeWebHelloWorld', () => ReactNativeHelloWorld);
```

```app/web/index.js```是这样

```
import React          from 'react';
import { render }     from 'react-dom';
import Root           from './containers/Root';
import configureStore from '../store/configureStore';

// load our css
require('./styles/style.less');

const store = configureStore();
const rootElement = document.getElementById('root');

render( <Root store={store} />, rootElement );
```

好的所以我们关心的区别是什么？

需要注意的主要事情是顶级组件如何渲染他们自己。在 Native 部分，我们必须在 app registry 中显式定义，而在 web 中我们可以使用 ```ReactDom```，把```Root```直接渲染在根元素中。

那是什么意思！？

基本上，React Native 和 React Web 有不同的方式的实例化顶级组件。

<b>正是这些差异，要求我们坚持每个平台独特的渲染逻辑。</b>

让我们也检查两种情况下```HelloWorld```组件的```render```方法。在 Native 中，它看起来是这样:


```
render() {
  const { onPress, color } = this.props;
  const style = StyleSheet.create({
    helloWorld: { color: color, textAlign: 'center' }
  });
  return (
    <View>
      <Text onPress={onPress} style={style.helloWorld}>Hello World</Text>
    </View>
  );
}

```

Web 中，它看起来是这样:

```
render() {
  const { onClick, color } = this.props;
  return (
    <div className="hello-world" onClick={onClick} style={ {color: color} }>Hello World</div>
  );
}
```
这就又证明了为什么我们需要坚持每个平台独特的渲染逻辑。 React Native 处理 ```<View>s``` 和```<Text>s``` ，而 web 处理```<div>s``` 和```<span>s```。不仅如此，而且事件系统和样式系统都不太一样。

<b>但我们也看看什么他们共享什么...</b>

实例化```HelloWorld```组件时， ```app/native/containers/App.js```的定义:


```
<HelloWorld
  onPress={() => dispatch(toggleColor())}
  color={color}
/>

```

```app/web/containers/App.js```的定义:

```
<HelloWorld
  onClick={() => dispatch(toggleColor())}
  color={color}
/>
```
这两个```dispatch```方法都是从```react-redux```导入的，```toggleColor```也是从相同的```actions```文件导入。只的渲染方式不同! 应用程序逻辑是共享的! 这是一个大的飞跃!

相比一个接一个的比较相似性和差异性,我们去看看在```package.json```中定义的脚本，让你可以生成并运行这个坏小子......

## 配置的脚本
### 在开发和生产环境运行

```package.json```有 8 定义的脚本:

1. start  
2. ios-bundle
3. ios-dev-bundle
4. android-bundle
5. android-dev-bundle
6. web-bundle
7. web-dev

<b>start</b>

start 是用来打包和运行 native 程序的。当你打开要么 xcode 项目或 android studio 并点击"run"时，它通过start命令启动一个 node 服务器。每次你改了 JavaScript，你不需要重建和重新编译您的应用程序，你只需刷新，这些更改将奇迹般地生效。由于本文不是 React Native 引导,我不会介绍更多信息 - 你可以去 [React Native Getting Started](https://facebook.github.io/react-native/docs/getting-started.html)  查看。

### 内置

通过```ios-bundle```、 ```ios-dev-bundle```、 ```android-bundle```和```android-dev-bundle```，此脚本生成 JavaScript bundle （是否压缩取决于你是否开启了```dev``` 选项），并将其放置在恰当的地方,以使可以在设备上本地运行。另外，你可以去 [React Native Getting Started](https://facebook.github.io/react-native/docs/getting-started.html)  查看关于本地运行的更多信息.

### web

```web-dev``` 在 3001 端口 打开了一个webpack 服务器上，它利用热加载和redux-time-machine-magic的特性，以使您可以进行还原和重放你的操作。

```web-bundle```创建压缩版的 JavaScript bundle(也压缩 css)，并将其放置在```web/public```的```index.html``` 中,你可以搭配任何静态文件服务器的使用它。

### 清除缓存

有时候，当 React Native 工作时，你会确信你已经改变了一些东西，但它(缓存)仍然会导致您的 App 出问题! 哦不! 我们该怎么办!

```npm run clear-cache```

## 进一步的配置
Webpack 把 ```PLATFORM_ENV```环境变量设置为了 web 。您可以使用此环境变量有条件地加载不同的文件，取决于您在构建 Web 还是 Native 程序。举个例子 - 你可以基于此抽象出本地存储机制之间的区别。

## 你得到什么？
嗯，首先，这是很有趣的事情。我们有时候单纯为了精神愉悦做某些事情。但如果这对你来说还不够，找出您的应用程序中的 action/reducer/request 层面的 bug,你可以一次性搞定他们，而不需要为你的 Web 和 Native 程序分别处理.
它还能加速你在多平台上的开发速度。经过2天的努力,我就能够得让我的一个 react/redux 的应用程序功能完整的运行在手机上了.

