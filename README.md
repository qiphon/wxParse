# wxParse —— 微信小程序富文本解析

![](https://img.shields.io/github/stars/csonchen/wxParse?style=flat-square)
![](https://img.shields.io/github/forks/csonchen/wxParse?style=flat-square)
![](https://img.shields.io/github/license/csonchen/wxParse?style=flat-square)

## 原因

由于原作者仓库 [wxParse](https://github.com/icindy/wxParse) 不再维护，我们项目中商品信息展示又是以wxParse这个用做富文本解析的；

于是乎，决定采用 **递归Component** 的方式对其进行重构一番；

原项目使用的 `template` 模板的方式渲染节点，存在以下问题：

1. 节点渲染支持到12层，超出会原样输出 `html` 代码；

2. 每一层级的循环模板都重复了一遍所有的可解析标签，代码十分臃肿。

3. `li `标签不支持 `ol` 有序列表渲染（统一采用的是 `ul` 无序列表），`a`标签无法实现跳转，也无法获取点击事件回调等等；

4. 节点渲染没有绑定 `key` 值，一是在开发工具看到一堆的 `warning`信息（看着十分难受），二是节点频繁删除添加，无法比较`key值`，造成 `dom` 节点频繁操作。

### changed 

#### v1.0.7

- 由于当前版本不能够控制内部样式，所以额外添加了外部类名注入     externalClasses: ['parse-wrapper'],  `<wxParse wrapperClass="color-red" nodes="{{ article.nodes }}" />`

```js
const selfClasses = {
  wrapper: 'wrapper-class',
  button: 'q-button',
  ol: 'q-ol',
  ul: 'q-ul',
  li: 'q-li',
  video: 'q-video',
  imginner: 'q-imginner',
  a: 'q-a',
  table: 'q-table',
  tr: 'q-tr',
  td: 'q-td',
  audio: 'q-audio',
  hr: 'q-hr',
  block: 'q-block',
  inline: 'q-inline',
  text: 'q-text',
}
```
- 取消a 链接的字体颜色
- 去除 tbody margin，table 补充margin

## 功能标签

目前该项目已经可以支持以下标签的渲染：

  - [x] audio标签（可自行更换组件样式，暂时采用微信公众号文章的`audio`音乐播放器的样式处理）**【扩展组件】**
  - [x] code标签 **【扩展组件】**
  - [x] video标签
  - [x] table标签
  - [x] ul标签
  - [x] ol标签 
  - [x] li标签
  - [x] a标签
  - [x] img标签
  - [x] video标签
  - [x] br标签
  - [x] button标签
  - [x] h1, h2, h3, h4标签
  - [x] 文本节点
  - [x] hr标签
  - [x] 其余块级标签
  - [x] 其余行级标签

## 使用

### 1. 原生组件使用方法

- 克隆 [项目](https://github.com/csonchen/wxParse) 代码，把 **dist目录下的wxParse目录** 拷贝到你的小程序组件目录下面;

- 在你的 **page页面** 对应的 `json` 文件引入 `wxParse` 组件

```json
{
  "usingComponents": {
    "wxParse": "/components/wxParse/wxParse"
  }
}
```

- 组件调用

```xml
<wxParse nodes="{{ htmlText }}" />
```


### 2. npm组件使用方法

- 安装组件

```shell
npm install --save wx-minicomponent
```

- 小程序开发工具的 `工具` 栏找到 `构建npm`，点击构建；

- 在页面的 json 配置文件中添加 `wxParse` 自定义组件的配置

```json
{
  "usingComponents": {
    "wxParse": "/miniprogram_npm/wx-minicomponent/wxParse"
  }
}
```

- `wxml` 文件中引用 wxParse

```xml
<wxParse nodes="{{ htmlText }}" />
```

## 组件扩展

基础的富文本组件只支持基础的标签解析，出于小程序包体积考虑，读者可以根据需要按需引入组件，打包构建。

名称|功能|构建命令
:--|:--|:--
wxAudio | 音频播放器 | npm run pack:wxAudio
highLight | 代码块高亮显示 | npm run pack:highLight

**使用方法：**

1. 需要引入音频播放器组件，执行命令:

```shell
npm run pack:wxAudio
```

2. 执行构建压缩命令，打包到 `dist` 目录下：

```shell
npm run build
```

3. 将dist目录下的`wxParse目录`和`wxAudio目录`克隆到你的项目组件目录，正常引入wxParse组件即可。

## 参数文档

### wxParse：富文本解析组件

参数|说明|类型|例子
:--|:--|:--|:--
nodes|富文本字符|String|"\<div\>test\</div\>"
language|语言|String| 可选："html" \| "markdown" ("md")

**标签使用说明补充：**

1. `a` 标签的内外链跳转根据的是 `http` 字符判断；

2. `a` 标签的跳转顺序为：

- 如果page页面有定义 `handleTagATap` 方法，优先执行该方法

- 如果page页面没有定义 `handleTagATap` 方法，将根据 `a标签` 的 `href` 字段判断采用内外链跳转方式，外链跳转需要在 `app.json` 文件中新增 `自定义webview` 页面配置，如下所示：

**webview页面配置：**

1. 原生webview页面配置：

```json
// app.json
{
  "pages" [
    "components/wxParse/webviewPage/webviewPage"
  ]
}
```

2. npm 包webview配置：

```json
// app.json
{
  "pages" [
    "miniprogram_npm/wx-minicomponent/wxParse/webviewPage/webviewPage"
  ]
}
```

3. `code` 标签代码高亮：

- `code` 标签添加 `lang`属性，默认值是`javascript` ；支持 `javascript|typescript|css|xml|sql|markdown|c++|c` 可选值

- 支持高亮解析结构如下：

```html
<pre>
<code lang="javascript">
  const name = 'csonchen'
</code>
</pre>
```

**受信任的节点**

节点|例子
:--|:--
audio|\<audio title="我是标题" desc="我是小标题" src="https://media.lycheer.net/lecture/25840237/5026279_1509614610000.mp3?0.1" /\>
a|\<a href="https://www.baidu.com"> 跳转到百度 \</a>  </br> \<a href="/pages/highLightPage/highLightPage"> 站内跳转 \</a>
code| \<code lang="javascript"> const name = "csonchen"; <\/code>
p|
div|
span|
li|
ul|
ol|
img|
button|
hr|
h1|
h2|
h3|
h4|
table|
tr|
th|
td|
....|

### highLight：代码高亮解析组件

参数|说明|类型|例子
:--|:--|:--|:--
codeText|原始高亮代码字符|String|"var num = 10;"
language|代码语言类型|String|可选值："javascript/typescript/css/xml/sql/markdown/c++/c"

**提示：如果是html语言，language的值为xml**


### wxAudio：仿微信公众号文章音频播放组件

参数|说明|类型|例子
:--|:--|:--|:--
title|标题|String|"test"
desc|副标题|String|"sub test"
src|音频地址|String|

## 示例展示

1. **富文本解析**

- **html文本解析实例**

<left>
<figure>
<img src="./static/code.png" style="height: 500px"/>
<img src="./static/wxParse.gif" style="height: 500px"/>
</figure>
</left>

- **markdown文本解析实例**
<left>
<figure>
<img src="./static/md.png" style="height: 500px" />
</figure>
</left>

2. **代码高亮（highLight组件）**

<left>
<figure>
<img src="./static/wxHigh.gif" style="height: 500px" />
</figure>
</left>

## 更新历史

- 2021-5-17：**`feat`** 将`wxAudio音频播放器`和`highLight代码块高亮组件`抽离出来作为扩展组件，可根据需要自行构建打包

- 2021-3-26: **`feat`** 修复图片二次请求加载的bug，优化图片懒加载 + 展示渐变动画

- 2021-2-6: **`fix`** 修复单层元素（如：`img`标签）渲染时候获取不到组件实例唯一id，导致大图预览失败的bug

- 2021-1-19： 

 1. **`fix`** 修复 `page页面` 多 `wxParse组件` 实例绑定唯一性的bug

 2. **`feat`** 新增 `hr`标签解析实现

- 2020-12-10：**`fix`** 修复页面跳转之后图片预览大图失效的bug

- 2020-12-8： **`feat`** 新增 `code` 标签的代码高亮功能，并修复解析语言注册失败的bug

- 2020-12-7：**`fix`** 修复 `code` 标签解析错误以及换行符被替换成空字符的bug

- 2020-8-17：**`feat`** 增加设置image标签的width，height属性以及内联样式属性设置

- 2020-8-10: **`feat`** 文本节点添加选中复制功能

- 2020-8-6：**`fix`** table，tr，td等相关元素标签去掉style样式内嵌，避免表格样式错乱问题

- 2020-8-5: **`feat`** 新增支持a标签的内外链跳转功能，并支持page页面点击方法回调控制

- 2020-6-18：**`fix`** 修复table渲染错位和image图片在个别情况无法预览的问题

- 2020-5-31: **`reflator`**

 1. 迁移utils目录到wxParse目录下；

 2. 富文本增加markdown文本解析支持；

- 2020-5-21: **`reflator`** 富文本组件image标签添加loading过渡态，优化图片加载体验

- 2020-5-17: **`doc`** 完善组件参数文档，增加wxParse对audio标签标题，副标题的解析功能

- 2020-5-13: **`feat`** 增加css，html，ts，sql，markdown代码高亮提示支持

- 2020-5-6：**`feat`** 增加图片预览功能

## TODO

- [x] 图片占位图优化，优化 `image`标签加载，避免出现一闪而过，优化加载体验；
- [ ] 编写插件，小程序可以通过插件方式引入；
- [ ] 支持 `template` 方式渲染（因为递归组件会引入组件生命周期，节点过多可能对性能有影响）；
- [ ] 尽可能多的修复原 `wxParse` 项目中的 `issue`
- [ ] ....