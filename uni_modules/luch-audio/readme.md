**插件使用说明**

- 基于 uni.createInnerAudioContext() 封装audio音频组件（样式同原生audio组件）
- 支持双向绑定


**Example**
---

``` javascript 
<luch-audio src="https://img-cdn-qiniu.dcloud.net.cn/uniapp/audio/music.mp3" :play.sync="audioPlay"></luch-audio>

// 控制变量audioPlay ，如果true音频会播放，否则暂停
```

**Attributes**
--



| 参数 | 说明 | 类型 | 可选值 | 默认值 | required |
| ---- | ---- | ---- | ------ | ------ | -------- |
|play|是否播放,双向绑定，需加 ` .sync `|Boolean| — | — |—|
|src|资源路径|String| — | — | —|
|poster|封面图片路径|String| — | — | —|
|name|音频名称|String| — | 未知音频 | —|
|author|作者名字|String| — | 未知作者 | —|
|autoplay|是否自动开始播放；H5端部分浏览器不支持|Boolean| — | — | —|
|loop|是否循环播放|Boolean| — | — | —|
|obeyMuteSwitch|是否遵循系统静音开关，当此参数为 false 时，即使用户打开了静音开关，也能继续发出声音|Boolean| — | — | —|
|initAudio|初始化AudioContext 回调；返回 innerAudioContext； { src，  index}|Function|—|—|—|
|index|自定义组件标识；在initAudio 返回|String, Number|—|—|—|



**Events**

| 事件名       | 说明                                         | 参数   |
| ------------ | -------------------------------------------- | ------ |
| onPlay       | createInnerAudioContext 的 onPlay 事件       | 同原生 |
| onPause      | createInnerAudioContext 的 onPause 事件      | 同原生 |
| onEnded      | createInnerAudioContext 的 onEnded 事件      | 同原生 |
| onTimeUpdate | createInnerAudioContext 的 onTimeUpdate 事件 | 同原生 |
| onError      | createInnerAudioContext 的 onError事件       | 同原生 |



**平台支持度**
--

|5+App|H5|微信小程序|支付宝小程序|百度小程序|头条小程序|
| ---- | ---- | ---- | ------ | ------ | -------- |
|yes|yes|yes|1.23.4+|yes|yes|

**注意：**以上平台支持度为uni-app 对` uni.createInnerAudioContext() ` api 的支持度[[文档](https://uniapp.dcloud.io/api/media/audio-context.html)]，本组件的支持度本人只在微信小程序使用过，其他平台*未做测试*

**使用**
--

下载后把该文件放至 components 文件夹或 特定平台组件 文件夹即可

**更新**
--

[ChangeLog](./changelog.md)

**可扩展性**
--

api 增加参考 ` https://uniapp.dcloud.io/api/media/audio-context?id=createinneraudiocontext `
<br>
组件内的播放暂停图标本人因为要发布组件，所以使用了base64,大家可自行替换；
<br>
实例获取可以通过initAudio获取

**说明**
--

切换src 或者切换页面都会销毁实例<br>
写本组件的原因就是原生audio组件无法修改宽度，导致某些情况下显示不全，本组件是100%自适应布局；

更多用法，参考示例
