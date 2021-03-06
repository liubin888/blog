### 背景
在小程序的webview里保存图片. 因为微信的 `js-sdk` 没有提供 `saveImageToPhotosAlbum` 方法

更多 `web` 和小程序的交互,请看 **[这里](https://www.chuchur.com/article/interaction-between-html-webchat-miniprogram)**    

### 解决思路   
先加载 微信 `js-sdk`
```html
<script src="https://res.wx.qq.com/open/js/jweixin-1.6.0.js"></script>
```
### 分三步
#### 1、 `html` 端把图片转为 `base64`,然后通过 `postmessage` 传递给小程序
```js
let img = new Image()
img.src = 'xxxx' //这里是图片的src
img.crossOrigin = 'anonymous'  //The opeartaion is insecure .其它跨域的问题 自行代理解决
img.onload = function () {
  let canvas = document.createElement('canvas')
  canvas.width = img.width
  canvas.height = img.height
  let ctx = canvas.getContext('2d')
  ctx.drawImage(this, 0, 0)
  let imgBase64Data = canvas.toDataURL('image/jpeg', 1)  //这里就拿到了base64
  wx.miniProgram.postMessage({
    data: {
      imgData: imgBase64Data // 刚才拿到的base64 数据
    }
  })
}
```
#### 2、小程序监听 `postmessage` 拿到 图片 `base64` 数据
```js
// wxml
<web-view  src="http://www.chuchur.com/save-image" bindmessage="getMessage"></web-view>

// js
Page({

  data:{
    imageData:null
  }

  getMessage(e){

    this.setData({
      imageData : e.detail.data[0].imgData
    })
  }
})

```

#### 3.保存图片到相册(在小程序里)

因为拿到是 `base64` 图片数据,首先要把它存为 图片文件
```js
wx.getFileSystemManager().writeFile({
  filePath: wx.env.USER_DATA_PATH + '/qrcode.png',  //这里先把文件写到临时目录里. 
  data: this.data.imageData.slice(22), // 注意这里
  encoding: 'base64',
  success: res => {
    console.log('success')
  },
  fail: error => { console.log(error) }
})
```
>`getFileSystemManager` 的 `writeFile` 写入的 `base64` 是不包含 图片的头字节的. 所以要干掉 `data:image/jpeg;base64,` 等字符. 

有了文件路径就可以保存到相册了

```js
wx.saveImageToPhotosAlbum({
  filePath: wx.env.USER_DATA_PATH + '/qrcode.png', //这是把临时文件 保存到 相册, 收工
  success: res => {
    wx.showToast({ title: '保存成功！' })
  },
  fail: error => { console.log(error) }
})
```

### 没有接收到？不是实时触发?

文档发现虽然h5中的 `postMessage` 会马上提交信息，但是小程序并不会马上受理，在小程序`webview`上的监听函数，只会在特定时机触发并收到消息：
也就是 `postMessage` 所有的消息都只能等得分享或 `webview` 的生命周期结束 才会被触发. 他是一个消息队列 :

```js
getMessage: function(e){
  if (e.type === 'message' && e.detail && e.detail.data && e.detail.data.length > 0) {
    e.detail.data.forEach(function (dataItem) {
      if (dataItem.type === 'qbreport' && dataItem.key) {
        // todo： yourFn(dataItem.key)
      }
    })
  }
}
```
**所以**, 我们在执行保存的时候 可以 立马 触发它的 返回 事件.
```js
function(){
  // 此处省略
  wx.miniProgram.postMessage({
    data: {
      xxx:'aaa'
    }
  })

  wx.miniProgram.navigateBack({ delta: 1 }) //注意这里.
}
```

### 完整的代码如下:
`html` 端代码:
```html
<html>
<title>webchat webview save image</title>
<header>
  <script src="https://res.wx.qq.com/open/js/jweixin-1.6.0.js"></script>
</hearder>
<body>
  <button id="saveImage" onclick="saveImage">下载图片</button>
  <script>  
    function saveImage(){
      let img = new Image()
      img.src = 'xxxx' //这里是图片的src
      img.crossOrigin = 'anonymous'  //The opeartaion is insecure ,其他跨域问题自行代理解决.
      img.onload = function () {
        let canvas = document.createElement('canvas')
        canvas.width = img.width
        canvas.height = img.height
        let ctx = canvas.getContext('2d')
        ctx.drawImage(this, 0, 0)
        let imgBase64Data = canvas.toDataURL('image/jpeg', 1)  //这里就拿到了base64

        wx.miniProgram.postMessage({
          data: {
            imgData: imgBase64Data // 刚才拿到的base64 数据
          }
        })

        wx.miniProgram.navigateBack({ delta: 1 }) //注意这里.
      }
    }
  </script>
</body>
</html>
```





小程序端代码:
```js
// wxml
<web-view  src="http://www.chuchur.com/save-image" bindmessage="getMessage"></web-view>

// js
Page({

  getMessage(e){
    let img =  e.detail.data[0].imgData

    wx.getFileSystemManager().writeFile({
      filePath: wx.env.USER_DATA_PATH + '/qrcode.jpeg',  //这里先把文件写到临时目录里. 
      data: img.slice(22), //注意这里
      encoding: 'base64',
      success: res => {
        console.log('success')
        wx.saveImageToPhotosAlbum({
          filePath: wx.env.USER_DATA_PATH + '/qrcode.jpeg', //这是把临时文件 保存到 相册, 收工
          success: res => {
            wx.showToast({ title: '保存成功！' })
          },
          fail: error => { console.log(error) }
        })
      },
      fail: error => { console.log(error) }
    })
  }
})

```

### 其它相关

#### 保存远程图片

```js
  wx.showLoading({ title: "正在下载图片...", mask: !1 })

  wx.downloadFile({
    url: '填写一个远程的图片链接',
    success: function (t) {
      wx.showLoading({ title: "正在保存图片", mask: !1 })
      wx.saveImageToPhotosAlbum({
        filePath: t.tempFilePath,
        success: function () {
          wx.showModal({ title: "自定义提示信息", content: "保存成功", showCancel: !1 });
        },
        fail: function (t) {
          wx.showModal({ title: "图片保存失败", content: t.errMsg, showCancel: !1 });
        },
        complete: function (t) { wx.hideLoading(); }
      });
    },
    fail: function (t) {
      wx.showModal({ title: "图片下载失败", content: t.errMsg, showCancel: !1 });
    },
    complete: function (t) { wx.hideLoading(); }
  })) 

```