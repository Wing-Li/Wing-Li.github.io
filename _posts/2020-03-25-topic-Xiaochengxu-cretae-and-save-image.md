---
title: 生成二维码，
excerpt: 并且将二维码保存到本地
categories:
  - topics
tag: 微信小程序 
---


先说生成二维码的流程：

> 1. 调用服务器 API 的接口，获取二维码的流数据；(请求微信的 API，微信给返回的是流)；
> 2. 通过小程序提供的 arrayBufferToBase64 生成 base64 的图片流；
> 3. 通过 wx.getFileSystemManager() 将流转化成图片，保存到本地。

接下来说一下详细的步骤：

#### 一、 服务端请求微信的 API，获取二维码
这里用的是小程序的云开发，新建一个名为 getQRCode 的云函数，secret 在小程序后台的设置里面，必须设置，否则获取不到。

代码如下：

    // 小程序的 secret ，这里必须填
    const secret = 'your secret'

    const rp = require('request-promise')

    exports.main = async (event, context) => {
        let opt = {
            method: 'GET',
            url: 'https://api.weixin.qq.com/cgi-bin/token',
            qs: {
                appid: event.userInfo.appId,
                secret,
                grant_type: 'client_credential'
            },
            json: true
        }

        let res = await rp(opt)
        opt = {
            method: 'POST',
            url: 'https://api.weixin.qq.com/wxa/getwxacodeunlimit?access_token=' + res.access_token,
            body: {
                'page': event.page,
                'width': event.width,
                'scene': event.scene
            },
            json: true,
            encoding: null
        }

        return await rp(opt)
    }

#### 二、 请求服务端的代码，获取二维码的流
因为服务端用的是小程序的云开发，这里请求时用的是调用云函数。

代码如下：

    wx.cloud.callFunction({
        name: 'getQRCode',
        data: {
            scene: 'scene',
            page: 'pages/index/index',
            width: 180
        }
    }).then(res => {
        // res.result 二维码的流数据
        let qr = "data:image/png;base64," + wx.arrayBufferToBase64(res.result)
        // 如果想要直接在小程序里显示二维码，到了这里就可以显示出来了
        // 直接将 qr 设置给 <image /> 就可以将二维码显示出来
    })

#### 三、 将流转化成图片，保存到本地

    // wx.env.USER_DATA_PATH 可以获取小程序本地的储存路径
    var imgFileName = wx.env.USER_DATA_PATH + '/gameImage/xxx.png'

    // res.result 为请求服务端 二维码的流数据
    var imageData = wx.arrayBufferToBase64(res.result)

    const fileManager = wx.getFileSystemManager()
    fileManager.writeFile({
                filePath: imgFileName, // 本地文件目录
                data: imageData, // base64 数据
                encoding: 'base64',
                success(res) {
                  console.log(res)

                  // 保存成功之后，图片就会保存在 imgFileName 路径下
                  // imgFileName 路径下就会生成一张图片，二位就不再是流的形式了

                },
                fail(error) {
                  console.log(error)
                  wx.hideLoading()
                }
              })

如此便将二维码生成图片，并且保存到本地了。