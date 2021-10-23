传送门：
[借助小程序云开发实现小程序支付功能](https://developers.weixin.qq.com/community/develop/article/doc/000ceae09288489c0e9886e6c59c13)

先看效果图：

![](https://upload-images.jianshu.io/upload_images/6273713-fce2f4ffa8f92d99.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

我们实现这个支付功能完全是借助小程序云开发实现的，不用搭建自己的服务器，不用买域名，不用备案域名，不用支持https。只需要一个简单的云函数，就可以轻松的实现微信小程序支付功能。
核心代码就下面这些：

![](https://upload-images.jianshu.io/upload_images/6273713-7433fba3b792bb28.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 一、创建一个云开发小程序

关于如何创建云开发小程序，这里我就不再做具体讲解。不知道怎么创建云开发小程序的同学，可以去翻看腾讯云云开发公众号内菜单【技术交流-视频教程】中的教学视频。

#### 创建云开发小程序有几点注意的

1.一定不要忘记在app.js里初始化云开发环境。

![](https://upload-images.jianshu.io/upload_images/6273713-c436567c3368ac74.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

2.创建完云函数后，一定要记得上传

# 二、创建支付的云函数 

1.创建云函数pay

![](https://upload-images.jianshu.io/upload_images/6273713-32302ade305b8a18.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](https://upload-images.jianshu.io/upload_images/6273713-8ea47ffa0b4cffca.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 三、引入三方依赖tenpay

我们这里引入三方依赖的目的，是创建我们支付时需要的一些参数。我们安装依赖是使用里npm 而npm必须安装node,关于如何安装node，我这里不做讲解，百度一下，网上一大堆。

#### 1.首先右键pay，然后选择在终端中打开

![](https://upload-images.jianshu.io/upload_images/6273713-8881030499ebe5ce.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 2.我们使用npm来安装这个依赖。

在命令行里执行  npm i tenpay

![](https://upload-images.jianshu.io/upload_images/6273713-c61cb1cb5880c475.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](https://puui.qpic.cn/vupload/0/20190815_1565838407431_frsvtt3nzii.png/0)

![](https://upload-images.jianshu.io/upload_images/6273713-768712337485bf67.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

安装完成后，我们的pay云函数会多出一个package.json 文件

![](https://upload-images.jianshu.io/upload_images/6273713-7e9236d8983ebb21.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

到这里我们的tenpay依赖就安装好了。

# 四、编写云函数pay
![](https://upload-images.jianshu.io/upload_images/6273713-cd36f9084fada492.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

完整代码如下

```
//云开发实现支付
const cloud = require('wx-server-sdk')
cloud.init()

//1，引入支付的三方依赖
const tenpay = require('tenpay');
//2，配置支付信息
const config = {
  appid: '你的小程序appid', 
  mchid: '你的微信商户号',
  partnerKey: '微信支付安全密钥', 
  notify_url: '支付回调网址,这里可以先随意填一个网址', 
  spbill_create_ip: '127.0.0.1' //这里填这个就可以
};

exports.main = async(event, context) => {
  const wxContext = cloud.getWXContext()
  let {
    orderid,
    money
  } = event;
  //3，初始化支付
  const api = tenpay.init(config);

  let result = await api.getPayParams({
    out_trade_no: orderid,
    body: '商品简单描述',
    total_fee: money, //订单金额(分),
    openid: wxContext.OPENID //付款用户的openid
  });
  return result;
}
```

### 一定要注意把appid，mchid，partnerKey换成你自己的。

到这里我们获取小程序支付所需参数的云函数代码就编写完成了。

不要忘记上传这个云函数。

![](https://upload-images.jianshu.io/upload_images/6273713-ba99ca6fe33401ec.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

出现下图就代表上传成功

![](https://upload-images.jianshu.io/upload_images/6273713-6133d61bc300dac4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 五、写一个简单的页面，用来提交订单，调用pay云函数。

![](https://upload-images.jianshu.io/upload_images/6273713-ee974aecada48f7c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这个页面很简单：

1.自己随便编写一个订单号（这个订单号要大于6位）

2.自己随便填写一个订单价（单位是分）

3.点击按钮，调用pay云函数。获取支付所需参数。

下图是官方支付api所需要的一些必须参数。

![](https://upload-images.jianshu.io/upload_images/6273713-2708b7475409199b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

下图是我们调用pay云函数获取的参数，和上图所需要的是不是一样。

![](https://upload-images.jianshu.io/upload_images/6273713-d94c566dd744f128.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


# 六、调用wx.requestPayment实现支付

下图是官方的示例代码：

![](https://upload-images.jianshu.io/upload_images/6273713-00e9315590e4e14c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这里不在做具体讲解了，把完整代码给大家贴出来

```
// pages/pay/pay.js
Page({
  //提交订单
  formSubmit: function(e) {
    let that = this;
    let formData = e.detail.value
    console.log('form发生了submit事件，携带数据为：', formData)
    wx.cloud.callFunction({
      name: "pay",
      data: {
        orderid: "" + formData.orderid,
        money: formData.money
      },
      success(res) {
        console.log("提交成功", res.result)
        that.pay(res.result)
      },
      fail(res) {
        console.log("提交失败", res)
      }
    })
  },

  //实现小程序支付
  pay(payData) {
    //官方标准的支付方法
    wx.requestPayment({
      timeStamp: payData.timeStamp,
      nonceStr: payData.nonceStr,
      package: payData.package, //统一下单接口返回的 prepay_id 格式如：prepay_id=***
      signType: 'MD5',
      paySign: payData.paySign, //签名
      success(res) {
        console.log("支付成功", res)
      },
      fail(res) {
        console.log("支付失败", res)
      },
      complete(res) {
        console.log("支付完成", res)
      }
    })
  }
})
```

到这里，云开发实现小程序支付的功能就完整实现了。

# 实现效果

### 1.调起支付键盘

![](https://upload-images.jianshu.io/upload_images/6273713-b20becb49e6fd26e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 2.支付完成

![](https://upload-images.jianshu.io/upload_images/6273713-b2a8266fdc83edc3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 3.log日志，可以看出不同支付状态的回调

![](https://upload-images.jianshu.io/upload_images/6273713-3a1fca73b650742e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

上图是支付成功的回调，我们可以在支付成功回调时，改变订单支付状态。

下图是支付失败的回调：

![](https://upload-images.jianshu.io/upload_images/6273713-1b306a9b35b292e0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

下图是支付完成的状态：

![](https://upload-images.jianshu.io/upload_images/6273713-906f64407be62c4c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

到这里我们就轻松的实现了微信小程序的支付功能了，是不是很简单啊。

# 源码地址：

[https://github.com/qiushi123/cloud-pay](https://github.com/qiushi123/cloud-pay)
