# React 服务端渲染实践指南

年前因为工作原因需要对原有 React 项目进行服务端渲染的改造，下面是我对之前工作经验的一些总结分享，希望可以对大家有所帮助。

## 适用场景

首先我们来了解一下 SSR 可以做什么，可以解决什么问题，诞生的原因又是什么。接下来是它到底长什么样子，最然后再是怎么做。需要提前明确的一点是 SSR 并不是万能的，它有它的优缺点和具体的适用场景，我们先来看一下它诞生的历史背景。

### 产生的原因

现如今 SPA 单页面应用已成为主流，相关的开发工具和 MVVM 框架为前端的开发带来了便利和无限的可能性。我们在体验着 SPA 页面开发便捷性的同时，随着业务的发展下面的两个问题也会逐渐暴露出来，而且是在原有模式下很难解决的。

### 发现的问题

- **首屏白屏时间过长。** 在常规 SPA 的页面渲染流程中，首先要加载 HTML 文件，之后要下载页面所需的 JavaScript 文件，然后 JavaScript 文件渲染生成页面， 如果有涉及到数据请求，那么这个耗时将会更加漫长，尤其是在弱网环境下，体验非常糟糕。

- **SEO 能力较弱。** 因为目前大多数搜索引擎主要识别的内容还是 HTML，对 JavaScript 文件内容的识别都还比较弱，所以很难在搜索引擎中有较好的排名。

## 技术原理

SSR 技术随之应运而生，SSR 全称 Server Side Rendering 。以 React 为例，首先我们让 React 代码在服务器端先执行一次，使得用户下载的 HTML 已经包含了所有的页面展示内容，同时，由于 HTML 中已经包含了网页的所有内容，所以网页的 SEO 效果也会变的非常好。之后，我们让 React 代码在客户端再次执行，为 HTML 网页中的内容添加数据及事件的绑定，页面就具备了 React 的各种交互能力，可以参考下图。

![image](./images/sequence.jpg)

核心 API：

服务端： renderToString() | ReactDOMServer.renderToNodeStream()

生成带有标记的 html 文档结构

客户端： ReactDOM.hydrate()

根据服务端携带的标记更新 React 组件树，并附加事件响应

## 劣势

技术改造成本相对较高，node 服务器资源不可控。所以我个人的建议是，要慎重考虑改造的成本和收益，如果收益没有那么明显，最好还是不要轻易尝试

### 可用于替代的技术：

1. 骨架屏 Skeleton
2. 预渲染 Pre-render

   这项技术主要用来解决 SEO 的问题，适用于短时间内不会产生频繁变动的网页。可在服务器端判断 UA，针对爬虫单独返回提前抓取好的 html 内容，最好不要应用在用户环境。

3. next.js (全新项目)
4. Jquery (弱交互页面)

## 项目结构

```
SSR Project
├─build
|  ├─client
|  ├─server
|  └assets.json
├─node_modules
├─public
├─components
├─webpack
|  ├─webpack.config.js
|  ├─webpack.client.config.js
|  └webpack.server.config.js
├─server
|  ├─App.jsx
|  ├─router.js
|  └index.js
├─src
|  ├─pages
|  ├─App.jsx
|  ├─router.js
|  └index.js
├─index.html
├─server.js // 服务端入口
├─package.json
```

## 基础配置

首先要明确我们的目标，我们需要构建一个 node 服务对于页面请求提供渲染完整的 html。

下面是一份 HTML 的 template 页面示例

```
<!DOCTYPE html>
<html lang="cn">
  <head>
    <meta charset="utf-8" />
    <meta
      name="viewport"
      content="width=device-width,height=device-height,maximum-scale=1.0,user-scalable=no;"
    />
    <link href="/main.css" rel="stylesheet" />
  </head>
  <body>
    <div id="root">
      {serverContent}
    </div>
    <script type="text/javascript" src="/bundle.js"></script>
  </body>
</html>
```

对比 SPA 项目其实做的改动很简单，就是把 root 节点内的 html 内容提前在服务端渲染成字符串拼接到模板内。我们采用 express 搭建 node 服务，server 端代码

server/index.js

```
import React from 'react'
import renderToString from 'reactdom/server'
import express from 'express'
import App from 'App'
const app = express()

app.get('/', (req, res, next) => {
  const serverContent = renderToString(<App/>)
  const html = mixin(template,serverContent)
  res.send(html)
})

export default startServer() => {
  app.listen('3000')
  return app
}
```

server.js

```
 var startServer = require('./build/server/index.js')
 startServer()
```

接下来我们要修改 webpack.server.config.js

```
{
  ...
  input: path.resolve(__dirname, '..', 'server/index.js'),
  output: path.resolve(__dirname, '..', 'build/server,'),
  target: 'node',
  ...
}
```

这样服务端基础代码就完成了，但这样是远远不够的，我们还需要做一些其他的处理。

## 静态资源

接下来我们处理静态资源。生产环境的 js bundle 和 css file 都是带有哈希值的，我们希望可以在 html 模板中直接引用到 build 生成的文件。其次图片资源我们也希望不要重复打包。

这里推荐使用 universal-webpack， 他通过帮我们修改 webpack 配置的方式, 帮我们解决上述的问题。插件在打包时会在 build 目录下生成 assets.json 资源定位文件，服务端我们引入这个文件处理即可。
[详细项目文档](https://github.com/catamphetamine/universal-webpack)

webpack.client.config.js

```
import { clientConfiguration } from 'universal-webpack'
const webpackConfig = {...}

return clientConfiguration(webpackConfig, {
    chunk_info_filename: 'assets.json'
}, {
  useMiniCssExtractPlugin : true,
})
```

webpack.server.config.js

```
import { serverConfiguration } from 'universal-webpack'
const webpackConfig = {...}

return serverConfiguration(webpackConfig, {
  // 生成的配置默认第三方模块都不打包，这里配置不支持commonjs的第三方模块
	excludeFromExternals:
	[
		'lodash-es',
		/^some-other-es6-only-module(\/.*)?$/
	],

  // 这里配置不需要重复打包的文件
	loadExternalModuleFileExtensions:
	[
		'css',
		'png',
		'jpg',
		'svg',
		'xml'
	],
})
```

assets.json

```
interface Chunks {
    javascript: {
        [scriptname: string]: string;
    };
    styles: {
        [scriptname: string]: string;
    };
}
```

改造 server/index.js

```
const assets = require('../assets.json')
const js = Object.values(assets.javascript).map(item => <link rel="stylesheet" href="${item}"></link>).join('\n')
const css = Object.values(assets.styles).map(item => `<script src="${item}"></script>`).join('\n')

const html = mixin(template, {
  js,
  css,
  serverContent,
})
```

## 路由

接下来我们进行路由改造，react-router 提供了 StaticRouter 组件，可以用于在服务端渲染，它依赖我们传入的 url 来定位

server/App.jsx

```
import React from 'react'
import { StaticRouter, Switch, Route } from 'react-router-dom'
import routes from './router.js'

export default App(props)  => {
  <StaticRouter location={props.url} context={{}}>
     <Switch>
       {routes.map((route) => {
         <Route {...route} />
       })}
     </Swtich>
  </StaticRouter>
}
```

server/index.js

```
 // relpace
 // const serverContent = renderToString(<App/>)
 const serverContent = renderToString(<App url={req.url}/>)
```

## 数据

在服务端我们可以通过 props 向组件中注入数据

我们一般在 componentDidMount 生命周期执行获取数据的方法，但是在服务端环境中生命周期是不完整的，只会执行 ComponentWillMount 之前的方法，所以我们必须在渲染前准备好数据，
react-router 提供了路由组件判断匹配的方法 matchPath, 根据 match 的组件获取相应的数据,以 Home 组件为例。

server/router.js

```
 // 这里可以通过客户端路由文件改造, 注入loadData方法即可
 const routes = {
   path:'/',
   component: Home,
   loadData: () => getSomeData()
 }
```

server/index.js

```
import { matchPath } from 'react-router-dom'

const promise = Promise.resolve()
routes.find(route => {
  const match = matchPath(req.path, route)
  if(match) promises.then(route.loadData)
  return match
})
promise.then((data) => {
    const serverData = formatData(data)
    // relpace serverContent
    const serverContent = renderToString(<App url={req.url} data={serverData}/>)
    ...
    res.send(html)
  }
})
```

server/App.jsx

```
 export default App(props)  => {
  <StaticRouter location={props.url} context={{}}>
     <Switch>
       {routes.map((route) => {
         <Route {...route} render={() => {
           <route.component data={props.data}/>
         }}/>
       })}
     </Swtich>
  </StaticRouter>
}
```

### 使用 Redux

如果使用 redux 管理同构的数据则会方便许多，这里注意每一个请求都需要重新生成一个新的 store，否则的话用户状态则会混乱。

server/App.jsx

```
 export default App(props)  => {
   <Provider store={props.store}>
    <StaticRouter location={props.url} context={{}}>
      <Switch>
        {routes.map((route) => {
          <Route {...route} />
        })}
      </Swtich>
    </StaticRouter>
  </Provider>
}
```

server/index.js

```
promise.then((data) => {
    const preloadedState = mixin(initData, data)
    const store = createStore(reducers, preloadedState)
    // relpace serverContent
    const serverContent = renderToString(<App url={req.url} stote={store}/>)
  ...
}
})
```

### 客户端同步数据

至此为止，服务器端最终会输出一个带有数据状态的完整页面。但是客户端这边重新渲染的时候，首先会渲染一个没有数据的框架，然后才会在 componentDidMount 里发起数据接口请求数据，这意味着在这之前客户端都为空数据状态，在用户看来就是表现为会执行重复 loading 。所以我们希望客户端可以共享服务端已经获取的数据，解决办法是我们会在服务端将数据注入到 HTML 中返回给客户端**脱水**(Dehydrate)。在浏览器端，客户端不再自己发起请求获取数据处理状态，直接使用这个脱水数据来初始化 React 组件**注水** (Hydrate)

![image](https://user-gold-cdn.xitu.io/2019/2/27/1692e44759620877?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

HTML 模板

```
    ...
    </div>
    <script type="text/javascript" src="/bundle_[hash].js"></script>
    <script>window.__initState__ = ${JSON.stringfy(store)}</script>
  </body></html>
```

server/index.js

```
  // replace
  // const html = mixin(template, {js, css, serverContent})
  // const html = mixin(template, {js, css, serverContent, store})
})
```

store.js

```
  const defaultState = JSON.parse(window.__initState__)
  const store = createStore(reducer, defaultState)
```

## 注意事项

1. 服务端执行环境没有 window 和 document 等宿主对象，且会执行组件的 constructor，componentWillReceiveProps，render 生命周期，所以务必避免代码中的此类调用。可以通过 type window 或 webpack.definePlugin 来对客户端和服务端做区分

## 待商讨&&完善

按需加载

HMR
