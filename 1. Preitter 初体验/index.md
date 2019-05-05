# Prettier 初体验

### 前言

最近正在新做一个新的项目，准备尝试一下社区很火的**Prettier**，一边学习的同时也正好总结了一些经验，在这里做一些记录

### 什么是 Prettier

Prettier 是一个代码格式化工具，它可以支持 JS/JSX/TS/Flow/JSON/CSS/LESS 等文件格式。

### 为什么要用 Prettier

用来替代 lint 中的一些场景，比如说分号/tab 缩进/空格/引号，这些在 lint 工具检查出问题之后还需要手动修改，而通常这样的错误都是空格或者符号之类的，这样相对来说不太优雅，利用格式化工具自动生成省时省力。

### 如何自定义配置

Prettier 提供了一套默认的配置，那么如何修改配置项符合我们自己的代码规范呢，有三种方法可以做到

1. .prettierrc 文件

2. prettier.config.js 文件

3. package.json 中配置 prettier 属性

Prettier 会检查配置文件并自动读取文件中的配置，我们只需要选一种方法配置就好了，我现在选的是第二种。
是不是有种感觉跟 lint 工具很像呢？

### 集成 VS CODE

只需要下载 prettier 的插件就可以了,在插件市场搜索可以直接搜索到

下面是愉快使用的相关配置

```
editor.formatOnPaste: Boolean 粘贴时自动格式化

editor.formatOnSave: Boolean 保存时自动格式化

editor.formatOnType: Boolean 键入一行后是否格式化

editor.formatOnSaveTimeout: Int 保存后延时格式化
```

### 可配置的属性

分享一下我的配置文件

```
module.exports = {
// tab缩进大小,默认为2
tabWidth: 2,


// 使用tab缩进，默认false
useTabs: true,


// 使用分号, 默认true
semi: false,


// 使用单引号, 默认false(在jsx中配置无效, 默认都是双引号)
singleQuote: true,


// 行尾逗号,默认none,可选 none|es5|all
// es5 包括es5中的数组、对象
// all 包括函数对象等所有可选
TrailingCooma: "none",


// 对象中的空格 默认true
// true: { foo: bar }
// false: {foo: bar}
bracketSpacing: true,


// JSX标签闭合位置 默认false
// false: <div
//          className=""
//          style={{}}
//       >
// true: <div
//          className=""
//          style={{}} >
jsxBracketSameLine：false,


// 箭头函数参数括号 默认avoid 可选 avoid| always
// avoid 能省略括号的时候就省略 例如x => x
// always 总是有括号
arrowParens: 'always',

}

```
