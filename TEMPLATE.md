# mpapi

> mpapi（miniProgram API），小程序 API 兼容插件，一次编写，多端运行。

:alarm_clock: 更新日期: <%= obj.updateDate %>，小程序功能一直在更新，新版本可能有所差异，请留意。

[![NPM][img-npm-badge]][url-npm] [![Language][img-javascript]][url-github] [![License][img-mit]][url-mit]

**此项目解决的问题**：寻找不同小程序 API 之间的差异，尽可能地通过**一套 API 兼容多个小程序使用**。


## 特点
- 一次编写，多端运行，支持: 微信小程序、支付宝小程序、百度智能小程序、字节跳动小程序
- 支持 Promise（包含 success 回调的才有）
- 针对某些 API 使用做了优化，如：`api.showToast` 可以直接传 `string`、`api.setStorageSync` 无需调用 `try catch 等`
- 支持特殊 API 的事件处理，例如：`request`、`downloadFile`，[详情](#特殊api的事件处理)
- 支持不同端的判断，`api.isWechat`、`api.isAlipay`、`api.isSwan`、`api.isTt`


## 安装
```bash
npm install mpapi --save
```
非 npm 安装方式，直接引入 `lib` 目录下的 `mpapi.js` 到项目即可


## 使用
```javascript
const api = require('mpapi')

api.alert({...}).then((res) => {})
api.confirm({...}).then((res) => {})
api.getLocation().then((res) => {})
...
```


## 快速查看
- [兼容 API 列表](#兼容api列表)
- [其它包装成 Promise 的 API](#其它包装成promise的api)
- [API 差异](#小程序之间的api差异)
- [使用说明](#使用说明)
- [特殊 API 的事件处理](#特殊api的事件处理)，`request`、`downloadFile`、`uploadFile` 等
- 官方 API 文档：[微信小程序](https://developers.weixin.qq.com/miniprogram/dev/api/)、[支付宝小程序](https://docs.alipay.com/mini/api/overview)、[百度智能小程序](http://smartprogram.baidu.com/docs/develop/api/net_rule/)、[字节跳动小程序](https://developer.toutiao.com/docs/framework/)

## 兼容API列表
> 所有小程序都可以使用的 API
<% _.each(obj.polyfills, function(group){ %>
- <%= group.title %>
<% _.each(group.items, function(item){ %>  - [x] `<%= item.title %>`
<% }) %>
<% }) %>


## 其它包装成Promise的API
> 只在特定小程序下才会支持

微信小程序![wx](./assets/wx.png)、支付宝小程序![my](./assets/my.png)、百度智能小程序![swan](./assets/swan.png)、字节跳动小程序![tt](./assets/tt.png)，有图标表示只支持对应小程序，没有图标表示都支持。

<% _.each(obj.normalApi, function(group){ %>
- <%= group.title %>
<% _.each(group.items, function(item){ %>  - [x] `<%= item.title %>` <% _.each(item.labels, function(lbl){ %> ![%= lbl %>](./assets/<%= lbl %>.png) <% }) %>
<% }) %>
<% }) %>

- **深层级的 API，注意：方法加了 `$` 前缀**
  - `api.ap` ![my](./assets/my.png)
    - [x] `api.ap.$faceVerify`
    - [x] `api.ap.$navigateToAlipayPage`
    - [x] `...`
  - `api.ai` ![swan](./assets/swan.png)
    - [x] `api.ai.$ocrIdCard`
    - [x] `api.ai.$ocrBankCard`
    - [x] `...`

- **某些新实例的对象上面的 API**
  <% _.each(obj.instanceApi, function(group){ %>
  - [x] `<%= group.title %>` <% _.each(group.labels, function(lbl){ %>![%= lbl %>](./assets/<%= lbl %>.png) <% }) %><% }) %>

例如：**注意：方法加了 `$` 前缀**
```javascript
let ctx = api.createMapContext('maper')

ctx.$getCenterLocation().then((res) => {
  console.log('createMapContext:getCenterLocation')
  console.log(res)
})
```


## 小程序之间的API差异

1、传参不一致
 
例如：`showLoading` 方法，加载的显示文案，微信和百度里面是 `title` 参数，支付宝里面是 `content` 参数，如下
```javascript
// 微信
wx.showLoading({
  title: '加载中'
})

// 百度
swan.showLoading({
  title: '加载中'
})

// 支付宝
my.showLoading({
  content: '加载中'
})

// 使用 mpapi 之后，多端兼容
api.showLoading('加载中')
api.showLoading({
  title: '提示内容'
})
```

2、返回参不一致
 
例如：`showActionSheet` 方法，执行完之后获取选择的索引，微信和百度里面是 `res.tapIndex`，支付宝里面是 `res.index`，如下
```javascript
// 微信
wx.showActionSheet({
  itemList: ['台球', '羽毛球', '篮球'],
  success: (res) => {
    // res.tapIndex
  }
})

// 支付宝
my.showActionSheet({
  items: ['台球', '羽毛球', '篮球'],
  success: (res) => {
    // res.index
  }
})

// 使用 mpapi，多端兼容
api.showActionSheet({
  itemList: ['台球', '羽毛球', '篮球'],
  success: (res) => {
    // res.tapIndex
  }
})
```

3、不支持，但可兼容

例如：支付宝里面有 `my.alert`，而微信和百度里面没有此方法，但是可以通过微信的 `wx.showModal` 或百度的 `swan.showModal` 封装一个 `alert` 方法，如下
```javascript
api.alert('提示内容')

api.alert({
  content: '提示内容'
})

// 请求数据，兼容多端
api.request({...}).then((res) => {})
```

4、不支持，无法兼容

有的 API 只在特定端里面有效，无法兼容处理，如下
```javascript
// 只在支付宝里面有效，微信和百度小程序里面会报错
api.startZMVerify({...})

// 建议这样处理
if(api.isAlipay){
  api.startZMVerify({...})
}

// 只在微信里面有效，支付宝或百度小程序里面会报错
api.setTopBarText({...})

// 建议这样处理
if(api.isWechat){
  api.setTopBarText({...})
}

// 百度智能小程序的特殊 API 一样的道理
if(api.isSwan){
  api.getSwanId().then((res) => {})
}
```


## 使用说明

1、支持 `Promise` 风格

所有小程序的 API 只要包含 `success` 回调，都已经用 `Promise` 封装过，可以直接使用，两种写法都支持，例如
```javascript
// 使用回调
api.showActionSheet({
  itemList: ['台球', '羽毛球', '篮球'],
  success: (res) => {
    // res.tapIndex
  }
})

// 或者
api.showActionSheet({
  itemList: ['台球', '羽毛球', '篮球']
}).then((res) => {
    // res.tapIndex
})

// 其它
api.setStorage({...}).then((res) => {})
api.chooseImage({...}).then((res) => {})
...
```

2、兼容方法里的传参和返回参，**以微信小程序调用为准**。其它端不兼容的参数不处理（某些参数也无法处理，特定小程序不支持）开发者需要留意，例如
```javascript
api.chooseImage({
  count: 1,
  sizeType: ['original', 'compressed'], // 只在微信可用
  sourceType: ['album', 'camera'],
}).then((res) => {
  // res.tempFilePaths 在微信和支付宝都可用
  // res.tempFiles 只在微信可用
})
```


## 特殊API的事件处理
某些 API 既要支持 Promise，又要调用它的事件，那么可以采用如下方式：

**以前：**
```javascript
const downloadTask = wx.downloadFile({
  url: 'https://example.com/audio/123', // 仅为示例，并非真实的资源
  success(res){
    console.log(res)
  }
})

downloadTask.onProgressUpdate((res) => {
  console.log(res)
})

downloadTask.abort() // 取消下载任务
```
**使用 `mpapi` 之后：**
```javascript
const api = require('mpapi')

const downloadTask = api.downloadFile({
  url: 'https://example.com/audio/123' // 仅为示例，并非真实的资源
}).then((res) => {
  console.log('success')
  console.log(res)
})

downloadTask.$event('onProgressUpdate', (res) => {
  console.log(res)
})

// downloadTask.$event('abort') // 取消下载任务
```
其它 API 可以类似处理，例如：`request`、`uploadFile`、`connectSocket`


## Issues
如果您在使用过程中发现 Bug，或者有好的建议，欢迎[报告问题](https://github.com/ChanceYu/mpapi/issues)。


## Changelog
[更新日志](./CHANGELOG.md)


## License

[![license][img-mit]][url-mit]


[url-github]: https://github.com/ChanceYu/mpapi
[url-npm]: https://www.npmjs.com/package/mpapi
[url-mit]: https://opensource.org/licenses/mit-license.php

[img-npm]: https://nodei.co/npm/mpapi.png?compact=true
[img-npm-badge]: https://img.shields.io/npm/v/mpapi.svg
[img-javascript]: https://img.shields.io/badge/language-JavaScript-brightgreen.svg
[img-mit]: https://img.shields.io/badge/license-MIT-blue.svg

