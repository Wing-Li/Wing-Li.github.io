---
title: 【小程序】小程序·云开发“微信支付”的代码
excerpt: 
categories:
  - topics
tag: 微信小程序 
---



### 云开发，云函数
getPay目录下共两个文件：
1､index.js
2､package.json

##### index.js代码如下：

    const key = "YOURKEY1234YOURKEY1234YOURKEY123"//这是商户的key，不是小程序的密钥。32位，必须大写。
    const mch_id = "1413090000" //你的商户号

    //将以上的两个参数换成你的，然后以下可以不用改一个字照抄

    const rp = require('request-promise')
    const crypto = require('crypto')

    function paysign({ ...args }) {
        let sa = []
        for (let k in args) sa.push( k + '=' + args[k])
        sa.push( 'key=' + key)
        return crypto.createHash('md5').update(sa.join('&'), 'utf8').digest('hex')
    }

    exports.main = async (event, context) => {
        const appid = event.userInfo.appId
        const openid = event.userInfo.openId
        const attach = event.attach
        const body = event.body
        const total_fee = event.total_fee
        const notify_url = "https://whatever.com/notify"
        const spbill_create_ip = "118.89.40.200"
        const nonce_str = Math.random().toString(36).substr(2, 15)
        const timeStamp = parseInt(Date.now() / 1000) + ''
        const out_trade_no = "otn" + nonce_str + timeStamp

        let formData = "<xml>"
        formData += "<appid>" + appid + "</appid>"
        formData += "<attach>" + attach + "</attach>"
        formData += "<body>" + body + "</body>"
        formData += "<mch_id>" + mch_id + "</mch_id>"
        formData += "<nonce_str>" + nonce_str + "</nonce_str>"
        formData += "<notify_url>" + notify_url + "</notify_url>"
        formData += "<openid>" + openid + "</openid>"
        formData += "<out_trade_no>" + out_trade_no + "</out_trade_no>"
        formData += "<spbill_create_ip>" + spbill_create_ip + "</spbill_create_ip>"
        formData += "<total_fee>" + total_fee + "</total_fee>"
        formData += "<trade_type>JSAPI</trade_type>"
        formData += "<sign>" + paysign({ appid, attach, body, mch_id, nonce_str, notify_url, openid, out_trade_no, spbill_create_ip, total_fee, trade_type: 'JSAPI' }) + "</sign>"
        formData += "</xml>"

        let res = await rp({ url: "https://api.mch.weixin.qq.com/pay/unifiedorder", method: 'POST',body: formData})
        let xml = res.toString("utf-8")
        if (xml.indexOf('prepay_id')<0) return
        let prepay_id = xml.split("<prepay_id>")[1].split("</prepay_id>")[0].split('[')[2].split(']')[0]
        let paySign = paysign({ appId: appid, nonceStr: nonce_str, package: ('prepay_id=' + prepay_id), signType: 'MD5', timeStamp: timeStamp })
        return { appid, nonce_str, timeStamp, prepay_id, paySign }
    }

##### package.json 代码如下：

    {
        "name": "getPay",
        "version": "1.0.0",
        "description": "",
        "main": "index.js",
        "scripts": {
            "test": "echo \"Error: no test specified\" && exit 1"
        },
        "author": "yourself",
        "license": "ISC",
        "dependencies": {
            "crypto": "^1.0.1",
            "request-promise": "^4.2.2"
        }
    }

选择：上传和部署：云端安装依赖

### 小程序端

    // 小程序支付参数必须是 int，所以这里虽然支付的是 0.01，但是也必须 *100 ，使其变为整型
    wx.cloud.callFunction({
        name: 'getPay' ,
        data: {
            total_fee: parseFloat(0.01).toFixed(2) * 100,
            attach: 'anything',
            body: 'whatever'
        }
    })
    .then( res => {
        wx.requestPayment({
            appId: res.result.appid,
            timeStamp: res.result.timeStamp,
            nonceStr: res.result.nonce_str,
            package: 'prepay_id=' + res.result.prepay_id,
            signType: 'MD5',
            paySign: res.result.paySign,
            success: res => {
                console.log(res)
            }
        })
    })

如此便可以支付了

***

#### 一些关于支付失败的

- 使用微信的在线签名工具检查签名是否和程序生成的一致，https://pay.weixin.qq.com/wiki/doc/api/micropay.php?chapter=20_1 签名工具用谷歌打开。
  选择MD5，XML,然后把请求参数xml放进去，就能校验签名。
- 如果和微信的在线签名工具一致，说明程序没有错误，确定是API密钥错误(被别人改动或者记错了)。
  在商户平台的账户信息中更改API密钥（账户设置-安全设置-API安全）, 15分钟后生效。
1. 统一下单用的是A商户号，也必须是A商户号登陆商户平台设置key才对。
2. 要注意统一下单请求参数中total_fee参数的类型是int类型。
- 如果和微信的在线签名工具不一致，说明程序有错误，常见的错误可能是:
1. 编码问题，确保所有的都是utf-8的. 如果有中文, 可以先把中文改成英文重新签名，看是否签名错误，如果英文不会错中文才会错，基本肯定是编码问题
2. 消息中字段大小写和文档中完全一致

