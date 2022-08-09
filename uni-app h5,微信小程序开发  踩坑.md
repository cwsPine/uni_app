# uni-app h5,微信小程序开发  踩坑

## 1.组件设置全屏覆盖

- `设置宽度和高度100%是没用的` 需要在里面加设置一层view，并且设置对应宽高分别为vw和vh

- ```vue
  <uni-popup>
    <view class="wrap_test"> </view>
  </uni-popup>
  
  <style lang="scss">
    .wrap_test{
    width:100vw;
    height:100vh;
  }
  </style>
  
  ```



## 2.接口请求写在哪里好？（即返回页面不刷新问题）

- 可以写在onShow、onLoad、created、mounted

  - 通过uni.navigateTo不会销毁页面，返回时候页面的onLoad不会触发。
  - onshow 只要页面展示了就会请求。

- 需要通过onLoad的参数来发起请求怎么办？

  - ```js
    // 方案一
    onLoad(data) {
        this.prePageParams = data;  // 先把参数存起来
    },
    onShow() {
        this.request(this.prePageParams);  // 再拿到参数发起请求
    },
    ```

  - ```js
    // 方案二
    onShow() {
    	let params = this.$mp.query
        this.fetch(params);  // 再拿到参数发起请求
    },
    ```



## 3.uni-app微信小程序底部margin-bottom失效（有且只有ios系统上是失效的）

- 底部有个 position:fixed；区域，上方的 `view` 块为了不被挡住，设置 `margin-bottom` 会发现安卓机正常，ios手机 `view` 部分被 底部区域遮挡住。

- 解放方法

  - ```scss
    //使用 padding-bottom
    padding-bottom:100rpx;
    ```

  - ios 部分机型下方 存在横线，所以 padding-bottom 要把下方的安全区域算上

    - ```scss
      padding-bottom:calc(100rpx + constant(safe-area-inset-bottom));
       
      padding-bottom:calc(100rpx + env(safe-area-inset-bottom));
      ```



## 4.IOS苹果手机上时间格式化错误显示Invalid Date

- ios 时间格式只要带有`-`符号就会出现错误

- 解决方法

  - ```js
    let time = "2020-03-30 14:39"
    let TF = new Date(time.replace(/-/g,'/'))
    ```



## 5. 微信小程序自定义导航栏 需要考虑状态栏(时间电量那行) 刘海

- 自定义导航栏，代码书写的元素，跑到了状态栏上

- 解决方法

  - ```js
    // 得到状态栏高度
    let statusBarHeight =  uni.getSystemInfoSync()['statusBarHeight']
    
    // 在<templete>模板顶部中加一个空白的view设置高度即可
    <view :style="'height:'+status_bar_height+'px'"></view>
    ```

- 同理在设置吸顶的时候

  - **吸顶的top高度 = 自定义头部的高度 + 不同机型的刘海高度。**

  - ```vue
     <view :style="{ top: `calc(${statusBarHeight}px + 92rpx)` }" >
        这是一个吸顶盒子
     </view>
    ```



## 6.uni-popup遮罩层组件下页面会滚动问题

- 当打开遮罩层时候，去滑动底层页面，底层页面会滚动。

- 解决方法：

  - 需要给uni-popup加一层view，给他设置高度和 `overflow:hidden`，**注意需要在弹窗打开时候设置高度（否则会导致底层的页面高度有问题）**，并且再给它增加阻止冒泡事件：

  - ```vue
    <view :style="{ height: showModal ? '100vh' : '' }">
         <uni-popup
          ref="popupRef"
          type="bottom"
          @touchmove.stop.prevent="moveHandle"
        >
          <view class="wrap_popup"> 遮罩内容 </view>
       </uni-popup>
    </view>
          
    <script>
    export default {
      data() {
        return {
          showModal: false,
        };
      },
      methods: {
        openModal() {
          this.showModal = true; //弹窗打开前设置高度
          this.$refs.popupRef.open();
        },
        moveHandle() {
        
        }
      },
    };
    </script>
    ```



## 微信小程序不支持自定义的系统提示样式（弹出框，加载中）

- ```js
  uni.showModal({
      title: '12',
      content: '123',
      showCancel: true,
      success: res => {},
      fail: () => {},
      complete: () => {}
  });
  ```

- 在 App.vue 文件中设置样式只有h5能生效

  - ```scss
    uni-modal .uni-modal {
        border-radius: 24rpx;
    
        .uni-modal__bd {
            color: #222222;
            font-size: 30rpx;
        }
    }
    ```

  - 原生的组件 如 `button` `checkbox` ，可以在 App.vue 文件中设置， 微信小程序是生效的



## ios 调用 previewImage 接口不生效

- ios 调用 `uni.previewImage()` 预览图片失效

- 解决方法：

  - 在图片链接加上 `Content-Type：image/jpg`

  - ```js
    	uni.previewImage({
       		current: index,
       		urls: url + '?Content-Type=image/jpg',
       	})
    ```



## 微信开发者工具无法获取输入框的光标位置

- `input` 框在 focus 后调用 `getSelectedTextRange` 接口获取光标位置，H5 和 真机可以正常获取到，微信开发工具得到 `undefined`

- ```js
  uni.getSelectedTextRange({
      success: res => {
          console.log(res.start);
          console.log(res.end);
      },
      complete: (res) => {
          console.log(res);
      }
  })
  // H5 可以
  // 微信开发者工具显示undefined   真机测试可以
  ```



## 微信小程序的视口高度不包括底部原生tabBar和原生navgationBar

- 微信小程序

  - ```css
    height:100vh
    ```

  - 实际指向的高度：去除顶部导航栏 和 底部tab栏，剩下中间区域视图高度

- H5

  - ```scss
    height:100vh
    ```

  - 实际指向的高度：包括顶部导航栏 和 底部tab栏  全部区域



## ios 微信小程序底部黑块区域

- ios 部分机型（如iPhoneX ）底部会有黑色区域 挡住视图，添加 `padding-bottom`  适配底部安全区域，保证视图的呈现

- ```scss
  padding-bottom: constant(safe-area-inset-bottom); 
  padding-bottom:env(safe-area-inset-bottom);
  // calc 需要计算的情况下
  padding-bottom:calc(15rpx + constant(safe-area-inset-bottom));
  padding-bottom:calc(15rpx + env(safe-area-inset-bottom));
  ```



## weixin-js-sdk，解决H5微信小程序功能

- 微信内 h5页面没办法 配置微信分享朋友圈、微信支付等功能
- 解决：  微信JS-SDK是微信公众平台面向网页开发者提供的基于微信内的网页开发工具包。
  - 引入 `weixin-js-sdk`   `npm install weixin-js-sdk --save`
  
  - ```js
    import wx from "weixin-js-sdk";
    
    function initJssdk(shareCover,shareTitle,shareDetail) {
        uni.request()  // 请求后台得到 config 数据 info
        wx.config({
            // beta: info.beta, // 必须这么写，否则wx.invoke调用形式的jsapi会有问题
            debug: false, // 开启调试模式
            appId: info.appId, // 必填，微信的ID
            timestamp: info.timestamp, // 必填，生成签名的时间戳
            nonceStr: info.nonceStr, // 必填，生成签名的随机串
            signature: info.signature, // 必填，签名，见 附录-JS-SDK使用权限签名算法
            jsApiList: ['onMenuShareWeibo', 'updateAppMessageShareData', 'updateTimelineShareData', 'chooseImage',
                        "onMenuShareTimeline"
                       ] ||
            info.jsApiList, // 必填，需要使用的JS接口列表，凡是要调用的接口都需要传进来
            openTagList: ['wx-open-launch-weapp']
        })
    
        wx.ready(function() {
            //自定义“分享给朋友”及“分享到QQ”按钮的分享内容（1.4.0）
            wx.updateAppMessageShareData({
                title: shareTitle || shareDefaultTitle,
                desc: shareDetail || shareDefaultInfo,
                link: window.location.href,
                imgUrl: shareCover || shareDefaultUrl,
                trigger: function(res) {
                    // 不要尝试在trigger中使用ajax异步请求修改本次分享的内容，因为客户端分享操作是一个同步操作，这时候使用ajax的回包会还没有返回
                    //alert('用户点击发送给朋友');
                },
                success: function(res) {
                    console.log("分享朋友成功")
                },
                cancel: function(res) {
                    //alert('已取消');
                },
                fail: function(res) {
                    console.log(JSON.stringify(res))
                }
            })
            //分享到朋友圈 和 qq空间
            wx.updateTimelineShareData({
                title: shareTitle || shareDefaultTitle,
                link: window.location.href,
                imgUrl: shareCover || shareDefaultUrl,
                trigger: function(res) {
                    // 不要尝试在trigger中使用ajax异步请求修改本次分享的内容，因为客户端分享操作是一个同步操作，这时候使用ajax的回包会还没有返回
                    //alert('用户点击分享到朋友圈');
                },
                success: function(res) {
                    console.log("分享朋友圈成功")
                },
                cancel: function(res) {
                    //alert('已取消');
                },
                fail: function(res) {
                    console.log(JSON.stringify(res))
                }
            })
        }
    }
    ```
  















# Vue 踩坑

## 表单验证

- form表单进行多重校，有时候表单需要验证的属性层级比较深，就没办法直接写属性名

- ```vue
  <FormItem label="商品类目" prop="productCateDTO.productCateUuid" class="ivu-form-item-required">
      <Cascader :data="productCate" v-model="product.productCateDTO.productCateUuid" change-on-select trigger="hover" @on-change="chooseProductCate" ></Cascader>
  </FormItem>
  
  <script>
      rules:{
          // 将整个键值写成一个字符串
          'productCateDTO.productCateUuid': [
              { type:'array', required: true, message: "商品类目必填", trigger: "change" 			  },
          ],
      }
  </script>
  ```

- 表单验证类型

  - string (字符串/默认类型)
  2. number (数字)

  3. boolean (布尔类型)

  4. method (函数)

  5. float (浮点数)

  6. integer (整数)

  7. array (数组)

  8. object (对象)

  9. date (日期)

  10. url (URL类型)

  - email (电子邮件类型)





## 富文本信息url传参

- 编码

  - encodeURIComponent()  **富文本选用这个**
    encodeURIComponent() 函数可把字符串作为 URI 组件进行编码。
    注：该方法不会对 ASCII 字母和数字进行编码，也不会对这些 ASCII 标点符号进行编码： - _ . ! ~ * ’ ( ) 。其他字符（比如 ：;/?😡&=+$,# 这些用于分隔 URI 组件的标点符号），都是由一个或多个十六进制的转义序列替换的。
  - encodeURI()
    encodeURI() 函数可把字符串作为 URI 进行编码。
    注：对在 URI 中具有特殊含义的 ASCII 标点符号，encodeURI() 函数是不会进行转义的（如：, / ? : @ & = + $ # ).
    提示：使用 decodeURI() 方法可以解码URI（通用资源标识符：UniformResourceIdentifier,简称"URI")。
    语法：encodeURI(uri)-uri 必需。一个字符串，含有 URI 或其他要编码的文本

- 解码

  - decodeURI() 函数可对 encodeURI() 函数编码过的 URI 进行解码。
    使用 encodeURI() 函数可以对 URI 进行编码。
    语法： decodeURI(uri) - uri 必需。一个字符串，含有编码 URI 组件或其他要解码的文本。

  - decodeURIComponent(） **富文本选用这个**

    decodeURIComponent() 函数对 encodeURIComponent() 函数编码的 URI 进行解码。
    使用 encodeURIComponent() 方法可以对 URI 进行编码。
    语法：decodeURIComponent(uri) - uri 必需。一个字符串，含有编码 URI 组件或其他要解码的文本。





## Table 表单中 使用 column 配置

- 绑定的table的数据是tableData，input 框输入一个字符后，输入框主动失焦。

  - ```js
    {
        title: "价格",
            align: "center",
                key: "skuUnitPrice",
                    width: 100,
                        render: (h, params) => {
                            return h("InputNumber", {
                                props: {
                                    class: "ivu-input-number-input",
                                    min: 0.01,
                                    value: params.row.skuUnitPrice,
                                },
                                on: {
                                    "on-change": (e) => {
                                        this.skuData[params.index].skuUnitPrice = e;
    
                                    },
                                },
                            });
                        },
    },
    ```

- 解决方法：假如你绑定的table的数据是tableData，input数据改变的时候你把整行的数据替换掉，就不会造成table重新渲染，导致input失焦了

  - ```js
    {
        title: "价格",
            align: "center",
                key: "skuUnitPrice",
                    width: 100,
                        render: (h, params) => {
                            return h("InputNumber", {
                                props: {
                                    class: "ivu-input-number-input",
                                    min: 0.01,
                                    value: params.row.skuUnitPrice,
                                },
                                on: {
                                    "on-change": (e) => {
    								// this.skuData[params.index].skuUnitPrice = e;
                                        params.row.skuUnitPrice = e
                                        this.skuData[params.index] =  params.row;
    
                                    },
                                },
                            });
                        },
    },
    ```

    





