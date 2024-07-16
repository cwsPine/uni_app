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
  











# **app开发**

## unipush2.0 推送消息

### 1. 开通配置unipush

- Hbuider 登录开发者中心
- ![](C:\Users\cws\Pictures\echarts\1717380399787.png)
- 服务空间不管有没有开通都需要开通，详细https://uniapp.dcloud.net.cn/unipush-v2.html#%E7%AC%AC%E4%B8%80%E6%AD%A5-%E5%BC%80%E9%80%9A
- 厂商推送设置按需要配置
- 客户端操作配置(离线推送建立在厂商配置完成之后)
  - 在`manifest.json`->`APP 模块配置`->`uniPush 2.0`配置界面勾选离线推送时启用的 sdk），获取到的 cid 的 phoneType 为 APP 类型。
  - 配置uniPush推送的小图标
- 云函数配置
  - 创建uniCloud开发环境
  - 新建云函数/对象
    - 选择uni-cloud-push扩展库，如果已有云函数则右键选择"管理公共模块或扩展库依赖"，选择uni-cloud-push扩展库。



### 2.uniCloud 业务代码（云函数代码）

- ```javascript
  
  // 简单的使用示例
  'use strict';
  const uniPush = uniCloud.getPushManager({appId:"__UNI__8FF979A"}) //注意这里需要传入你的应用appId，用于指定接收消息的客户端
  exports.main = async (event, context) => {
  	
  	console.log(event);
  	var data = JSON.parse(event.body) // 根据需求得到data数据
  	
  
  	const res =  await uniPush.sendMessage({
  		// "push_clientid": "67b38fa7a3c6680f27bd17e3206d4035", 
  		// "title":  "离线asfdas",
  		// "content": "离线asDA", 
  		// "payload": {
  		// 	"text": "离线asdasdsa",
  		// 	'url': 'https://knitting.oss-cn-heyuan.aliyuncs.com/shareLogo.png',
  		// },
  		// "platform":data.platform,
  		
          "force_notification":true,
  		"push_clientid": data.push_clientid, 	//填写上一步在uni-app客户端获取到的客户端推送标识push_clientid
  		"title":  data.title, 
  		"content":data.content, 
  		"payload": {
  			"text":data.payload,
  			'url':data.url
  		},
  		
  		"badge":+1, // app角标
  		
  		"settings":{
  			"schedule_time":data.scheduleTime
  		},
  		
  		
  		"channel":{
  			"HW":"NORMAL",
  		},
  		
  		"options": {
  		   "HW": {
  			"/message/android/category":"IM",
  		    "/message/android/notification/badge/class": "io.dcloud.PandoraEntry",
  		    "/message/android/notification/badge/add_num": 1,
  			"/message/android/notification/image": data.url,
  		  }
  		}
  
  		
  		// "option":{
  		// 	"HW":{
  				// "/message/android/category":"IM",
  		// 		"/message/android/notification/image":data.logoUrl,
  		// 		"/message/android/notification/style": 1,
  		// 		"/message/android/notification/big_title": data.title,
  		// 		"/message/android/notification/big_body": data.content,
  		// 	},
  		// 	"XM": {
  		// 		"/extra.notification_style_type": 1,
  		// 		"/extra.notification_large_icon_uri":data.logoUrl,
  		// 	},
  		// 	"OP": {
  		// 		"/small_picture_id": 1,
  		// 		"/extra.notification_bigPic_uri":data.oppoSmallPicId,
  		// 	}
  		// }
  	})
  	
  	console.log('uniPushRes',res);
  	
  	return {
  		code: 0,
  		message: '推送消息成功？',
  		data: res
  	}
  };
  ```

- 云函数推送消息

  - 服务器端调用

    - 在开发者中心中—`云函数/云对象`—`函数/对象列表`—`选择uni-push函数详情`—设置URL的PATH部分—提供url地址给服务端人员调用

  - 客户端直接调用

    ```js
    uni.getPushClientId({ // 获取设备cid
        success: (res) => {
            console.log(res);
            
            uniCloud.callFunction({
                name: 'uni-push', // 云函数名称
                data: { //传给云函数的参数
                    'cid': res.cid,
                    // title:'',
                },
            }).then(response => {
               
    
            }).catch(err => {
                // 处理错误
                console.log('错误', err)
            })
        },
        fail(err) {
            console.log(err)
        }
    })
    ```



### 3.客户端监听推送消息、清楚角标

- 监听推送消息

  ```js
  uni.onPushMessage((res) => {
      console.log(res)
  
      // var data = JSON.parse(res)
      if (res.type == 'click') {
          // 监听到消息被点击 进来跳转到对应页面
          uni.reLaunch({
              url: res.data.payload.url,
              success: (res) => {
                  console.log(res)
              },
              fail: (error) => {
                  console.log('error', error)
              }
          })
      }
  })
  
  ```

- 清楚APP角标

  ```js
  //清除角标
  // #ifdef APP-PLUS
  	plus.runtime.setBadgeNumber(0) // 安卓 ios 手机
  // application.applicationIconBadgeNumber = 0;
  
  	// ios 也可以调用云服务空间函数 setBadgeByCid
  	uniCloud.callFunction({
          name: 'uni-push-setbadge', // 云函数名称
          data: { //传给云函数的参数
              'cid': res.cid,
          },
      }).then(response => {
          console.log(response, '角标ios');
  
      }).catch(err => {
          // 处理错误
          console.log('错误', err)
      })
  // #endif
  ```
  
  ```js
  /* uni-push-setbadge.js */
  
  'use strict';
      const uniPush = uniCloud.getPushManager({appId:"__UNI__8FF979A"}) //注意这里需要传入你的应用appId，用于指定接收消息的客户端
      exports.main = async (event, context) => {
      	
      	console.log(event);
      	var data = event
      
      	const res =  await uniPush.setBadgeByCid({
      		// "cid": "ded6977206599e709f5ccec822310a34", 
      		
      		"cid":data.cid, 
      		"badge":'0'
      		
      	})
      	console.log(res);
      	return {
      		code: 0,
      		message: '设置应用角标',
      		data: res
      	}
      };
  ```
  





## 制作海报 包含二维码 （待补充）



## 全局弹窗

### 1.配置

- 在page.json文件中注册一个页面

  ```json
  {
      "path": "components/cvs-app-modal/cvs-app-modal",
      "style": {
          "navigationStyle": "custom",
          "backgroundColor": "transparent",
          "backgroundColorTop": "transparent",
          "backgroundColorBottom": "transparent",
          "app-plus": {
          "animationType": "fade-in",
          "background": "transparent",
          "popGesture": "none",
          "bounce": "none",
          "titleNView": false
          }
      }
  },
  ```

- 在main.js中引用并Vue.use进行注册

  ```js
  //main.js
  import globalPopup from '@/utils/globalPopup.js';
  Vue.use(globalPopup);
  
  /* globalPopup.js */
  // #ifdef H5
  // import globalPopup from './globalPopup.vue'
  // #endif
  const install = Vue => {
  	Vue.prototype.$popup = {
             //  可以注册多个组件 
  		show(params) {
  			let that = this;
  			
  			
  			// type: 1, // 弹窗类型（1、提交反馈弹窗 2、modal弹窗 3、自定义弹窗（设为此后其他参数无效））
  			// showCancel: false, // 是否显示取消按钮
  			// confirmText:'确定', // 确定按钮文字
  			// cancelText:'取消', // 取消按钮文字
  			// closeOpacity:false,// 点击遮罩是否关闭弹窗
  			// title: '', // 标题
  			// content: '' ,// 内容文本
  			// icon: 1,// 反馈弹窗状态 (1、成功 2、失败 3、正常)
  			// show: false,//是否显示
  			
  			uni.navigateTo({
  				url: '/components/cvs-app-modal/cvs-app-modal?params=' + JSON.stringify(params),
  				events:{
  					confirm: (data)=> {
  					 params.confirm && params.confirm()
  					},
  					cancel: (data)=> {
  					  params.cancel && params.cancel()
  					}
  				}
  			})
  			
  			// params.show =true;
  			// Vue.store 传递参数
  			// Vue.prototype.$store.dispatch('updatePopupConfig',params)
  			// // #ifdef APP-PLUS
  			// uni.navigateTo({
  			// 	url: '/components/globalPopup/globalPopup',
  			// 	events:{
  			// 		confirm: function(data) {
  			// 		  params.confirm && params.confirm()
  			// 		},
  			// 		cancel: function(data) {
  			// 		  params.cancel && params.cancel()
  			// 		}
  			// 	}
  			// })
  			// // #endif
  			// // #ifdef H5  // 参数依赖于vuex传递 按需修改  Vue.extend + vm.$mount 实现像js调用函数一样调用组件
  			// 	const PopupVue = Vue.extend(globalPopup);
  			// 	const popupDom = new PopupVue();
  			// 	popupDom.vm = popupDom.$mount();
  			// 	const lastEl = document.body.lastElementChild;
  			// 	if(lastEl.id !== 'popup-box'){
  			// 		setTimeout(()=>{
  			// 			document.body.appendChild(popupDom.vm.$el)
  			// 		})
              
  			// 	}
  			// // #endif
  		}
  	}
  }
  
  export default install;
  
  ```




### 2. 弹窗界面

```vue
<template>
	<view>
		<view class="dark">
			<view class="cvsMadal">
				<view class="cvsModalTitle" v-if="title">
					{{title}}
				</view>
				<view class="cvsModalContent">
					<text v-if="icon==2" class="bold">
						原因:
					</text>
					{{content}}
				</view>
				<view class="cvsModalBtn">	
					<view class="cvsModalCancel" @click="cancel" v-if="showCancel">
						{{cancelText}}
					</view>
					<view class="cvsModalConfirm" @click="confirm">
						{{confirmText}}
					</view>
				</view>
			</view>
		</view>
		
		<!-- <cvs-modal v-if="showModal" :title="modalTitle" :content="modalContent" :cancel="modalCancel"  @confirm="modalConfirm" ></cvs-modal>> -->
	</view>
</template>

<script src="./cvs-app-modal.js">
	export default {
		data() {
			return {
				eventChannel:null,
                title: '', // 标题
                content: '' ,// 内容文本
                showCancel: true, // 是否显示取消按钮
                confirmText:'确定', // 确定按钮文字
                cancelText:'取消', // 取消按钮文字
                // closeOpacity:false,// 点击遮罩是否关闭弹窗
                icon: 3,// 反馈弹窗状态 (1、成功 2、失败 3、正常)
                // show: false,//是否显示
			};
		},
        methods:{
            cancel(){
                let _this = this;
                uni.navigateBack({
                    delta:1,
                    success() {
                        _this.eventChannel.emit('cancel');
                    }
                })
                this.$emit('cancel')
            },

            confirm(){
                let _this = this;
                uni.navigateBack({
                    delta:1,
                    success() {
                        _this.eventChannel.emit('confirm');
                    }
                })

                this.$emit('confirm')
            }
        },
        onLoad(options){
            console.log(JSON.parse(options.params));

            // console.log(this);
            var obj = JSON.parse(options.params)
            this.title = obj.title ||  ''
            this.content = obj.content || ' '
            this.icon = obj.icon || 3
            this.cancelText = obj.cancelText || '取消'
            this.confirmText = obj.confirmText || '确定'
            if(obj.showCancel == false){
                this.showCancel = obj.showCancel
            }

            this.eventChannel = this.getOpenerEventChannel();

        }
	}
</script>

<style lang="scss">
    body{
        background-color: transparent;
    }
    .dark {
        @include dark;
        z-index: 200;
        @include flex_c_c;	
        // align-items: flex-end;
        .cvsMadal {
            width: 562rpx;
            background-color: #FFFFFF;
            border-radius: 24rpx;
            padding-top: 24rpx;
            // @include safeBottom(68rpx);

            .cvsModalTitle {
                // height: 104rpx;
                @include flex_c_c;	
                padding-top: 24rpx;
                font-size: 36rpx;
                font-weight: bold;
                color: #222222;
            }

            .cvsModalContent {
                @include flex_c_c;
                padding: 32rpx  24rpx;
                // height: 200rpx;
                font-size: 36rpx;
                color: #222222;
                border-bottom: 2rpx solid #F5F5F5;
            }
            .cvsModalBtn {
                // border-top: 2rpx solid #F1F1F1;
                @include flex_sb_c;
                // padding: 0 40rpx;
                // @include safeBottom(0);
                height: 94rpx;
                font-size: 36rpx;
                view {
                    @include flex_c_c;
                    flex: 1;
                    height: 94rpx;
                }

                .cvsModalCancel {
                    // border: 1px solid #DDDDDD;
                    color: #888;
                }
                .cvsModalConfirm {
                    color: $eColor;
                    // background: $eColor;
                }
            }
        }
    }
</style>

```





### 3.实际调用

```js
this.$popup.show({
     title: '',
    content: '',
    confirmText: '我知道了',
    showCancel: false,
    confirm: () => {
        // 回调
    }
})
```





## 检测APP更新

```js
this.$popup.update({
    title: `版本更新V${res.data.versionName}`,
    content: res.data.tips,
    hideCancel: data.uploadType == 1 ? false : true,
    cancelText: '下次再说',
    confirmText: '立即更新',
    confirm: (confirmRes) => {
        // 服务器
        if (data.uploadWay == 1) {
            // 服务器直接下载更新 
            if (brandName == 'apple') {
                // plus.runtime.openURL('https://www.baidu.com/')
                // 苹果 不支持服务器下载更新 所以都需要跳转到 appStore
                plus.runtime.openURL('itms-appss://apps.apple.com/cn/app/6480490706') // 地址填写app下载链接地址
                plus.ios.import("UIApplication").sharedApplication().performSelector(
                    "exit") // ios 退出APP

            } else {
                const downloadTask = uni.downloadFile({
                    url: data.android64Package, // Android 下载地址
                    success: (downloadResult) => {
                        console.log(downloadResult);
                        if (downloadResult.statusCode === 200) {
                            console.log('下载成功');
                            plus.runtime.install(downloadResult.tempFilePath, {
                                force: true
                            }, function() {
                                console.log('成功安装版本' + res.data
                                            .versionName)
                                // uni.showToast({
                                // 	title: '成功安装' + res.data
                                // 		.versionName,
                                // 	mask: true,
                                // 	duration: 2000
                                // });
                                setTimeout(() => {
                                    plus.runtime.restart(); // 跳到安装页面 重启app
                                }, 3000)
                            }, function(e) {
                                console.log("安装失败...")
                                uni.showToast({
                                    title: '安装失败...',
                                });
                            });
                        } else {
                            uni.navigateBack()
                            this.$popup.show({
                                // title: ,
                                content: '更新应用时遇到问题，建议您先卸载当前版本，然后前往应用商店下载最新版本。',
                                confirmText: '我知道了',
                                showCancel: false,
                                confirm: () => {
                                    plus.runtime.quit()
                                }
                            })
                        }
                    }
                });

                if(downloadTask){
                    // 下载进度全局弹窗
                    this.$popup.progress({

                    })
                }

                downloadTask.onProgressUpdate((res) => {
                    console.log('下载进度' + res.progress);
                    // this.globalData.updateProgress = res.progress
                    this.updateProgress(res.progress)

                    // console.log('已经下载的数据长度' + res.totalBytesWritten);
                    // console.log('预期需要下载的数据总长度' + res.totalBytesExpectedToWrite);

                    // 满足测试条件，取消下载任务。
                    // if (res.progress > 90) {
                    // 	console.log('取消了');
                    // 	downloadTask.abort();
                    // }
                });

            }
        } else if (data.uploadWay == 2) {
            // 打开各个厂商 app应用商店
            if (brandName == 'apple') {
                plus.runtime.openURL('itms-appss://apps.apple.com/cn/app/6480490706')
            } else if (brandName == "huawei") {
                plus.runtime.openURL(
                    'market://com.huawei.appmarket.applink?appId=C107999715')
            } else {
                plus.runtime.openURL('market://details?id=com.qinxiang.knitting')
            }
        }
    }
})
```



## app 分享 H5显示 <打开应用App>

- 客户端分享

  ```js
  // app 分享微信好友
  appShareSession() {
      console.log(this.share);
      console.log('href:', this.$online + '/' + this.share.path);
      uni.share({
          provider: "weixin",
          scene: "WXSceneSession",
          title: this.share.title,
          imageUrl: this.share.imageUrl,
  		
          // 0 分享打开是微信内嵌H5页面
          type:0,
          href: this.$online + this.share.path,
  
  		// type 5  分享出去是 小程序卡片
          // summary: this.share.title,
          // type: 5,
          // miniProgram: {
          // 	id: 'gh_094e9f2515d2',
          // 	path: this.share.path,
          // 	type: 0,
          // 	webUrl: this.share.path,
          // },
  
          success: function(res) {
              console.log("success:" + JSON.stringify(res));
          },
          fail: function(err) {
              console.log("fail:" + JSON.stringify(err));
          }
      });
  },
  ```

- 客户端打开组件

  ```vue
  <template>
  	
  	
  	
  		<view class="">
  			<view class="popDark" @touchmove.stop.prevent="" v-if="$wx_open">
  				<view class="popAppArea" :class="{'masterTopic':senceType==1,'mallTopic':senceType==2}">
  					<wx-open-launch-app appid="wx38d2915626e4a0b8" :extinfo="extinfo"  @ready="handleComponentReady" @error="handleLaunchError" @launch="handleSuccess" @click.stop="handleLaunchClick">
  						<script type="text/wxtag-template">
  							<style >
  								.btn {
  									display:flex;
  									justify-content: space-between;
  									align-items: center;
  									/* border-radius: 44rpx;
  									background-color: #4278FF;
  									width: 452rpx;
  									height: 88rpx;
  									box-shadow: 0rpx 8rpx 16rpx 2rpx rgba(0,0,0,0.16); */
  								}
  								
  								.eLoukong {
  									width: 96px;
  									height: 28px;
  									display:inline-block;
  									margin:8px 12px;
  								}
  								
  								.text {
  									display:inline-block;
  									font-weight: bold;
  									font-size: 18px;
  									color: #FFFFFF;	
  								}
  							</style>
  								<view class="btn">
  									<image class="eLoukong" src="https://knitting.oss-cn-heyuan.aliyuncs.com/shareLogo.png" mode="aspectFit"></image>
  									<view class="text">
  										App 内打开
  									</view>
  								</view>
  						</script>
  					</wx-open-launch-app>
  				</view>
  			</view>
  		</view>
  	
  </template>
  
  <script src="./cvs-app-awaken.js">
  	export default {
          props: {
              // 携带信息 用于打开指定页面
              extinfo:{
                  default:"",
              },
              // 场景 0 针织e家  1 师傅  2 商城 // 换主题色的
              senceType:{
                  default:0
              }
  
          },
  		data() {
  			return {
  				id: 'wxopenLanchAppId' ,
                  appId: 123,
                  // globalConfig.WEIXIN_APP_ID
                  enable: false,
                  extinfoStr: '',
                  isLaunchLoad:false,
                  launchReady:true,
  			};
  		},
          methods:{
              handleComponentReady(data) {
                // 组件是否加载成功
                  this.isLaunchLoad = true;
                  console.log('handleComponentReady:', data);
              },
              handleLaunchError(err) {
                  console.error('error:', err);
                  this.launchReady = false;
                  // 标签初始化失败
                  // 原方式跳转下载页面
                  this.waitJump();
              },
              // 走app store或者腾讯应用宝
              waitJump() {
                  console.log('跳转外链下载');
  
  
                  uni.getSystemInfo({
                      success: (res) => {
                          console.log(res.osName)
  
                          if (res.osName == 'android') {
                              // this.$u.toast(`android`)
                              window.location.href = "https://sj.qq.com/appdetail/com.qinxiang.knitting?source=gamedetail"
                          } else {
                              // phoneType = 2
                              // this.$u.toast(`ios`)
                              window.location.href = "itms-appss://apps.apple.com/cn/app/6480490706"
                          }
                      }
                  })
  
                  // window.location.href = 'https://www.baidu.com/'
                  // if (this.isAdr) {
                  //     console.log('走到android下载地址');
                  // } else {
                  //     console.log('走到ios下载地址');
                  //     window.location.href ='https://www.baidu.com/'
                  // }
              },
              handleSuccess(data) {
                  console.log('handleSuccess:', data);
              },
              // 下载APP按钮-------组件回调
              handleLaunchClick(data) {
                // 点击唤起APP 传参给予客户端
                // launch组件没有加载出来则点击，就跳转下载页面
                if (!this.isLaunchLoad) {
                    this.waitJump();
                }
                if (!this.launchReady) {
                    this.waitJump();
                }
              }
  		
          }
  	}
  </script>
  
  <style lang="scss">
  	@import "./cvs-app-awaken.scss"
  </style>
  
  ```

- 实际使用时在页面中引用改组件



## 手机号一键登录

- **`manifest.json` **文件中-> `APP模块配置` -> `OAuth(登录鉴权)`->`勾选 一键登录（univerify）`

- uniCloud 云函数

  - 需要的云函数扩展库：uni-cloud-jql、uni-cloud-verify

  - 云函数

    ```js
    'use strict';
    const crypto = require('crypto')
    exports.main = async (event, context) => { //event为客户端上传的参数
    	console.log('evenzt : ', event, 5);
    	console.log('参数', event.queryStringParameters);
    	// event里包含着客户端提交的参数
    	const res = await uniCloud.getPhoneNumber({
    		appid: '__UNI__8FF979A', // 替换成自己开通一键登录的应用的DCloud appid，使用callFunction方式调用时可以不传（会自动取当前客户端的appid），如果使用云函数URL化的方式访问必须传此参数     		 provider: 'univerify',
    		provider: 'univerify',
    		apiKey: '8dab5aa4283fc1079c833ab13c3e597a', // 在开发者中心开通服务并获取apiKey
    		apiSecret: 'c722719775d0bb15a4dedc15fab252be', // 在开发者中心开通服务并获取apiSecret
    		access_token: event.access_token,
    		openid: event.openid
    	})
    	console.log('res', res); // res里包含手机号
    	// 执行用户信息入库等操作，正常情况下不要把完整手机号返回给前端
    	// 如果数据库在uniCloud上，可以直接入库
    	// 如果数据库不在uniCloud上，可以通过 uniCloud.httpclient API，将手机号通过http方式传递给其他服务器的接口，详见：https://uniapp.dcloud.net.cn/uniCloud/cf-functions?id=httpclient
    
    	return {
    		code: 0,
    		message: '获取手机号成功',
    		data: res
    	}
    };
    ```

- 一键登录 预登录

  - ```js
    import {
    	return_page
    } from "@/utils/common/common.js"
    
    
    export default {
    	/**     * 一键登录预登录检查     * @return {boolean} 是否支持一键登录       */
    	pre_login() {
    		uni.getProvider({
    			//获取可用的服务提供商            
    			service: 'oauth',
    			success: function(res) {
    				console.log(res.provider) // ['weixin', qq', 'univerify']           
    			},
    		});
    		return new Promise((resolve, reject) => {
    			uni.preLogin({
    				//预登录                
    				provider: 'univerify', //用手机号登录                
    				success() {
    					console.log('预登录成功,16')
    					resolve(true)
    				},
    				fail(err) {
    					//预登录失败                    
    					console.log(`预登录失败(${err.errCode})`, err.errMsg)
    					resolve(false)
    				}
    			})
    		})
    	},
    	/**  * 本机号码一键登录     */
    	async fast_login() {
    		return new Promise((resolve, reject) => {
    			uni.login({
    				//正式登录，弹出授权窗                
    				provider: 'univerify',
    				univerifyStyle: {
    					// 自定义登录框样式                    
    					"fullScreen": true, // 是否全屏显示，true表示全屏模式，false表示非全屏模式，默认值为false。                    
    					"backgroundColor": "#ffffff", // 授权页面背景颜色，默认值：#ffffff                    
    					"phoneNum": {
    						"color": "#000000", // 手机号文字颜色 默认值：#000000                       
    					},
    					"authButton": {
    						"normalColor": "#3479f5", // 授权按钮正常状态背景颜色 默认值：#3479f5                          
    						"highlightColor": "#2861c5", // 授权按钮按下状态背景颜色 默认值：#2861c5（仅ios支持）                          
    						"disabledColor": "#73aaf5", // 授权按钮不可点击时背景颜色 默认值：#73aaf5（仅ios支持） 
    						"textColor": "#ffffff", // 授权按钮文字颜色 默认值：#ffffff                          
    						"title": "本机号码一键登录", // 授权按钮文案 默认值：“本机号码一键登录”                     
    					},
    					"privacyTerms": {
    						"defaultCheckBoxState": false, // 条款勾选框初始状态 默认值： true
    						"isCenterHint" :true,
    						"uncheckedImage": "/static/img/check.png", // 可选 条款勾选框未选中状态图片（仅支持本地图片 建议尺寸 24x24px）(3.2.0+ 版本支持)   
    						"checkedImage": "/static/img/checked_blue.png", // 可选 条款勾选框选中状态图片（仅支持本地图片 建议尺寸24x24px）(3.2.0+ 版本支持)   
    						"checkBoxSize": 16, // 可选 条款勾选框大小，仅android支持
    						"textColor": "#BBBBBB", // 文字颜色 默认值：#BBBBBB  
    						"termsColor": "#5496E3", //  协议文字颜色 默认值： #5496E3  
    						"prefix": "我已阅读并同意", // 条款前的文案 默认值：“我已阅读并同意”  
    						"suffix": "并使用本机号码登录", // 条款后的文案 默认值：“并使用本机号码登录”  
    						"privacyItems": [ // 自定义协议条款，最大支持2个，需要同时设置url和title. 否则不生效  
    							{
    								"url": "https://app.kintting.com/packageMsg/page/mine_aboutUs/agreement/agreement", // 点击跳转的协议详情页面  
    								"title": "《用户协议》" // 协议名称 
    							},
    							{
    								"url": "https://app.kintting.com/packageMsg/page/mine_aboutUs/privacy/privacy", // 点击跳转的协议详情页面  
    								"title": "《隐私政策》" // 协议名称  
    							}
    						]
    					},
    					// "buttons": {
    					    // "iconWidth": "30px",
    					    // "list": [
    					        // {
    					        //     "provider": "sinaweibo",
    					        //     "iconPath": "/static/sina.png"
    					        // }, 
    					        // {
    					        //     "provider": "weixin",
    					        //        "iconPath": "/static/img/wxIcon.png"
    					        // }, 
    					        // {
    					        //     "provider": "qq",
    					        //        "iconPath": "/static/qq.png"
    					        // }
    					       // ]
    					    // },
    				},
    				success(res) {
    					// 正式登录成功  
    					console.log(111)
    					console.log(res.authResult, 50);
    					// uni.showModal({
    					// 	title: '系统提示',
    					// 	content: '正式登录成功',
    					// 	showCancel: false,
    					// 	cancelText: '',
    					// 	confirmText: '是',
    					// 	success: res => {},
    					// 	fail: () => {},
    					// 	complete: () => {}
    					// });
    					//  {
    					//   openid:'登录授权唯一标识',
    					//   access_token:'接口返回的 token'
    					// }                    
    					resolve(res.authResult)
    					// 在得到access_token后，通过callfunction调用云函数              
    					// uniCloud.callFunction({
    					// 	name: 'login', // 云函数名称                    
    					// 	data: { //传给云函数的参数                    
    					// 		'access_token': res.authResult
    					// 			.access_token, //客户端一键登录接口返回的access_token                    
    					// 		'openid': res.authResult
    					// 			.openid // 客户端一键登录接口返回的openid                    
    					// 	},
    					// 	success(callRes) {
    					// 		console.log('调用云函数成功' + callRes + '获取手机号成功')
    
    
    					// 	},
    					// 	fail(callErr) {
    					// 		console.log('调用云函数出错' + callErr)
    					// 	},
    					// 	complete() {
    					// 		uni.closeAuthView() //关闭授权登录界面                   
    					// 	}
    					// })
    					// uni.closeAuthView() //关闭授权登录界面               
    				},
    				fail(err) {
    					uni.showModal({
    						title: '系统提示',
    						content: err,
    						showCancel: false,
    						cancelText: '',
    						confirmText: '',
    						success: res => {},
    						fail: () => {},
    						complete: () => {}
    					});
    					// 正式登录失败                    
    					console.log(`一键登录失败(${err.errCode})`, err.errMsg)
    					if(err.errCode == 30003){
    						
    						let router = this.$router
    						return_page(router)
    					}else if(err.errCode==30002){
    						uni.closeAuthView()
    					}
    					// console.log(err.errCode);
    						
    					reject(err)
    					// uni.closeAuthView() 
    					//关闭授权登录界面              
    				}
    			})
    		})
    	}
    }
    
    
    ```

- 页面调用

  - ```js
    
    async onLoad(){
    	const can_login = await service.pre_login() // 预登录 
        if (can_login) {
            // 当预登录允许 才支持手机号一键登录
        	this.toLogin()
        }
    }
    
    
    
    methods :{
        // 还未登录 去登录
    		async toLogin() {
    			// uni.navigateTo({
    			// 	url: "/login_package/pages/login/login"
    			// })
    			var that = this
    
    		
    
    			try {
    				// const { access_token, openid } = await service.fast_login()
    				// console.log("uniapp一键登录", access_token, openid, 29) // 登录操作，获取token和用户信息等操作        
    				
    				const loginRes = await service.fast_login()
    				console.log(loginRes);
    				// uni.showModal({
    				// 	title: '登录参数',
    				// 	content: loginRes,
    				// 	showCancel: false,
    				// 	cancelText: '',
    				// 	confirmText: '是',
    				// 	success: res => {},
    				// 	fail: () => {},
    				// 	complete: () => {}
    				// });
    				uniCloud.callFunction({
    					name: 'login', // 云函数名称                    
    					data: { //传给云函数的参数                    
    						'access_token': loginRes.access_token, //客户端一键登录接口返回的access_token                    
    						'openid': loginRes.openid // 客户端一键登录接口返回的openid                    
    					},
    				}).then(res => {
    					console.log(res.result.data.phoneNumber);
    					// uni.showModal({
    					// 	title: '拿到电话',
    					// 	content: '',
    					// 	showCancel: false,
    					// 	cancelText: '',
    					// 	confirmText: '是',
    					// 	success: res => {},
    					// 	fail: () => {},
    					// 	complete: () => {}
    					// });
    					console.log(this);
    				
    					that.appTelLogin(res.result.data.phoneNumber) // 拿到手机号后 请求后端接口
    					// 登录成功，可以关闭一键登录授权界面了
    				}).catch(err => {
    					// 处理错误
    					console.log('错误',err)
    				})
    				
    			} catch (e) {
    				console.log('一键登录失败', e)
    			}
    
    		},
    }
    
    
    ```



## 苹果登录

- **`manifest.json` **文件中-> `APP模块配置` -> `OAuth(登录鉴权)`->`勾选苹果登录（sign in with apple）`

- ```js
  // 苹果登录
  AppleLogin(){
  
      uni.login({
          provider: 'apple',
          success:  (loginRes)=> {
              console.log(loginRes);
              // 登录成功
              uni.getUserInfo({
                  provider: 'apple',
                  success: (info)=> {
                      console.log(info);
                      this.appleLoginApi(info.userInfo) // 请求后台
                      // 获取用户信息成功, info.authResult中保存登录认证数据
                  }
              })
          },
          fail:  (err)=> {
              console.log(err);
              this.$u.toast(err.errMsg)
              // 登录授权失败	
              // err.code错误码参考`授权失败错误码(code)说明`
          }
      });
  },
  ```













## socket 聊天总结

### 1.注册socket 连接

- App.vue 

  ```js
  onLaunch:function(){
      var tokenId = uni.getStorageSync('userToken');
  
      // 商城
      if (tokenId) {
          // 使用token就可以查询用户信息
          this.$myRequest("POST", "/api/user/getUserInfo", {
              sourceType: 'APP'
          }).then(loginRes => {
              console.log(loginRes.data.nettyToken);
              if (loginRes.code == 0) {
                  uni.setStorage({
                      key: "c_userInfo",
                      data: loginRes.data
                  })
  
                  var nettyToken = loginRes.data.nettyToken;
                  var nettyAccountUuid = loginRes.data.nettyAccountUuid + ',' + loginRes.data.mallUser
                  .nettyAccountDTO.nettyAccountUuid
                  uni.setStorageSync('nettyAccountUuid', nettyAccountUuid);
                  uni.setStorageSync('nettyToken', nettyToken);
  
                  //触发socket连接
                  this.$store.dispatch('initNimSDK', loginRes.data.mallUser)
  
                  loginRes.data.mallUser.avatar = loginRes.data.avatar
                  this.login(loginRes.data.mallUser); //将用户信息保存起来
                  this.$isResolve()
              } else {
                  console.log('同步用户信息失败, tokenId: ' + tokenId);
              }
          })
      }
}
  
  ```
  



### 2. store文件夹 vuex

- index.js

  ```js
  
  import {
  	getUsers,
  	getUser,
  	addFriend,
  	deleteFriend,
  	sendMsg,
  	sendFileMsg,
  	getHistoryMsgs, // 查询历史消息
  	getTopMsgs, // 置顶通知栏消息
  	sendMsgReceipt,
  
  	sendRobotMsg,
  	revocateMsg,
  	updateLocalMsg,
  	resetNoMoreHistoryMsgs,
  	continueRobotMsg,
  	// session
  	setCurrSession,
  	resetCurrSession,
  	
  } from './msgs'
  import {
  	initNimSDK,
  	getIMSessions
  } from './initNimSDK'
  const socketUrl = 'wss://apimall.kintting.cn/ws'  
  
  
  state:{
      	nim: null,
  		IMfriends: [],
  		friendslist: [],
  		currSessionId: '', // 聊天室id
  		currSessionMsgs: [], // 临时 聊天室 消息
  		currSessionLastMsg: '', // 记录历史消息的最后一条消息 用来做 消息截断
  		sessionNewMsg: '',
  
  		noMoreHistoryMsgs: 0,
  		sessionTopNocticeNewMsg:"", // 聊天室 置顶通知栏消息
  
  }，
  
  mutation:{
      connetSocket(state, payload) {
  			state.nim = null
  			// netty
  			const nettyToken = uni.getStorageSync('nettyToken');
  			const nettyAccountUuid = uni.getStorageSync('nettyAccountUuid');
  			
  			console.log(nettyToken);
  			// const userInfo = uni.getStorageSync('userInfo');
  			// console.log(userInfo);
  			// let url = socketUrl + `?accId=${userInfo.nettyAccountDTO.nettyAccountUuid}&token=${nettyToken}`
  			let url = socketUrl + `?accId=${nettyAccountUuid}&token=${nettyToken}`
  			state.nim = uni.connectSocket({
  				url: url, //仅为示例，并非真实接口地址。
  				complete: (complete) => {
  					console.log(complete);
  				}
  			});
  		},
  
  		// 好友用户资料更新
  		updateUserInfo(state, users) {
  			state.IMfriends = JSON.parse(JSON.stringify(users))
  			state.IMfriends.forEach(r => {
  				if (r.unread > 0) {
  					uni.showTabBarRedDot({
  						index: 3,
  						success: (e) => {
  							console.log(e);
  						}
  					})
  				}
  			})
  		},
  
  		updateFriends(state, friends, cutFriends = []) {
  			// const nim = state.nim
  			console.log(friends);
  			state.friendslist = friends
  			// state.friendslist = nim.mergeFriends(state.friendslist, friends)
  			// state.friendslist = nim.cutFriends(state.friendslist, cutFriends)
  			// state.friendslist = nim.cutFriends(state.friendslist, friends.invalid)
  		},
  
  		updateSessions(state, sessions) {
  			const nim = state.nim
  			state.sessionlist = nim.mergeSessions(state.sessionlist, sessions)
  			state.sessionlist.sort((a, b) => {
  				return b.updateTime - a.updateTime
  			})
  			state.sessionlist.forEach(item => {
  				state.sessionMap[item.id] = item
  			})
  		},
  
  		// putMsg(state, msg) {
  		// 	let sessionId = msg.sessionId
  		// 	if (!state.msgs[sessionId]) {
  		// 		state.msgs[sessionId] = []
  		// 	}
  		// 	store.commit('updateMsgByIdClient', msg)
  		// 	let tempMsgs = state.msgs[sessionId]
  		// 	let lastMsgIndex = tempMsgs.length - 1
  		// 	if (tempMsgs.length === 0 || msg.time >= tempMsgs[lastMsgIndex].time) {
  		// 		tempMsgs.push(msg)
  		// 	} else {
  		// 		for (let i = lastMsgIndex; i >= 0; i--) {
  		// 			let currMsg = tempMsgs[i]
  		// 			if (msg.time >= currMsg.time) {
  		// 				state.msgs[sessionId].splice(i, 0, msg)
  		// 				break
  		// 			}
  		// 		}
  		// 	}
  		// },
  
  
  
  		// 没有更多历史消息
  		setNoMoreHistoryMsgs(state) {
  			state.noMoreHistoryMsgs = 2;
  			uni.hideLoading()
  		},
  		// 重置
  		resetNoMoreHistoryMsgs(state) {
  			state.noMoreHistoryMsgs = 0;
  			state.sessionNewMsg = ''
  		},
  
  		// 修改 推入消息 状态
  		updatePutMsg(state, payload) {
  			state.sessionNewMsg = payload
  		},
  
  
  
  		// 记录当前 聊天室的id
  		updateCurrSessionId(state, obj) {
  			let type = obj.type || ''
  			if (type === 'destroy') {
  				state.currSessionId = null
  			} else if (type === 'init') {
  				if (obj.sessionId && (obj.sessionId !== state.currSessionId)) {
  					state.currSessionId = obj.sessionId
  				}
  			}
  		},
  		
  		updateCurrSessionTopNoticeMsg(state, obj) {
  			let type = obj.type || ''
  			
  			if (type === 'put') { // 清空会话消息
  				let newMsg = obj.msg
  				if(newMsg && newMsg.content){
  					newMsg.content = JSON.parse(newMsg.content)
  					state.sessionTopNocticeNewMsg = newMsg.content
  				}else{
  					state.sessionTopNocticeNewMsg = ''
  				}
  			}
  		},
  
  		// 更新 聊天室消息
  		updateCurrSessionMsgs(state, obj) {
  			let type = obj.type || ''
  			if (type === 'destroy') { // 清空会话消息
  				state.currSessionMsgs = []
  				state.currSessionLastMsg = null
  				store.commit('updateCurrSessionId', {
  					type: 'destroy'
  				})
  			} else if (type === 'concat') {
  				let currSessionMsgs = []
  				let lastMsgTime = 0
  				obj.msgs.forEach(msg => {
  					if ((msg.time - lastMsgTime) > 1000 * 60 * 5) {
  						lastMsgTime = msg.time
  						currSessionMsgs.push({
  							type: 'timeTag',
  							time: msg.time
  						})
  					}
  					currSessionMsgs.push(msg)
  				})
  				// currSessionMsgs = obj.msgs
  				currSessionMsgs.reverse()
  				currSessionMsgs.forEach(msg => {
  					state.currSessionMsgs.unshift(msg)
  				})
  				if (obj.msgs[0]) {
  					state.currSessionLastMsg = obj.msgs[0]
  				}
  				console.log(currSessionMsgs);
  				// store.dispatch('checkTeamMsgReceipt', currSessionMsgs)
  			} else if (type === 'put') { // 追加一条消息
  				let newMsg = obj.msg
  				let lastMsgTime = 0
  				let lenCurrMsgs = state.currSessionMsgs.length
  				if (lenCurrMsgs > 0) {
  					lastMsgTime = state.currSessionMsgs[lenCurrMsgs - 1].time
  				}
  				if (newMsg) {
  					if ((newMsg.time - lastMsgTime) > 1000 * 60 * 5) {
  						state.currSessionMsgs.push({
  							type: 'timeTag',
  							// text: util.formatDate(newMsg.time, false)
  							time: newMsg.time
  						})
  					}
  					state.currSessionMsgs.push(newMsg)
  					state.sessionNewMsg = newMsg
  					// store.dispatch('checkTeamMsgReceipt', [newMsg])
  				}
  				// state.sessionsonNewMsg = newMsg
  			}
  
  			uni.hideLoading()
  		}
  
  
  },
      
      
  actions:{
      
  		// 商城 消息
  		initNimSDK,
  		getIMSessions,// 获取 消息列表人
  		getHistoryMsgs,
  		getTopMsgs, // 获取 置顶通知消息
  		getUsers,
  		getUser, // 获取自己的资料
  		addFriend, // 加好友
  		deleteFriend, // 删除好友
  		sendMsg,
  		sendFileMsg,
  		sendMsgReceipt, // 消息已读回执
  		
  		// session
  		setCurrSession, // 设置聊天室id
  		resetCurrSession, // 重置聊天室id
  }
  
  ```

- initNimSDK.js

  ```js
  import store from './index.js'
  import app from '@/main.js'
  // #ifdef H5
  // import NIM from '@yxim/nim-web-sdk/dist/SDK/NIM_Web_NIM.js'
  // import NIM from '../NIM_Web_NIM_miniapp.js'
  // import NIM from 'https://mall.kintting.com/NIM_Web_NIM_miniapp.js'
  // #endif
  
  // #ifdef MP-WEIXIN
  // import NIM from '@yxim/nim-web-sdk/dist/SDK/NIM_Web_NIM_miniapp.js'
  // #endif
  
  import {
  	onMsg
  } from './msgs.js'
  
  // 好友用户资料卡
  function onUserInfo(users) {
  	console.log('收到用户资料列表', users);
  	if (!Array.isArray(users)) {
  		users = [users]
  	}
  	// users = users.map(formatUserInfo)
  	// store.commit('updateUserInfo', users)
  	getIMSessions()
  }
  
  export function getIMSessions() {
  
  
  	// IM云信
  	// let postData = {
  	// 	keyArray:[]
  	// }
  
  	// app.$api.request.getIMSessions(postData, res => {
  	// 	if (res.body.status.statusCode === '0') {
  	// 		// 轮播图
  	// 		// console.log(res);
  	// 		let users = res.body.data.sessions
  	// 		store.commit('updateUserInfo', users)
  	// 	} else {
  	// 		console.log(res.body.status.errorDesc);
  	// 	}
  	// });
  
  
  
  	// netty
  	const nettyToken = uni.getStorageSync('nettyToken');
  	var message = {
  		code: 104,
  		token: nettyToken,
  	}
  	
  	var encrypted = app.$api.util.encryptPwd(message)
  
  	store.state.nim.send({
  		data: encrypted.toString(),
  		complete: (complete) => {
  			console.log('发消息获取会话列表', complete);
  			// store.commit('updateUserInfo', users)
  		}
  	})
  
  
  }
  
  // 好友关系，回调
  function onFriends(friends) {
  	// console.log('收到好友列表', friends);
  	friends = friends.map(item => {
  		if (typeof item.isFriend !== 'boolean') {
  			item.isFriend = true
  		}
  		return item
  	})
  	// console.log(friends);
  	// store.commit('updateFriends', friends)
  	// 更新好友信息字典，诸如昵称
  	// store.commit('updateUserInfo', friends)
  }
  
  // // 收到消息 回调
  // function onMsg(msg) {
  // 	console.log('收到消息', msg.scene, msg.type, msg);
  // 	// pushMsg(msg);
  // }
  
  function onRoamingMsgs(obj) {
  	console.log('收到漫游消息', obj);
  	// pushMsg(obj.msgs);
  }
  
  function onOfflineMsgs(obj) {
  	console.log('收到离线消息', obj);
  	// pushMsg(obj.msgs);
  }
  
  
  // onSessions只在初始化完成后回调
  function onSessions(sessions) {
  	console.log('会话列表' + sessions);
  	console.log(sessions);
  	// updateSessionAccount(sessions)
  	// store.commit('updateSessions', sessions)
  }
  
  function onUpdateSession(res) {
  	console.log('更新会话列表' + res);
  	// console.log(res);
  	// updateSessionAccount(sessions)
  	// store.commit('updateSessions', sessions)
  }
  
  // 如果会话对象不是好友，需要更新好友名片
  function updateSessionAccount(sessions) {
  	let accountsNeedSearch = []
  	sessions.forEach(item => {
  		if (item.scene === 'p2p') {
  			// 如果不存在缓存资料
  			if (!store.state.userInfos[item.to]) {
  				accountsNeedSearch.push(item.to)
  			}
  		}
  	})
  	if (accountsNeedSearch.length > 0) {
  		store.dispatch('searchUsers', {
  			accounts: accountsNeedSearch
  		})
  	}
  }
  
  
  
  		
  
  // 重新初始化 NIM SDK
  export function initNimSDK({
  	state,
  	commit,
  	dispatch
  }, loginInfo) {
  	// console.log(NIM);
  
  	var timer
  	var interTimer
  	
  		commit('connetSocket')
  
  
  	state.nim.onOpen(res => {
  		// console.log(res);
  		console.log('WebSocket连接已打开！');
  		clearInterval(timer)
  		// 获取会话列表
  		getIMSessions()
  	})
  	
  	
  
  	state.nim.onError(res => {
  		console.log(res);
  		console.log('WebSocket连接打开失败，请检查！');
  	})
  	
  	state.nim.onClose(res => {
  		console.log(res);
  		console.log('WebSocket关闭');
  		
  		var resetSocketNumber =  uni.getStorageSync('resetSocket')
  		uni.setStorageSync('resetSocket',resetSocketNumber+1)
  		// uni.setStorageSync('resetSocket',0)
  		
  		var userToken =  uni.getStorageSync('userToken')
  		if(userToken && resetSocketNumber<12){
  			timer = setTimeout(()=>{
  				console.log('socket关闭了重连');
  				dispatch('initNimSDK')
  			},1000*5)
  		}else{
  			console.log('重登吧');
  		}
  		
  
  	})
  
  	// console.log(state.nim);
  
  	state.nim.onMessage(res => {
  		// console.log('收到解密消息', app.$api.util.decrypt(res.data));
  
  		let receiveMsg = JSON.parse(app.$api.util.decrypt(res.data))
  		console.log(receiveMsg);
  		if (receiveMsg.code == 200) {
  			if (receiveMsg.dataType == 203) {
  				// 更新会话列表
  				console.log('收到会话列表消息', receiveMsg);
  				store.commit('updateUserInfo', receiveMsg.data)
  			} else if (receiveMsg.dataType == 205) {
  				// 会话聊天记录
  				console.log('收到会话聊天记录', receiveMsg);
  				if (receiveMsg.data.length === 0) {
  					commit('setNoMoreHistoryMsgs')
  				} else {
  					let msgs = receiveMsg.data.map(msg => {
  						return msg
  					})
  					// msgs = msgs.reverse()
  					commit('updateCurrSessionMsgs', {
  						type: 'concat',
  						msgs: msgs
  					})
  				}
  			}else if (receiveMsg.dataType == 212) {
  				// 会话聊天记录
  				console.log('单独收到会话置顶通知栏消息', receiveMsg);
  				
  				commit('updateCurrSessionTopNoticeMsg', {
  					type: 'put',
  					msg: receiveMsg.data.message
  				})
  			} else if (receiveMsg.dataType == 206) {
  				console.log('收到删除会话消息', receiveMsg);
  
  				var friends = state.IMfriends.filter(r => {
  					return r.sessionId !== receiveMsg.data.sessionId;
  				});
  				store.commit('updateUserInfo', friends);
  			} else {
  				console.log('收到普通消息', receiveMsg);
  				onMsg(receiveMsg)
  			}
  		} else {
  			console.log('收到心跳消息', receiveMsg);
  
  			// netty
  			const nettyToken = uni.getStorageSync('nettyToken');
  			var message = {
  				code: 202,
  				// token: nettyToken,
  			}
  			console.log('发送心跳信息', message);
  			var encrypted = app.$api.util.encryptPwd(message)
  			store.state.nim.send({
  				data: encrypted.toString(),
  				complete: (complete) => {
  					console.log('发完心跳', complete);
  					// store.commit('updateUserInfo', users)
  				}
  			})
  		}
  
  		// var responKey = cryptoJs.enc.Utf8.parse("GREEDISGOOD-2200");
  		// var decrypt = cryptoJs.AES.decrypt(res.data, responKey, {
  		// 	mode: cryptoJs.mode.ECB,
  		// 	padding: cryptoJs.pad.Pkcs7
  		// });
  		// console.log(cryptoJs.enc.Utf8.stringify(decrypt));
  	})
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  	// IM 云信
  
  	// if (state.nim) {
  	// 	state.nim.disconnect()
  	// }
  	// // dispatch('showLoading')
  	// // 初始化SDK
  	// state.nim = NIM.getInstance({
  	// 	debug: false,
  	// 	appKey: '6fe51d190542e42b9d8f931dc75c580a',
  	// 	account: loginInfo.imAccountDTO.imAccountUUID,
  	// 	token: loginInfo.imAccountDTO.imToken,
  	// 	// transports: ['websocket'],
  	// 	db: true,
  	// 	needReconnect:true,
  	// 	// logFunc: new SDK.NIM.LoggerPlugin({
  	// 	//   url: '/webdemo/h5/getlogger',
  	// 	//   level: 'info'
  	// 	// }),
  	// 	// syncSessionUnread: true, //是否同步会话的未读数
  	// 	// syncRobots: true,
  	// 	// autoMarkRead: true, // 默认为true
  	// 	onconnect: function onConnect(event) {
  	// 		console.log('连接成功');
  	// 		// if (loginInfo) {
  	// 		//   // 连接上以后更新uid
  	// 		//   commit('updateUserUID', loginInfo)
  	// 		// }
  
  	// 		getIMSessions()
  	// 	},
  	// 	onerror: function onError(event) {
  	// 		// alert(JSON.stringify(event))
  	// 		// alert('网络连接状态异常')
  	// 	},
  	// 	onwillreconnect: function onWillReconnect() {
  	// 		console.log('即将重连');
  	// 		console.log(event)
  	// 	},
  	// 	ondisconnect: function onDisconnect(error) {
  	// 		 console.log('丢失连接');
  	// 		 // if(state.nim){
  	// 			//  state.nim.connect();
  	// 		 // }
  
  	// 		// switch (error.code) {
  	// 		// 	// 账号或者密码错误, 请跳转到登录页面并提示错误
  	// 		// 	case 302:
  	// 		// 		// pageUtil.turnPage('帐号或密码错误', 'login')
  	// 		// 		break
  	// 		// 		// 被踢, 请提示错误后跳转到登录页面
  	// 		// 	case 'kicked':
  	// 		// 		let map = {
  	// 		// 			PC: '电脑版',
  	// 		// 			Web: '网页版',
  	// 		// 			Android: '手机版',
  	// 		// 			iOS: '手机版',
  	// 		// 			WindowsPhone: '手机版'
  	// 		// 		}
  	// 		// 		let str = error.from
  	// 		// 		 let errorMsg =
  	// 		// 		 	`你的帐号于${util.formatDate(new Date())}被${(map[str] || '其他端')}踢出下线，请确定帐号信息安全!`
  	// 		// 		 pageUtil.turnPage(errorMsg, 'login')
  	// 		// 		break
  	// 		// 	default:
  	// 		// 		break
  	// 		// }
  	// 	},
  	// 	// // 多端登录
  	// 	// onloginportschange: onLoginPortsChange,
  	// 	// 用户关系及好友关系 黑名单
  	// 	// onblacklist: onBlacklist,
  	// 	// onsyncmarkinblacklist: onMarkInBlacklist,
  	// 	// onmutelist: onMutelist,
  	// 	// onsyncmarkinmutelist: onMarkInMutelist,
  	// 	onfriends: onFriends,
  	// 	// onsyncfriendaction: onSyncFriendAction,
  	// 	// // 机器人
  	// 	// onrobots: onRobots,
  	// 	// // 用户名片 - actions/userInfo
  	// 	// onmyinfo: onMyInfo,
  	// 	// onupdatemyinfo: onMyInfo,
  	// 	// onusers: onUserInfo, // 获取好友名片
  	// 	// onupdateuser: onUserInfo,
  	// 	// // // 群组
  	// 	// onteams: onTeams,
  	// 	// onsynccreateteam: onSynCreateTeam,
  	// 	// syncTeams: true,
  	// 	// onteammembers: onTeamMembers,
  	// 	// onCreateTeam: onCreateTeam,
  	// 	// onDismissTeam: onDismissTeam,
  	// 	// onUpdateTeam: onUpdateTeam,
  	// 	// onAddTeamMembers: onAddTeamMembers,
  	// 	// onRemoveTeamMembers: onRemoveTeamMembers,
  	// 	// onUpdateTeamManagers: onUpdateTeamManagers,
  	// 	// onupdateteammember: onUpdateTeamMember,
  	// 	// onUpdateTeamMembersMute: onUpdateTeamMembersMute,
  	// 	// onTeamMsgReceipt: onTeamMsgReceipt,
  	// 	// // // 会话
  	// 	onsessions: onSessions,
  	// 	onupdatesession: onUpdateSession,
  	// 	// // // 消息
  	// 	onroamingmsgs: onRoamingMsgs,
  	// 	onofflinemsgs: onOfflineMsgs,
  	// 	onmsg: onMsg,
  	// 	// // // 系统通知
  	// 	// onsysmsg: onSysMsg,
  	// 	// onofflinesysmsgs: onSysMsgs,
  	// 	// onupdatesysmsg: onSysMsg, // 通过、拒绝好友申请会收到此回调
  
  	// 	// onsysmsgunread: onSysMsgUnread,
  	// 	// onupdatesysmsgunread: onSysMsgUnread,
  
  	// 	// onofflinecustomsysmsgs: onCustomSysMsgs,
  	// 	// oncustomsysmsg: onCustomSysMsgs,
  	// 	// // // 同步完成
  	// 	// onsyncdone: function onSyncDone () {
  	// 	//   dispatch('hideLoading')
  	// 	//   // 说明在聊天列表页
  	// 	//   if (store.state.currSessionId) {
  	// 	//     dispatch('setCurrSession', store.state.currSessionId)
  	// 	//   }
  	// 	// }
  	// })
  }
  
  
  ```

- msg.js

  ```js
  import store from './'
  // import config from '../../configs'
  import {
  	getIMSessions
  } from './initNimSDK.js'
  import app from '@/main.js'
  
  export function getUser({
  	state,
  	commit
  }, account) {
  	const nim = state.nim
  	nim.getUser({
  		account: store.state.userInfo.userUuid,
  		done: (error, user) => {
  			console.log(user);
  		}
  	})
  }
  
  
  // 获取好友列表资料
  export function getUsers({
  	state,
  	commit
  }, accounts) {
  	const nim = state.nim
  	nim.getUsers({
  		accounts,
  		done: (error, user) => {
  			console.log(user);
  			// 更新好友列表
  			// store.commit('updateFriends', [], friends)
  			// 更新好友资料
  			store.commit('updateUserInfo', user)
  		}
  	})
  }
  
  
  
  // 删除好友
  export function deleteFriend({
  	state,
  	commit
  }, account) {
  	const nim = state.nim
  	nim.deleteFriend({
  		account,
  		done: (error, friends) => {
  			console.log(friends);
  			if (error) {
  				alert(error)
  				return
  			}
  			if (!Array.isArray(friends)) {
  				friends = [friends]
  			}
  			friends = friends.map(item => {
  				item.isFriend = false
  				return item
  			})
  			// 更新好友列表
  			// store.commit('updateFriends', [], friends)
  			// 更新好友资料
  			// store.commit('updateUserInfo', friends)
  		}
  	})
  }
  
  export function addFriend({
  	state,
  	commit
  }, account) {
  	// console.log(state);
  	const nim = state.nim
  	nim.addFriend({
  		account,
  		ps: '',
  		done: (error, friends) => {
  			console.log(friends);
  			if (error) {
  				alert(error)
  				return
  			}
  			if (!Array.isArray(friends)) {
  				friends = [friends]
  			}
  
  			friends = friends.map(item => {
  				if (typeof item.isFriend !== 'boolean') {
  					item.isFriend = true
  				}
  				return item
  			})
  
  
  			// // 补充好友资料
  			// store.dispatch('searchUsers', {
  			// 	accounts: friends.map(item => {
  			// 		return item.account
  			// 	}),
  			// 	done: (users) => {
  			// 		const nim = store.state.nim
  			// 		friends = nim.mergeFriends(friends, users).map(formatUserInfo)
  			// 		// 更新好友列表
  			// store.commit('updateFriends', friends)
  			// 		// 更新好友资料
  			// 		store.commit('updateUserInfo', friends)
  			// 	}
  			// })
  		}
  	})
  }
  
  
  
  
  
  // 消息
  export function formatMsg(msg) {
  	const nim = store.state.nim
  	// 网易im
  	// if (msg.type === 'robot') {
  	// 	if (msg.content && msg.content.flag === 'bot') {
  	// 		if (msg.content.message) {
  	// 			msg.content.message = msg.content.message.map(item => {
  	// 				switch (item.type) {
  	// 					case 'template':
  	// 						item.content = nim.parseRobotTemplate(item.content)
  	// 						break
  	// 					case 'text':
  	// 					case 'image':
  	// 					case 'answer':
  	// 						break
  	// 				}
  	// 				return item
  	// 			})
  	// 		}
  	// 	}
  	// }
  	return msg
  }
  
  export function onRoamingMsgs(obj) {
  	let msgs = obj.msgs.map(msg => {
  		return formatMsg(msg)
  	})
  	store.commit('updateMsgs', msgs)
  }
  
  export function onOfflineMsgs(obj) {
  	let msgs = obj.msgs.map(msg => {
  		return formatMsg(msg)
  	})
  	store.commit('updateMsgs', msgs)
  }
  
  export function onMsg(msg) {
  	console.log('收到消息', msg.scene, msg.type, msg);
  	var uniqueNettyToken = msg.data.token
  	msg = formatMsg(msg.data.message)
  	console.log(msg);
  	if(!msg) return
  
  
  	// if(msg.from){
  		let frieds = store.state.IMfriends;
  		let idx = frieds.findIndex(item => item.to === msg.from)
  	// }
  	// 网易im
  	// var obj = frieds.find(item => item.to === msg.from);
  	// netty
  	var obj = frieds.find(item => item.sessionId === msg.sessionId);
  
  	// console.log(idx);
  	// addFriend(msg.from)
  	// store.commit('putMsg', msg)
  	if (msg.sessionId === store.state.currSessionId) {
  		store.commit('updateCurrSessionMsgs', {
  			type: 'put',
  			msg
  		})
  		// 发送已读回执
  		// store.dispatch('sendMsgReceipt')
  		if (msg.flow == 'out') {
  			// 网易im
  			// var listItem = frieds.find(item => item.to === msg.to);
  
  			// netty
  			var listItem = frieds.find(item => item.sessionId === msg.sessionId);
  			if (listItem) {
  				listItem.lastMessageDateTime = msg.time;
  				if (msg.type === 'text') {
  					// 网易im
  					listItem.lastMessage = msg.text;
  					// netty
  					listItem.lastMessageDT0.text = msg.text;
  				} else if (msg.type === 'image') {
  					// 网易im
  					listItem.lastMessage = '[图片]';
  					// netty
  					listItem.lastMessageDT0.text = '[图片]';
  				} else if (msg.type === 'custom') {
  					// let content = JSON.parse(msg.content);
  					// let customType = content.customType;
  					// listItem.lastMessage = '[其他]';
  					let content = JSON.parse(msg.content);
  					let customType = parseInt(content.customType)
  					var typeName = ''
  					switch (customType) {
  						case 1:
  							typeName = '[商品信息]'
  							break;
  						case 2:
  							typeName = '[订单信息]'
  							break;
  						case 3:
  							typeName = '[商品信息]'
  							break;
  						case 4:
  							typeName = '[订单信息]'
  							break;
  						case 5:
  							typeName = '[订单信息]'
  							break;
  						case 6:
  							typeName = '[订单信息]'
  							break;
  						case 7:
  							typeName = '[优惠券信息]'
  							break;
  						default:
  							typeName = '[其他]'
  							break;
  					}
  					// 网易im
  					listItem.lastMessage = typeName;
  
  					// netty
  					listItem.lastMessageDT0.text = typeName;
  				}
  				store.commit('updateUserInfo', frieds);
  			} else {
  				getIMSessions();
  			}
  		} else {
  			if (obj) {
  				obj.lastMessageDateTime = msg.time;
  				if (msg.type === 'text') {
  					obj.lastMessage = msg.text;
  					// netty
  					obj.lastMessageDT0.text = msg.text;
  				} else if (msg.type === 'image') {
  					obj.lastMessage = '[图片]';
  					// netty
  					obj.lastMessageDT0.text = '[图片]';
  				} else if (msg.type === 'custom') {
  					// let content = JSON.parse(msg.content);
  					// console.log(msg.content);
  					// let customType = content.customType;
  					// obj.lastMessage = '[其他]';
  					let content = JSON.parse(msg.content);
  					let customType = content.customType;
  					var typeName = ''
  					switch (customType) {
  						case 1:
  							typeName = '[商品信息]'
  							break;
  						case 2:
  							typeName = '[订单信息]'
  							break;
  						case 3:
  							typeName = '[商品信息]'
  							break;
  						case 4:
  							typeName = '[订单信息]'
  							break;
  						case 5:
  							typeName = '[订单信息]'
  							break;
  						case 6:
  							typeName = '[订单信息]'
  							break;
  						case 7:
  							typeName = '[优惠券信息]'
  							break;
  						default:
  							typeName = '[其他]'
  							break;
  					}
  					obj.lastMessage = typeName;
  					// netty
  					obj.lastMessageDT0.text = typeName;
  				}
  				store.commit('updateUserInfo', frieds);
  			}
  
  			// store.dispatch('sendMsgReceipt', msg.from);
  			store.dispatch('sendMsgReceipt', {
  				sessionId:msg.sessionId,
  				token:uniqueNettyToken
  			});
  		}
  
  	} else {
  		// 收到消息 不是现在这个会话框的人
  		if (obj) {
  			obj.unread++;
  			obj.lastMessageDateTime = msg.time;
  			if (msg.type === 'text') {
  				obj.lastMessage = msg.text;
  				// netty
  				obj.lastMessageDT0.text = msg.text;
  			} else if (msg.type === 'image') {
  				obj.lastMessage = '[图片]';
  				// netty
  				obj.lastMessageDT0.text = '[图片]';
  			} else if (msg.type === 'custom') {
  				// console.log(msg.content);
  				// let content = JSON.parse(msg.content);
  				// let customType = content.customType;
  				// switch (customType){
  				// 	case value:
  				// 		break;	
  				// 	default:
  				// 		break;
  				// }
  				obj.lastMessage = '[其他]';
  				// netty
  				obj.lastMessageDT0.text = '[其他]';
  			}
  
  			// 收到消息 置顶 列表置顶未读
  			frieds.map((item, index) => {
  				frieds.unshift(frieds.splice(idx, 1)[0]);
  			})
  
  
  			store.commit('updateUserInfo', frieds);
  		} else {
  			setTimeout(() => {
  				getIMSessions();
  			}, 200)
  		}
  
  		if (msg.flow !== 'out') {
  			uni.showTabBarRedDot({
  				index: 3,
  				success: (e) => {
  					// console.log(e);
  				}
  			})
  		}
  	}
  	// if (msg.scene === 'team' && msg.type === 'notification') {
  	// 	store.dispatch('onTeamNotificationMsg', msg)
  	// }
  
  
  }
  
  function onSendMsgDone(error, msg) {
  	// store.dispatch('hideLoading')
  	// console.log(error);
  	// console.log(msg);
  	console.log('发送' + msg.scene + ' ' + msg.type + '消息' + (!error ? '成功' : '失败') + ', id=' +
  		msg.idClient);
  	if (error) {
  		// 被拉黑
  		// if (error.code === 7101) {
  		// 	msg.status = 'success'
  		// 	this.$u.toast(error.message)
  		// } else {
  		// 	this.$u.toast(error.message)
  		// }
  		console.log(error);
  	} else {
  
  		// let frieds = store.state.IMfriends;
  		// var obj = frieds.find(item => item.to === msg.to);
  		// if (obj) {
  		// 	obj.lastMessageDateTime = msg.time;
  		// 	if (msg.type === 'text') {
  		// 		obj.lastMessage = msg.text;
  		// 	} else if (msg.type === 'image') {
  		// 		obj.lastMessage = '[图片]';
  		// 	} else if (msg.type === 'custom') {
  		// 		// let content = JSON.parse(obj.content);
  		// 		// let customType = content.customType;
  		// 		obj.lastMessage = '[其他]';
  		// 	}
  		// 	store.commit('updateUserInfo', frieds);
  		// }
  
  		let postData = {
  			ope: msg.scene,
  			to: msg.to,
  			type: msg.type,
  		}
  		if (msg.type == 'custom') {
  			postData.body = msg.content
  		} else if (msg.type == 'text') {
  			postData.text = msg.text
  		} else if (msg.type == 'image') {
  			postData.body = JSON.stringify(msg.file)
  		}
  
  		app.$api.request.saveMessage(postData, res => {
  			if (res.body.status.statusCode === '0') {
  				// 轮播图
  				console.log(1);
  				onMsg(msg)
  			} else {
  				console.log(res.body.status.errorDesc);
  			}
  		}, true);
  	}
  
  	// uni.pageScrollTo({
  	// 	scrollTop: 3000000,
  	// 	duration: 0
  	// });
  }
  
  
  
  // 消息撤回
  export function onRevocateMsg(error, msg) {
  	const nim = store.state.nim
  	if (error) {
  		if (error.code === 508) {
  			alert('发送时间超过2分钟的消息，不能被撤回')
  		} else {
  			alert(error)
  		}
  		return
  	}
  	let tip = ''
  	if (msg.from === store.state.userUID) {
  		tip = '你撤回了一条消息'
  	} else {
  		let userInfo = store.state.userInfos[msg.from]
  		if (userInfo) {
  			tip = `${util.getFriendAlias(userInfo)}撤回了一条消息`
  		} else {
  			tip = '对方撤回了一条消息'
  		}
  	}
  	nim.sendTipMsg({
  		isLocal: true,
  		scene: msg.scene,
  		to: msg.to,
  		tip,
  		time: msg.time,
  		done: function sendTipMsgDone(error, tipMsg) {
  			let idClient = msg.deletedIdClient || msg.idClient
  			store.commit('replaceMsg', {
  				sessionId: msg.sessionId,
  				idClient,
  				msg: tipMsg
  			})
  			if (msg.sessionId === store.state.currSessionId) {
  				store.commit('updateCurrSessionMsgs', {
  					type: 'replace',
  					idClient,
  					msg: tipMsg
  				})
  			}
  		}
  	})
  }
  
  
  export function revocateMsg({
  	state,
  	commit
  }, msg) {
  	const nim = state.nim
  	let {
  		idClient
  	} = msg
  	msg = Object.assign(msg, state.msgsMap[idClient])
  	nim.deleteMsg({
  		msg,
  		done: function deleteMsgDone(error) {
  			onRevocateMsg(error, msg)
  		}
  	})
  }
  export function updateLocalMsg({
  	state,
  	commit
  }, msg) {
  	store.commit('updateCurrSessionMsgs', {
  		type: 'replace',
  		idClient: msg.idClient,
  		msg: msg
  	})
  	state.nim.updateLocalMsg({
  		idClient: msg.idClient,
  		localCustom: msg.localCustom
  	})
  	store.commit('replaceMsg', {
  		sessionId: msg.sessionId,
  		idClient: msg.idClient,
  		msg: msg
  	})
  }
  
  // 发送普通消息
  export function sendMsg({
  	state,
  	commit
  }, obj) {
  	// console.log(state);
  
  	console.log(obj);
  	const nim = state.nim
  	obj = obj || {}
  	let type = obj.type || ''
  	let customInfo = obj.custom || {}
  
  
  	// netty
  	const nettyToken = uni.getStorageSync('nettyToken');
  	// obj.token = nettyToken.split(',')[0] // 针织E家
  	// obj.token = nettyToken.split(',')[1] // 商城
  	
  	switch (type) {
  		case 'text':
  			// obj.type = 0
  			console.log('未加密前的文本信息', obj)
  			var encrypted = app.$api.util.encryptPwd(obj)
  			console.log('加密后', encrypted)
  			store.state.nim.send({
  				data: encrypted.toString(),
  				complete: (complete) => {
  					console.log('发送普通文本消息', complete);
  					// store.commit('updateUserInfo', users)
  					// onSendMsgDone()
  				}
  			})
  			break
  		case 'image':
  			// obj.type = 0
  			console.log('未加密前的文本信息', obj)
  			var encrypted = app.$api.util.encryptPwd(obj)
  			console.log('加密后', encrypted)
  			store.state.nim.send({
  				data: encrypted.toString(),
  				complete: (complete) => {
  					console.log('发送普通文本消息', complete);
  					// store.commit('updateUserInfo', users)
  					// onSendMsgDone()
  				}
  			})
  			break
  		case 'custom':
  			nim.sendCustomMsg({
  				scene: obj.scene,
  				to: obj.to,
  				pushContent: obj.pushContent,
  				custom: JSON.stringify(customInfo),
  				content: JSON.stringify(obj.content),
  				done: onSendMsgDone
  			})
  		case 'history':
  
  			var encrypted = app.$api.util.encryptPwd(obj)
  
  			store.state.nim.send({
  				data: encrypted.toString(),
  				complete: (complete) => {
  					console.log('发消息获取历史记录', complete);
  					// store.commit('updateUserInfo', users)
  				}
  			})
  			break
  		case 'topNoticeMsg':
  		
  			var encrypted = app.$api.util.encryptPwd(obj)
  		
  			store.state.nim.send({
  				data: encrypted.toString(),
  				complete: (complete) => {
  					console.log('发消息获取置顶通知栏消息', complete);
  					// store.commit('updateUserInfo', users)
  				}
  			})
  			break
  	}
  
  
  	// im
  	// switch (type) {
  	// 	case 'text':
  	// 		nim.sendText({
  	// 			scene: obj.scene,
  	// 			to: obj.to,
  	// 			text: obj.text,
  	// 			custom: JSON.stringify(customInfo),
  	// 			done: onSendMsgDone,
  	// 			needMsgReceipt: obj.needMsgReceipt || false
  	// 		})
  	// 		break
  	// 	case 'custom':
  	// 		nim.sendCustomMsg({
  	// 			scene: obj.scene,
  	// 			to: obj.to,
  	// 			pushContent: obj.pushContent,
  	// 			custom: JSON.stringify(customInfo),
  	// 			content: JSON.stringify(obj.content),
  	// 			done: onSendMsgDone
  	// 		})
  	// }
  }
  
  // 发送文件消息
  export function sendFileMsg({
  	state,
  	commit
  }, obj) {
  	const nim = state.nim
  	// let { type, fileInput } = obj
  	// if (!type && fileInput) {
  	//   type = 'file'
  	//   if (/\.(png|jpg|bmp|jpeg|gif)$/i.test(fileInput.value)) {
  	//     type = 'image'
  	//   } else if (/\.(mov|mp4|ogg|webm)$/i.test(fileInput.value)) {
  	//     type = 'video'
  	//   }
  	// }
  	// store.dispatch('showLoading')
  	const data = Object.assign({
  		// type,
  		uploadprogress: function(data) {
  			// console.log(data.percentageText)
  		},
  		uploaderror: function() {
  			// fileInput.value = ''
  			console && console.log('上传失败')
  		},
  		uploaddone: function(error, file) {
  			// fileInput.value = ''
  			// console.log(error);
  			// console.log(file);
  		},
  		beforesend: function(msg) {
  			// console && console.log('正在发送消息, id=', msg);
  		},
  		done: function(error, msg) {
  			onSendMsgDone(error, msg)
  		}
  	}, obj)
  	nim.sendFile(data)
  }
  
  // 发送机器人消息
  export function sendRobotMsg({
  	state,
  	commit
  }, obj) {
  	const nim = state.nim
  	let {
  		type,
  		scene,
  		to,
  		robotAccid,
  		content,
  		params,
  		target,
  		body
  	} = obj
  	scene = scene || 'p2p'
  	if (type === 'text') {
  		nim.sendRobotMsg({
  			scene,
  			to,
  			robotAccid: robotAccid || to,
  			content: {
  				type: 'text',
  				content,
  			},
  			body,
  			done: onSendMsgDone
  		})
  	} else if (type === 'welcome') {
  		nim.sendRobotMsg({
  			scene,
  			to,
  			robotAccid: robotAccid || to,
  			content: {
  				type: 'welcome',
  			},
  			body,
  			done: onSendMsgDone
  		})
  	} else if (type === 'link') {
  		nim.sendRobotMsg({
  			scene,
  			to,
  			robotAccid: robotAccid || to,
  			content: {
  				type: 'link',
  				params,
  				target
  			},
  			body,
  			done: onSendMsgDone
  		})
  	}
  }
  
  // 发送消息已读回执
  export function sendMsgReceipt({
  	state,
  	commit
  }, obj) {
  	
  	console.log(obj);
  	// // 如果有当前会话
  	// let currSessionId = store.state.currSessionId
  	// if (currSessionId) {
  	// 	// 只有点对点消息才发已读回执
  	// 	if (util.parseSession(currSessionId).scene === 'p2p') {
  	// 		let msgs = store.state.currSessionMsgs
  	// 		const nim = state.nim
  	// 		if (state.sessionMap[currSessionId]) {
  	// 			nim.sendMsgReceipt({
  	// 				msg: state.sessionMap[currSessionId].lastMsg,
  	// 				done: function sendMsgReceiptDone(error, obj) {
  	// 					// do something
  	// 				}
  	// 			})
  	// 		}
  	// 	}
  	// }
  	// 	netty
  	const nettyToken = uni.getStorageSync('nettyToken');
  	// let postData = {
  	// 	sessionId: obj,
  	// 	token:nettyToken,
  	// 	code:106
  	// };
  	
  	// 新 
  	let postData = {
  		sessionId: obj.sessionId,
  		token:obj.token || nettyToken,
  		code:106
  	};
  	
  	console.log('未加密的已读回执消息', postData)
  	var encrypted = app.$api.util.encryptPwd(postData)
  	console.log('加密后', encrypted)
  	store.state.nim.send({
  		data: encrypted.toString(),
  		complete: (complete) => {
  			console.log('发送普通文本消息', complete);
  			// store.commit('updateUserInfo', users)
  			// onSendMsgDone()
  		}
  	})
  	
  	
  	// 	// 网易im
  	// let postData = {
  	// 	to: obj
  	// };
  
  	// app.$api.request.readMessage(postData, res => {
  	// 	if (res.body.status.statusCode === '0') {
  
  	// 	} else {
  	// 		console.log(res.body.status.errorDesc);
  	// 	}
  	// });
  
  	let frieds = store.state.IMfriends;
  	if (obj.sessionId) {
  		// 网易im
  		// var session = frieds.find(item => item.to === obj);
  		// netty
  		var session = frieds.find(item => item.sessionId === obj.sessionId);
  		if (session) {
  			session.unread = 0
  		}
  		
  		console.log(session);
  	} else {
  		frieds.forEach(r => {
  			r.unread = 0
  		})
  	}
  
  	store.commit('updateUserInfo', frieds);
  
  }
  
  function sendMsgReceiptDone(error, obj) {
  	console.log('发送消息已读回执' + (!error ? '成功' : '失败'), error, obj);
  }
  
  
  
  export function getTopMsgs({
  	state,
  	commit
  }, obj) {
  	const nim = state.nim
  	if (nim) {
  		// netty
  		console.log(obj);
  
  		let getTopMsgs = {
  			code: 111,
  			to: obj.to,
  			type: 'topNoticeMsg',
  			time:obj.time||'',
  			// pageNum: obj.pageNum,
  			pageSize: obj.pageSize,
  			token:obj.token
  		}
  
  		store.dispatch('sendMsg', getTopMsgs)
  	}
  }
  
  
  export function getHistoryMsgs({
  	state,
  	commit
  }, obj) {
  	const nim = state.nim
  	if (nim) {
  		// netty
  		console.log(obj);
  
  		let getHistoryMsgs = {
  			code: 103,
  			to: obj.to,
  			type: 'history',
  			time:obj.time||'',
  			// pageNum: obj.pageNum,
  			pageSize: obj.pageSize,
  			token:obj.token
  		}
  
  		store.dispatch('sendMsg', getHistoryMsgs)
  		
  
  
  
  
  		// im
  		// let {
  		// 	scene,
  		// 	to
  		// } = obj
  		// let options = {
  		// 	scene,
  		// 	to,
  		// 	// reverse: true,
  		// 	asc: true,
  		// 	limit: 20,
  		// 	done: function getHistoryMsgsDone(error, obj) {
  		// 		console.log(obj);
  		// 		// if (obj.msgs) {
  		// 		if (obj.msgs.length === 0) {
  		// 			commit('setNoMoreHistoryMsgs')
  		// 		} else {
  		// 			let msgs = obj.msgs.map(msg => {
  		// 				return formatMsg(msg)
  		// 			})
  		// 			commit('updateCurrSessionMsgs', {
  		// 				type: 'concat',
  		// 				msgs: msgs
  		// 			})
  		// 		}
  		// 		// }
  		// 		// store.dispatch('hideLoading')
  		// 	}
  		// }
  		// if (state.currSessionLastMsg) {
  		// 	options = Object.assign(options, {
  		// 		lastMsgId: state.currSessionLastMsg.idServer,
  		// 		endTime: state.currSessionLastMsg.time,
  		// 	})
  		// }
  
  		// nim.getHistoryMsgs(options)
  	}
  }
  
  export function resetNoMoreHistoryMsgs({
  	commit
  }) {
  	commit('resetNoMoreHistoryMsgs')
  }
  
  // 继续与机器人会话交互
  export function continueRobotMsg({
  	commit
  }, robotAccid) {
  	commit('continueRobotMsg', robotAccid)
  }
  
  
  
  
  
  
  
  
  
  
  // session  聊天室
  // 设置聊天室ID
  export function setCurrSession({
  	state,
  	commit,
  	dispatch
  }, sessionId) {
  	const nim = state.nim
  	if (sessionId) {
  		console.log(sessionId);
  		commit('updateCurrSessionId', {
  			type: 'init',
  			sessionId
  		})
  		// if (nim) {
  		//   // 如果在聊天页面刷新，此时还没有nim实例，需要在onSessions里同步
  		//   nim.setCurrSession(sessionId)
  		//   commit('updateCurrSessionMsgs', {
  		//     type: 'init',
  		//     sessionId
  		//   })
  		//   // 发送已读回执
  		// dispatch('sendMsgReceipt')
  		// }
  	}
  }
  
  // 重置聊天室ID
  export function resetCurrSession({
  	state,
  	commit
  }) {
  	const nim = state.nim
  	commit('updateCurrSessionMsgs', {
  		type: 'destroy'
  	})
  	// nim.resetCurrSession()
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

    





