# uni-app                     追崇技术,乐在分享！

## 尺寸单位

1. rpx 自适应多种分辨率布局 
2. 750px下  1rpx = 1px
3. 计算公式：`750*元素宽度/设计稿宽度 = uni-app 宽度rpx`

## 修改uni-app内部样式

1.  class  uni-app 原生样式
2.  class /deep/  uni-app 原生样式

## uniapp 自定义loading  showTotast 样式

1. ```css
   // 找到元素类名，在APP.vue中 重新书写
   uni-modal .uni-modal {
       border-radius: 24rpx;
   }
   
   uni-toast .uni-icon_toast.uni-loading {
       height: 80rpx;
       width: 80rpx;
       background: url(static/img/loading.gif);
       background-size: 100% 100%;
       animation: none;
   }
   ```
   
2. 在App.vue 中的  style  中书写样式，类名可以在element中查找到

   ```css
   ::-webkit-scrollbar {
       display: none;
   }
   
   uni-modal .uni-modal {
       border-radius: 24rpx;
   }
   
   uni-toast .uni-icon_toast.uni-loading {
       height: 80rpx;
       width: 80rpx;
       background: url(static/img/loading.gif);
       background-size: 100% 100%;
       animation: none;
   }
   ```

   

## 吸顶 CSS3

1. ```css
   position: -webkit-sticky;
   position: sticky;
   top: 0;
   z-index: 9;
   ```




## CSS 引用字体跨域问题（解决）

### 1.方案: 将文字设置为 base64 编码

- 字体转base64编码网址：https://transfonter.org/

- 记得打开 base64 encode ， 并且勾选 TTF 选项（一般字体文件是 .ttf  格式）

- 下载转换好的文件，解压 并找到 **stylesheet.css** 文件

- 打开 **stylesheet.css** 文件，复制其中的内容 

- 粘贴到需要引用的css文件中

  ```css
  @font-face{
      font-family: 'iconfont';
      src: url('data:application/font-woff2;charset=utf-8;base64,......') format('woff2'),
          url('data:application/font-woff;charset=utf-8;base64,.....') format('woff'),
          url('iconfont.ttf') format('truetype');
      /* url('iconfont.ttf') 是iconfont.ttf的文件路径 （相对路径或绝对路径） */
      font-weight: normal;
      font-style: normal;
      font-display: swap;
  }
  ```

  





## 适配特殊端 ifdef 

1. ```html
   <!-- 仅在H5端生效 -->
   <!-- #ifdef H5 -->
   <view class="history_head_nav">
       <view class="back_page" @click="back_page">
           <image src="/static/img/mine/fanhui.png" mode=""></image>
       </view>
       <view class="history_head_title">
           浏览记录
       </view>
       <view class="clear_history" @click="clear_history">
           清空
       </view>
   </view>
   <!-- #endif -->
   ```



## 沉浸式导航条

### 1. view-UI 封装的 沉浸式导航条

1. 先 导入 v-view

2. ```json
   "pages": [
       {
           "path": "pages/index/index",
           "style" : {
               "navigationStyle": "custom",
               "navigationBarTextStyle": "white"
           }
       }
   ]
   ```

   

3. ```html
   <u-navbar :is-back="true" title="个性定制" :background="background" title-color="#FFFFFF" :border-bottom="false" :custom-back="hhhhh">
       <view class="slot-wrap" slot="right">
           <view class="release" @click="cl">
               发布
           </view>
       </view>
   </u-navbar>
   ```

4. js

   ```js
   data() {
       return {
           // 自定义顶部导航-渐变色
           background: {
               // backgroundImage: 'linear-gradient(90deg, rgb(255, 0, 69), rgb(255, 106, 95))',
               backgroundImage: 'linear-gradient(0deg, #3AA4F8, #2C9BF9); '
           },
           offsetTop: 0,// 如果需要吸顶的情况下使用
       }
   },
       methods: {
           hhhhh(){
               console.log(2);
           },
   
               cl(){
                   console.log(1);
               },
   
   
                   // 适配吸顶tab高度
                   setOffestTop() {
                       let systemInfo = uni.getSystemInfoSync()
                       let topPx = systemInfo.statusBarHeight + 44 // 顶部状态栏+沉浸式自定义顶部导航
                       this.offsetTop = topPx / (uni.upx2px(topPx) / topPx) // px转rpx
                   },
   
       },
           onShow() {
               this.setOffestTop()
           },
   ```

5. 详见

   ```
   https://zhuanlan.zhihu.com/p/269773201
   ```


### 2. 自定义封装 沉浸式导航栏组件

1. ```vue
   <template>
   <view class="status_bar" :style="'height:'+status_bar_height+'px'"></view>
   </template>
   
   <script>
       export default {
           data() {
               return {
                   status_bar_height:25,
               };
           },
   
           mounted(){
               // console.log(uni.getSystemInfoSync().statusBarHeight);
               this.status_bar_height = uni.getSystemInfoSync().statusBarHeight
               
               // statusBarHeight  H5端为 0   微信小程序则会获取各个设备的状态栏高度
           }
       }
   </script>
   
   <style>
       .status_bar {
           height: var(--status-bar-height);
           width: 100%;
       }
   </style>
   ```

2. 使用

   ```vue
   // js 引入
   import statusBar from "../../components/status-bar/status-bar.vue"
   
   // template 模板使用  放置最上方
   <status-bar></status-bar>
   ```

   



## 多余文字用省略号代替

1. ```css
   display: -webkit-box;
   /* 设置为两行 第二行多余用省略号代替 */
   -webkit-line-clamp: 2;
   -webkit-box-orient: vertical;
   overflow: hidden;
   ```

2. ```css
   display: block;
   overflow: hidden;
   white-space: nowrap;
   text-overflow:ellipsis;
   ```
   
3. ```css
   display: -webkit-box;
   word-break: break-all;
   -webkit-box-orient: vertical;
   -webkit-line-clamp:2;   //行数为2
   overflow: hidden;
   text-overflow:ellipsis;
   ```

   



## 多个条件多个类样式的切换

1. ```html
   :class="{'showEdit':navIndex===0, 'add_second':navIndex===1}"
   ```




## style 样式判断

1. ```html
    <i:style="{'color':isBling?'red':'white'}" @click=""></i>
   ```

   



## 联系搜索

1. ```html
   <!-- @input 用户输入的时候执行搜索函数 边输入边执行搜索函数 -->
   <input type="text" v-model="search_val" placeholder="请按关键字搜索" @input="ipt_search()" />
   ```

   



## 滚动标签

1. ```html
   <view class="scroll_tab" style="width: 650rpx;">
   		<scroll-view scroll-x="true" style="width: 100%;">
   				<view @click="change(item.id)" class="scroll_tab_item"  :class="currentID === item.id? 'active':''" v-for="(item,index) in rec_list" :key="item.id">
   						<text>{{item.name}}</text>
   				</view>
   			</scroll-view>
   </view>
   
   // 样式
   .scroll_tab {
   	white-space: nowrap;
   	padding: 0 32rpx;
   	height: 100rpx;
   	display: flex;
   	align-items: center;
   }
   
   .scroll_tab /deep/ .uni-scroll-view-content {
   	display: flex;
   	box-sizing: border-box;
   	/* padding: 32rpx 68rpx; */
   	display: flex;
   	align-items: center;
   }
   
   .scroll_tab .scroll_tab_item {
   	display:inline-block;
   	/* 兼容微信小程序 否则flex 布局无效 */
   	margin-right: 44rpx;
   	font-size: 26rpx;
   	text-align: center;
   	color: #1A1A1A;
   	box-sizing: border-box;
   	white-space: nowrap;
   	/* 防止文字换行 */
   }
   
   .scroll_tab .active {
	color: #4278FF;
   	font-weight: bold;
   }
   ```
   



## 滚动条的制作（兼容微信小程序）

1. html文件

   ```html
   <!-- 自制滚动条 -->
   <view class="srcoll_line">
       <view class="line_56_E3">
           <view class="line_40_3D" :style="{left:left+'rpx'}"></view>
       </view>
   </view>
   ```

2. js 文件

   ```js
   // 滚动条的移动
   scroll_move(e) {
       // console.log(e.target.scrollLeft);
       uni.getSystemInfo({
           success: (res) => {
               this.winWidth = res.screenWidth
           }
       })
       this.left = Math.ceil(e.target.scrollLeft / 10 / (this.winWidth / 750))
   },
   ```

   



## swiper踩坑

1. 横向滚动，必须给定 `width`  `white-space:nowrap` 的值

   - ```css
     // 防止文字自动换行
     white-space: nowrap;
     ```

2. `swiper-item` 必须给定 `width`， `height`，`display:inline-block`

   - ```css
     // 设置该属性 兼容微信小程序  否则微信小程序没有横向效果 
     display:inline-block;
     ```
   
3. `swiper-item`  



## inline-block 元素掉落 不在一水平线上

1. ```css
   /* 可以将inline-block的元素 放在 顶部 */
   vertical-align:top;
   /* 垂直对齐方式 */
   ```



## 解决 滚动组件srcoll-view内部使用了flex布局后文字会自动换行

1. ```css
   // 解决办法 对文字设置  强制文字不进行换行
   white-space: nowrap;
   ```



## 使用 flex后，子元素设置超过一行省略不生效

1. ```html
   <div class="main">
       <img alt="" class="logo" src="pic.jpg">
       <div class="content">
           <h4 class="name">a name</h4>
           <p class="info">a info</p>
           <p class="notice">Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum.</p>
       </div>
   </div>
   ```

2. ```scss
   .main {
       display: flex;
   }
   .logo {
       width: 100px;
       height: 100px;
       margin: 10px;
   }
   .content {
       flex: 1;
   }
   .content > * {
       white-space: nowrap;
       overflow: hidden;
       text-overflow: ellipsis;
   }
   
   /* 修改 .conent 样式 */
   .content {
       flex: 1;
       width: 0;
   }
   ```

   




## 可滚动的多列选择菜单  Picker

1. html 文件如下：

   ```html
   <!-- 分类选择 -->
   <view  :class="showCategory ? 'picker_window' : 'picker_window_none' " >
       <view class="window_top_btn">
           <view class="" @click="cancel">
               取消
           </view>
           <view class="" @click="confirm">
               确定
           </view>
       </view>
       
   	<!-- indicator-style: 选中项的样式 value：滚动选中项的索引值，是数组 @change：滚动时所触发的事件  -->
       <picker-view  :indicator-style="indicatorStyle" :value="categoryIndex" @change="bindCategoryChange">
           <picker-view-column>
               <view class="picker_view_item" v-for="(item,index) in firstCategory" :key="index">{{item.name}}</view>
           </picker-view-column>
           <picker-view-column>
               <view class="picker_view_item" v-for="(item,index) in secondCategory[categoryIndex[0]]" 										:key="index">{{item.name}}</view>
               
            	<!-- 在分类中 添加一个自定义选项 -->
               <view class="picker_view_item">
                   自定义
               </view>
           </picker-view-column>
           
       </picker-view>
   </view>
   ```
   
2. css 如下：

   ```scss
   /* 分类窗口 */
   /* 设置下拉的过度动画 */
   .picker_window {
   	position: fixed;
   	bottom: 0;
   	height: 500rpx;
   	width: 750rpx;
   	background-color: #fff;
   	transform: translateY(0rpx);
   	transition-duration: 0.3s;
   	z-index: 999;
   }
   /* 设置收起的过度 */
   .picker_window_none{
   	position: fixed;
   	bottom: 0;
   	height: 0;
   	width: 750rpx;
   	background-color: #fff;
   	transform: translateY(0rpx);
   	transition-duration: 0.3s;
   	z-index: 999;
   }
   
   /* 每一列的高度 */
   .picker_window /deep/ picker-view-column {
   	height: 360rpx;
   }
   
   
   /* 选中的 上划线 */
   .picker_window /deep/ .uni-picker-view-indicator:before {
   	border-top: 1rpx solid #EBEBEB;
   }
   
   /* 选中的 下划线 */
   .picker_window /deep/ .uni-picker-view-indicator:after {
   	border-bottom: 1rpx solid #EBEBEB;
   }
   
   /* 选中的样式 */
   .picker_window /deep/ .uni-picker-view-indicator{
   	height: 100rpx;
   }
   ```

3. js

   ```js
   // 监听 分类的滚动
   bindCategoryChange: function(e) {
       // e.detail.value 为选中项的索引的集合，是一个数组
   	this.categoryIndex = e.detail.value
   },
   
   ```

   

## 正则表达式判断

### 1.手机号码格式判断

```js
if (!(/^1(3|4|5|6|7|8|9)\d{9}$/.test(this.telValue))) {
    uni.showToast({
        title: "请输入正确的手机号码",
        icon: "none",
        mask: true,
    })
} else {
    // 格式正确
    uni.setStorage({
        key: "tel",
        data: this.telValue
    })
    // this.sendSms()
    uni.navigateTo({
        url: "/pages/login/obtian_code/obtian_code?forget=true"
    })
}
```

### 2.身份证号码格式判断

```js
if (this.nickname != null && (/(^\d{15}$)|(^\d{18}$)|(^\d{17}(\d|X|x)$)/.test(this.identity))) {

    this.info_collect = step

} else {
    uni.showToast({
        title: "请输入正确信息",
        icon: "none"
    })
}
```




## 图片文件上传 

1. ```json
   Content-Type: multipart/form-data; boundary=----WebKitFormBoundaryuhsX0TJL3QceqY5X
   ```
   
2.  实例：选择图片进行身份验证

   - ```js
     chose_img_from_photo() {
     			uni.chooseImage({
     				count: 1,
     				success: (res) => {
     					// console.log(res)
     					// 正面 人像
     					if (this.chooseId === 0) {
     						this.portrait = res.tempFilePaths[0] // 选择上传的图片中第一张
     						uni.uploadFile({
                                 // 图片上传地址
     							url: "https://api.kintting.com/api/user/uploadAuthentic",
     							header: {
     								"token": this.loginToken, // 头部添加 登录的token，视实际api情况而定，可不填写
     							},
     							filePath: this.portrait, // 被上传的图片的文件地址
     							name: "file",
     							success: (res) => {
     								let data = JSON.parse(res.data)
     								this.portrait = data.url
     							}
     						})
     
     
     
     					} else { // 反面 国徽
     						this.national = res.tempFilePaths[0]
     						uni.uploadFile({
     							url: "https://api.kintting.com/api/user/uploadAuthentic",
     							header: {
     								"token": this.loginToken
     							},
     							filePath: this.national,
     							name: "file",
     							success: (res) => {
     								let data = JSON.parse(res.data)
     								this.national = data.url
     							}
     						})
     					}
     					this.bottomTip = false
     				}
     			})
     		},
     ```
     



## 图片自适应

1.  修改`<image>` 标签中 `mode=“widthFix”`  的值， 宽度不变化，高度自适应，保持原图的宽高比

   - ```html
     <view class="second_product_item_img">
     	<image :src="$baseImgUrl + item.cover" mode="widthFix"></image>
     </view>
     
     <style>
         .second_product_item_img image {
     		height: 468rpx;
     		width: 350rpx;
     		border-radius: 4px 4rpx 0 0;
     	}
     </style>
     ```

     




## 遮罩层显示

1. ```css
   .dark {
   	width: 100vw;
   	height: 100vh;
   	top: 0;
   	left: 0;
   	position: fixed;
   	background: rgba(0, 0, 0, 0.7);
   	z-index: 100;
   	display: flex;
   	flex-direction: column;
   	justify-content: center;
   	align-items: center;
   }
   // 内部东西写在 dark 
   ```




## 遮罩层 禁止滑动 滚动 (适配 h5 微信小程序)

1. ```html
   <view class="dark" v-if="msg_tip_show"  @touchmove.stop.prevent="moveHandle" @click="msg_tip_show=false">
   ```

2. ```js
   moveHandle() {
       return
   }
   ```

   



## 获取元素属性

1. 注册 成为 ref 

2. ```javascript
   this.$refs.vref[index].$el.offsetTop
   // 可以获取到元素的位置
   ```



## 生命周期

1. 页面生命周期
   1. `onload` `onhide` 等  与小程序类似
2. 组件生命周期
   1. `created` `mounted`  等  与vue相同

## vue-devtools

1. [](https://www.jianshu.com/p/63f09651724c)





## uniapp 引入多个< script  > 标签

1. 在根目录下新建一个 html 页面

2. 写入 以下内容

   ```html
   <!DOCTYPE html>
   <html lang="zh-CN">
   	<head>
   		<meta charset="utf-8">
   		<meta http-equiv="X-UA-Compatible" content="IE=edge">
   		<meta name="viewport" content="width=device-width, user-scalable=no, initial-scale=1.0, maximum-scale=1.0, minimum-scale=1.0">
   		<title>
   			<%= htmlWebpackPlugin.options.title %>
   		</title>
   		<script>
   			document.addEventListener('DOMContentLoaded', function() {
   				document.documentElement.style.fontSize = document.documentElement.clientWidth / 20 + 'px'
   			})
   		</script>
   		<link rel="stylesheet" href="<%= htmlWebpackPlugin.options.baseUrl %>static/index.css" />
   		<!-- 引入外部js 写你想引入的 script  -->
   		<script src="http://pv.sohu.com/cityjson?ie=utf-8"></script>
           <!-- 上面这个 script 是搜狐 获取ip地址和城市名的url  -->
           <script type="text/javascript">  
               console.log(returnCitySN["cip"]+','+returnCitySN["cname"])  
           </script>
   	</head>
   	<body>
   		<noscript>
   			<strong>Please enable JavaScript to continue.</strong>
   		</noscript>
   		<div id="app"></div>
   		<!-- built files will be auto injected -->
   	</body>
   </html>
   
   ```

3. 在 manifest.json  找到 `H5配置`  ， 对index.html模板路径进行选择



## uniapp getLocation() 方法踩坑

1. 只支持  抬头 为  `https`  的
2. 在http://localhost/ 任然可以查看，但是实际使用还是需要将网站部署到https中
3. 微信小程序可以使用



## 微信小程序使用 getLocation()

1. 需要在 App.json 文件中配置 `permission`

   ```json
     /* 小程序特有相关 */
       "mp-weixin" : {
           "appid" : "wxb241004ae2cef9e6",
           "setting" : {
               "urlCheck" : true,
               "postcss" : true,
               "es6" : true,
               "minified" : true
           },
           "usingComponents" : true,
           "permission" : {
               "scope.userLocation" : {
                   "desc" : "你的位置信息将用于小程序位置接口的效果展示" // 高速公路行驶持续后台定位
               }
           }
       },
   ```

   



## map 组件的使用  腾讯地图api

### 1.templete 模板

```html
<!-- map 要给宽高， markers 是地图上的标记点（数组格式） latitude和longitude 是经纬度 -->
<map style="width: 660rpx;height: 432rpx;" :markers="markers" :latitude="point.lat" :longitude="point.lng"></map>
<map :latitude="lat" :longitude="lng" :markers="mark" @tap.stop="get_map_point"></map>
```

### 2. 微信小程序的使用

```js
export default {
    data() {
        return {
            lat: "", // 经度
            lng: "", // 维度
            point: "",
            //地图标记点
            mark: [{
                id: 0,
                latitude: "",
                longitude: "",
                iconPath: '/static/img/index/positon.png',
                width: 84,
                height: 64,
                // label: {
                // 	content: "在这里"
                // }
            }]
        }
    },
    methods: {
        // 点击地图控件
        get_map_point(e) {
            // console.log(e.detail);
            this.lat = e.detail.latitude
            this.lng = e.detail.longitude
            this.mark[0].latitude = this.lat
            this.mark[0].longitude = this.lng
        },

        // 地图获取经纬度
        getPoint() {
            // #ifdef MP-WEIXIN
			uni.request({
				url: 'https://apis.map.qq.com/ws/geocoder/v1/?address=', //仅为示例，并非真实接口地址。
				method: "GET",
				data: {
					address: this.add_name, // 地址名称 "福建省厦门市集美区"
					key: "SVOBZ-ZJEKF-BYKJ4-NGC4N-JXKVE-NOFPU"
				},
				header: {
					// 'custom-header': 'hello' //自定义请求头信息
				},
				success: res => {
					console.log(res.data.result)
					if (res.data.result.location) {
						this.point = res.data.result.location
						this.lat = res.data.result.location.lat
						this.lng = res.data.result.location.lng
						this.mark[0].latitude = this.lat
						this.mark[0].longitude = this.lng
					}
				}
			});
			// #endif
        },
    }
}
```

### 3.兼容H5 (解决 H5 调用腾讯地图api 跨域问题  使用jsonp)

```js
// npm 命令   下载  npm install vue-jsonp
// #ifdef H5
var that = this
$.ajax({
    type: "GET",
    async: false,
    url: "https://apis.map.qq.com/ws/geocoder/v1/",
    data: {
        output: "jsonp",
        address: this.add_name,
        key: "SVOBZ-ZJEKF-BYKJ4-NGC4N-JXKVE-NOFPU"
    },
    dataType: "jsonp",
    success: function(res) {
        console.log(res);
        that.point = res.result.location
        that.lat = that.point.lat
        that.lng = that.point.lng
        that.mark[0].latitude = that.lat
        that.mark[0].longitude = that.lng
    },
    error: function() {
        //  alert('fail');
    }
})
// #endif
```

#### 4. getLocation() 的调用

```json
// app.json  需要先配置
"permission" : {
    "scope.userLocation" : {
        "desc" : "你的位置信息将用于小程序位置接口的效果展示" // 高速公路行驶持续后台定位
    }
}

// 具体使用
uni.getLocation({
    type: 'wgs84 ',
    success: res => {
        this.lat = res.latitude
        this.lng = res.longitude
        this.mark[0].latitude = this.lat
        this.mark[0].longitude = this.lng
    }
})
```

   



## Vuex 之actions

1. 先在mutaticons中定义好请求方法

   - ```js
     // 更新预览通用 （企业详情，产品详情，资讯详情）
     		async setPreview(state, payload) {
     			const res = await myRequest("POST", "/api/mobile/setPreview", {
     				recordId: payload.recordId,
     				type: payload.type
     			})
     			console.log(res)
     			uni.setStorage({
     				key: JSON.stringify(payload.type),
     				data: JSON.stringify(payload.recordId)
     			})
     		},
     ```

2. 然后在Actions 中进行执行调用

   - ```js
     // 更新预览
     		SET_PREVIEW({commit}, payload) {
     			commit('setPreview', payload)
     		}
     ```

3. 实际页面中进行执行

   - ```js
     import { mapActions } from "vuex"
     
     ...mapActions(["SET_PREVIEW"]),
     
     // 然后像正常的方法一样进行调用即可
     ```




## request Promise 的封装

1. ```js
   const BASE_URL = 'https://m.kintting.cn' // 测试服网关地址
   export const BASE_IMG_URL = 'https://m.kintting.cn/' // 测试服请求图片
   export const myRequest = (method, url, data) => {
   	return new Promise((reslove, reject) => {
   		uni.showLoading({
   			title: "加载中",
   			mask: true
   		})
   
   		var token = ""
   		uni.getStorage({
   			key: "loginToken",
   			success: (storage) => {
   				uni.request({
   					url: BASE_URL + url,
   					method: method,
   					header: {
                           // 添加请求头
   						"Content-Type": "application/x-www-form-urlencoded",
   						// "token":sessionStorage.getItem("loginToken")
   						"token": storage.data
   					},
   					data: data,
   					dataType: "json",
   					success: (res) => {
   						reslove(res.data)
   						setTimeout(function() {
   							uni.hideLoading();
   						}, 100);
   					},
   					fail: (err) => {
   						reject(err)
   						setTimeout(function() {
   							uni.hideLoading();
   						}, 1000);
   					},
   					complete: () => {
   						setTimeout(function() {
   							uni.hideLoading();
   						}, 1000);
   					}
   				})
   			},
   			fail: () => {
   				uni.request({
   					url: BASE_URL + url,
   					method: method,
   					header: {
   						"Content-Type": "application/x-www-form-urlencoded",
   						// "token":sessionStorage.getItem("loginToken")
   						// "token": token
   					},
   					data: data,
   					dataType: "json",
   					success: (res) => {
   						reslove(res.data)
   						setTimeout(function() {
   							uni.hideLoading();
   						}, 0);
   					},
   					fail: (err) => {
   						reject(err)
   						setTimeout(function() {
   							uni.hideLoading();
   						}, 1000);
   					},
   					complete: () => {
   						setTimeout(function() {
   							uni.hideLoading();
   						}, 1000);
   					}
   				})
   			}
   		})
   	})
   }
   ```

   



## 双循环踩坑

1. `：key=" "`  的值不要相同

   - 实例

     ```html
     // key = i 
     <view v-for="(item,i) in AreaList" :key="i" class="province_item">
         
       <view class="province_item_letter" @click="hide">
             <text>{{item.provinceCode}}</text>
         </view>
         <view class="common">
             <!-- 省份 -->
             // key = j
             <view class="common_item" v-for="(item,j) in item.paramsObj" :key="j" 			@click.stop="select(item.provinceid,item.id,item.province,item.paramsObj)">
                 <text class="province_item_name">{{item.province}}</text>
     
                 <!-- 市 -->
                 <view class="cityList" v-if="cityshow && item.id===showCityId">
                     <scroll-view scroll-y="true" style="height: 100%;">
                         <view class="city_title">
                             <text>{{cityProvince}}</text>
                         </view>
                         <view class="underline20f8"></view>
     
                         <view class="city">
                             <view class="city_item" @click.stop="selectCity()">
                                 <view class="city_item_name">
                                     <text>全部</text>
                                 </view>
                                 <view class="underline1f1"></view>
                             </view>
                             <view @click.stop="selectCity(item.city,item.cityid)" class="city_item" v-for="(item,index) in cityList" :key="index">
                                 <view class="city_item_name">
                                     <text>{{item.city}}</text>
                                 </view>
                                 <view class="underline1f1"></view>
                             </view>
                         </view>
                     </scroll-view>
                 </view>
     
             </view>
     
         </view>
     
     </view>
     ```
     
     

2. `（item,i）in list`  item的名字不能重复都是 `item`



## 事件修饰符

1. `.stop`：各平台均支持， 使用时会阻止事件冒泡，在非 H5 端同时也会阻止事件的默认行为
2. `.native`：监听原生事件，仅在 H5 平台支持
3. `.prevent` 仅在 H5 平台支持
4. `.self`：仅在 H5 平台支持
5. `.once`：仅在 H5 平台支持
6. `.capture`：仅在 H5 平台支持
7. `.passive`：仅在 H5 平台支持







## 获取滚动高度

1. ```js
   
   与onLoad() {
    	// 获取硬件信息 获取屏幕宽度   
       uni.getSystemInfo({
           success: (screenWidth) => {
               this.windowWidth = screenWidth.screenWidth
           }
       })
   },
   
   // 与onLoad 同级
   onPageScroll(Object) {
       // #ifdef H5
       if (Object.scrollTop >= (this.windowWidth * (700 / 750))) {
           this.CHANGE_SCROLL(true)
       } else {
           this.CHANGE_SCROLL(false)
       }
       // #endif
   
       // #ifdef MP-WEIXIN
       if (Object.scrollTop >= (this.windowWidth * (700 / 750))) {
           this.CHANGE_SCROLL(true)
       } else {
           this.CHANGE_SCROLL(false)
       }
       // #endif
   
   },
   ```

   


## 从手机选择相片进行上传（文件流上传）

```js
uni.chooseImage({
    count: 1,
    success: (res) => {
        // 正面 人像
        if (this.chooseId === 0) {
            this.portrait = res.tempFilePaths[0]  // 临时路径
            uni.uploadFile({
                // 线上
                url: "https://api.kintting.com/api/user/uploadAuthentic",
                header: {
                    "token": this.loginToken
                },
                filePath: this.portrait,
                name: "file",
                formData:{
                  "/path":"/about"  
                }, // 图片其他的 formData参数 传参无效的话可以选择参数写在url地址中
                success: (res) => {
                    let data = JSON.parse(res.data)
                    this.portrait = data.url
                }
            })

        } else { // 反面 国徽
            this.national = res.tempFilePaths[0]
            uni.uploadFile({
                url: "https://api.kintting.com/api/user/uploadAuthentic",
                header: {
                    "token": this.loginToken
                },
                filePath: this.national,
                name: "file",
                success: (res) => {
                    let data = JSON.parse(res.data)
                    this.national = data.url
                }
            })
        }
        this.bottomTip = false
    }
})
```





## 照片裁剪上传

### 1. 引入 uview 插件

### 2. pages.json 文件中 注册地址  裁剪页是是一个新的页面

- ```json
  {
      "path": "uview-ui/components/u-avatar-cropper/u-avatar-cropper",
      "style": {
          "navigationBarTitleText": "头像裁剪",
          "navigationBarBackgroundColor": "#000000"
      }
  }
  ```

### 3. 通过点击事件 进入裁剪页面

- ```js
  choose_logo() {
      this.change_dark_logo()
      var windowWidth
      uni.getSystemInfo({
          success: function(res) {
              // console.log(res.windowWidth);
              windowWidth = res.windowWidth
          }
      });
      // 此为uView的跳转方法，详见"文档-JS"部分，也可以用uni的uni.navigateTo
      this.$u.route({
          url: '/uview-ui/components/u-avatar-cropper/u-avatar-cropper',
          // 内部已设置以下默认参数值，可不传这些参数
          params: {
              // 输出图片宽度，高等于宽，单位px
              destWidth: windowWidth,
              destHeight: windowWidth / 2, // 自定义的参数 需要去修改源码
              // 裁剪框宽度，高等于宽，单位px
              rectWidth: windowWidth,
              rectHeight: windowWidth / 2, // 自定义的参数 需要去修改源码
              // 输出的图片类型，如果'png'类型发现裁剪的图片太大，改成"jpg"即可
              fileType: 'png',
          }
      })
  },
  ```

### 4. 监听剪裁，获取剪裁后的图片地址 用于上传服务器

- ```js
  onLoad(){
      // 监听裁剪之后的图片
      uni.$on('uAvatarCropper', path => {
          uni.uploadFile({
              url: this.$baseUrl + "/companyapi/common/oUpload", // 接口地址
              header: {},
              filePath: path,
              name: "file",
              formData: {
                  path: '/about'
              },
              success: res => {
                  let data = JSON.parse(res.data)
                  this.logo = data.url
                  uni.showToast({
                      title: data.msg,
                      icon: "none",
                      success: () => {
                          setTimeout(() => {
                              uni.hideToast()
                          }, 1000)
                      }
                  })
              },
              complete: () => {
                  setTimeout(() => {
                      uni.hideLoading()
                  })
              }
          })
      })
  }
  ```


### 5. 移除剪裁监听事件

- ```js
  onUnload() {
      // 移除自定义事件监听器。不然各个页面中的图像剪裁 可能会冲突
      uni.$off('uAvatarCropper')
  }
  ```

  




## 对后台取到的数据处理 （绑定属性）

1. ```js
   // 获取已发布的二手需求
   async getMyDemand() {
       const res = await this.$myRequest("POST", "/api/user/getMyDemand", {
           pageNum: this.pageNum,
           pageSize: this.pageSize
       })
       if (res.code === 0) {
           this.list = res.data.rows
           this.list.forEach((item, index, arr) => {
               // this.$set(被添加的对象, 添加的属性名 , 添加的属性值) 
               this.$set(item, 'checked', false) // checked 标记 属性
               this.$set(item, 'show', false) // show  控制属性
           })
       } else {
           return
       }
   },
   ```





## 函数封装

```js
// 返回上一级页面函数 （适配H5 微信小程序  小程序等）
function return_page(router){
	let canBack = true
	const pages = getCurrentPages()
	// 有可返回的页面则直接返回，uni.navigateBack默认返回失败之后会自动刷新页面 ，无法继续返回  
	if (pages.length > 1) {
		uni.navigateBack(1)
		return;
	}else{
		uni.reLaunch({
			url: "/pages/index/index"
		})
	}
	// vue router 可以返回uni.navigateBack失败的页面 但是会重新加载  
	let a = router.go(-1)
	// router.go失败之后则重定向到首页  
	if (a == undefined) {
		uni.reLaunch({
			url: "/pages/index/index"
		})
	}
	return;
	// endif  
	uni.navigateBack(1)
}


// 图片放大预览  (兼容ios)
function swiper_preview(index, imgList) {
	var b = [...imgList]

	// Content-Type=image/jpg 兼容ios
    // BASE_IMG_URL 是网络图片路径前缀 例如 "https://m.kintting.cn"
	b.forEach((item, index, arr) => {
		arr[index] = `${BASE_IMG_URL}${item}?Content-Type=image/jpg`
	})

	// console.log(b);

	uni.previewImage({
		current: index,
		urls: b,
	})
	// return
}



// 时间排序 评论
reverse_comment() {
    this.last = !this.last
    if (this.last === true) {
        // 从大到小
        this.demand_comment.sort((a, b) => {
            return b.createTime < a.createTime ? -1 : 1
        })
    } else {
        // 从小到大
        this.demand_comment.sort((a, b) => {
            return b.createTime < a.createTime ? 1 : -1
        })
    }

},

    
    
// 时间管理  (几分钟前，几小时前，几天前，几月前，几年前)
function setTime(value) {
    // 正则表达式 以 / 替换 -  是为了兼容iOS  ios时间格式为 “2020/01/01”
    let oldDate = new Date(value.replace(/-/g, '/'));
    let newDate = new Date();
    var dayNum = "";
    var getTime = (newDate.getTime() - oldDate.getTime()) / 1000; // 转换成秒 的差距
    if (getTime < 60 * 5) {
        dayNum = "刚刚";
    } else if (getTime >= 60 * 5 && getTime < 60 * 60) {
        dayNum = parseInt(getTime / 60) + "分钟前";
    } else if (getTime >= 3600 && getTime < 3600 * 24) {
        dayNum = parseInt(getTime / 3600) + "小时前";
    } else if (getTime >= 3600 * 24 && getTime < 3600 * 24 * 30) {
        dayNum = parseInt(getTime / 3600 / 24) + "天前";
    } else if (getTime >= 3600 * 24 * 30 && getTime < 3600 * 24 * 30 * 12) {
        dayNum = parseInt(getTime / 3600 / 24 / 30) + "个月前";
    } else if (getTime >= 3600 * 24 * 30 * 12) {
        dayNum = parseInt(getTime / 3600 / 24 / 30 / 12) + "年前";
        // dayNum = value
    }
    return dayNum;
}



// 获取包含参数的完整路径 （微信小程序 H5）
function getCurrentPageUrlWithArgs() {
	var pages = getCurrentPages() //获取加载的页面
	var currentPage = pages[pages.length - 1] //获取当前页面的对象
	var url = currentPage.route //当前页面url
	var options = currentPage.options //如果要获取url中所带的参数可以查看options

	//拼接url的参数
	var urlWithArgs = url + '?'
	for (var key in options) {
		var value = options[key]
		urlWithArgs += key + '=' + value + '&'
	}
	urlWithArgs = urlWithArgs.substring(0, urlWithArgs.length - 1)

	return urlWithArgs
}


// 图片url转 base64 （不支持网络路径）
function getBase64(url, callback) {
  var Img = new Image(),
  dataURL = '';
  Img.src = url + '?v=' + Math.random();
  Img.setAttribute('crossOrigin', 'Anonymous');
  Img.onload = function() {
    var canvas = document.createElement('canvas'),
    width = Img.width,
    height = Img.height;
    canvas.width = width;
    canvas.height = height;
    canvas.getContext('2d').drawImage(Img, 0, 0, width, height);
    dataURL = canvas.toDataURL('image/jpeg');
    return callback ? callback(dataURL) : null;
  };
}


//字符串序列化
function urlstringify(obj) {
	var str = '';
	for (let key in obj) {
		if (Object.prototype.toString.call(obj[key]) === '[object Array]' || obj[key].constructor === Object) {
			//数组,对象
			for (var arrKey in obj[key]) {
				if (Object.prototype.toString.call(obj[key][arrKey]) === '[object Array]' || obj[key][arrKey].constructor ===
					Object) {
					//数组,对象
					for (var arrarrKey in obj[key][arrKey]) {
						str += '&' + key + '[' + arrKey + '][' + arrarrKey + ']=' + obj[key][arrKey][arrarrKey];
					}
				} else {
					//普通
					str += '&' + key + '[' + arrKey + ']=' + obj[key][arrKey];
				}
			}
		} else {
			//普通
			str += '&' + key + '=' + obj[key];
		}
	}
	return str.substring(1);
}


// 定时器
function time(){
    let vcode = 60
    let timer = setInterval(() => {
        vcode--
        if (vcode <= 0) {
            clearInterval(timer)
         	vcode = 60
        }
    }, 1000)
    }

```




## 取页面栈中的数据方法路由等

1. ```js
   console.log(getCurrentPages())
   ```




## 组件的复用

1. 组件 （组件中写静态方法 不与后端交互）

   ```html
   <template>
   	<view class="image" @tap="onClick" >
   		<image
   			:src="src"
   		></image>
   	</view>
   </template>
   
   <script>
   export default {
       name:"u-img",
   	props: {
   		// 图片地址
   		src: {
   			type: String,
   			default: ''
   		},
   	},
   	data() {
   		return {
   			
   		};
   	},
   	methods: {
   		// 点击图片
   		onClick() {
   			this.$emit('comClick',data);
   		},
   	}
   };
   </script>
   ```

2. 页面（页面中写 ajax 调用）

   ```html
   
   <template>
   	<u-img :src="srcUrl" @comClick="change" ></u-img>
   </template>
   
   <script>
   export default {
   	data() {
   		return {
   			srcUrl:"..."
   		};
   	},
   	methods: {
   		change(data) {
   			....
               // data 是子组件传过来的数据
   		},
   	}
   };
   </script>
   
   ```



## 富文本编辑器 editor组件

1. editor.vue
   
   ```html
   <template>
       <view class="container">
        <view class="page-body">
               <view class='wrapper' v-if="editor_show">
                   <editor id="editor" class="ql-container_up" placeholder="从这里开始分享"
                           showImgToolbar  showImgResize @statuschange="onStatusChange" :read-only="readOnly" @ready="onEditorReady"
                           @input="onEditorBlur">
                   </editor> 
                   <!-- <rich-text :nodes="htmlFile" @click="editor_show=true" v-if="editor_show===false"></rich-text> -->
   
   
                   <!-- 更换标签 -->
                   <view class="dark" v-if="show_info_category" @click="show_info_category=false">
                       <view class="dark_view">
                           <view class="change_tag_nav">
                               <view class="change_tag_label">
                                   更换标签
                               </view>
                               <view class="tag_choose_cancel" @click="show_info_category=true">
                                   取消
                               </view>
                           </view>
   
                           <view class="tag_item_con" @click.top="">
                               <view class="tag_item" v-for="(item,index) in info_category_list" :key="index" @click.stop="confirm_category(item.id,item.name)">
                                   <text class="tag_logo">#</text>
                                   <text>{{item.name}}</text>
                               </view>
                           </view>
                       </view>
                   </view>
   
   
   
                   <!--  正常的 工具栏 -->
                   <view class='toolbar' :class="{'up':scroll_show}" @tap="format">
                       <!-- 分类名称 -->
                       <view class="info_category" >
                           <view class="" @tap="show_info_category=true">
                               # {{categoryName}} <text class="dark_right"></text>
                           </view>
                           <view class="" style="flex: 1;">
   
                           </view>
                       </view>
   
                       <!-- 首行 -->
                       <view class="toolbar_line_0">
                           <view class="toolbar_line_0_left">
                               <view :class="scroll_show ? 'active_color_icon' : ''" class="iconfont blue_color_icon" 
                                     @click="change_stretch"></view>
                               <view class="iconfont icon_zuoduiqi" @click="choose_align"></view>
                               <view class="iconfont insert_img" @tap="insertImage"></view>
                               <view class="iconfont icon_undo" @tap="undo"></view>
                               <view class="iconfont icon_redo" @tap="redo"></view>
                           </view>
                           <view class="toolbar_line_0_right" @click="change_stretch">
                               <view class="push_or_pop">
                                   <image :src="scroll_show ? '/static/img/info/pop.png' : '/static/img/info/push.png' " mode=""></image>
                               </view>
                           </view>
                       </view>
   
   
                       <!-- 选择 对齐四个方式 -->
                       <scroll-view style="height: 40vh; padding-bottom: 20rpx; box-sizing: border-box;" v-if="scroll_tab === 1">
                           <view class="align_con">
                               <view :class="formats.align === 'left' ? 'skyblue' : ''" data-name="align" data-value="left">
                                   <view :class="formats.align === 'left' ? 'active_icon_zuoduiqi' : ''" class="iconfont icon_zuoduiqi" data-name="align"
                                         data-value="left"></view>
                               </view>
                               <view :class="formats.align === 'center' ? 'skyblue' : ''" data-name="align" data-value="center">
                                   <view :class="formats.align === 'center' ? 'ql-active' : ''" class="iconfont icon_juzhongduiqi" data-name="align"
                                         data-value="center"></view>
                               </view>
                               <view :class="formats.align === 'right' ? 'skyblue' : ''" data-name="align" data-value="right">
                                   <view :class="formats.align === 'right' ? 'ql-active' : ''" class="iconfont icon_youduiqi" data-name="align"
                                         data-value="right"></view>
                               </view>
                               <view :class="formats.align === 'justify' ? 'skyblue' : ''" data-name="align" data-value="justify">
                                   <view :class="formats.align === 'justify' ? 'ql-active' : ''" class="iconfont icon_zuoyouduiqi" data-name="align"
                                         data-value="justify"></view>
                               </view>
                           </view>
                       </scroll-view>
   
                       <!-- 全部选项 -->
                       <scroll-view scroll-y="true" style="height: 40vh; padding-bottom: 20rpx; box-sizing: border-box;" v-if="scroll_tab === 0">
                           <!-- 加粗 斜体 删除线 下划线 -->
                           <view class="toolbar_line_1">
                               <view :class="formats.bold ? 'ql-active' : ''" class="iconfont icon-zitijiacu" data-name="bold"></view>
                               <view :class="formats.italic ? 'ql-active' : ''" class="iconfont icon-zitixieti" data-name="italic"></view>
                               <view :class="formats.strike ? 'ql-active' : ''" class="iconfont icon-zitishanchuxian" data-name="strike"></view>
                               <view :class="formats.underline ? 'ql-active' : ''" class="iconfont icon-zitixiahuaxian" data-name="underline"></view>
                           </view>
   
                           <!-- 标题1 标题2 -->
                           <view class="toolbar_line_2">
                               <view :class="formats.header === 1 ? 'ql-active skyblue' : ''" class="iconfont " data-name="header" :data-value="1">标题1</view>
                               <view :class="formats.header === 2 ? 'ql-active skyblue' : ''" class="iconfont " data-name="header" :data-value="2">标题2</view>
                           </view>
   
                           <!-- 标题3 标题4 原文 -->
                           <view class="toolbar_line_2">
                               <view :class="formats.header === 3 ? 'ql-active skyblue' : ''" class="iconfont " data-name="header" :data-value="3">标题3</view>
                               <view :class="formats.header === 4 ? 'ql-active skyblue' : ''" class="iconfont " data-name="header" :data-value="4">标题4</view>
                               <view :class="formats.header=== undefined ? 'ql-active skyblue' : ''" class="iconfont " data-name="header">正文</view>
                           </view>
   
                           <!-- 文字颜色选择 -->
                           <view class="toolbar_line_4" @click="show_color">
                               <view class="toolbar_line_left">
                                   <view :class="formats.color === '#0000ff' ? 'ql-active' : ''" class="iconfont icon_text_color position"
                                         data-name="color">
                                       <view class="color_block" v-if="showColor">
                                           <view class="color_item_block red" data-name="color" data-value="#ff0000"></view>
                                           <view class="color_item_block orange" data-name="color" data-value="#FF8800"></view>
                                           <view class="color_item_block yellow" data-name="color" data-value="#FFFF00"></view>
                                           <view class="color_item_block green" data-name="color" data-value="#00FF00"></view>
                                           <view class="color_item_block blue" data-name="color" data-value="#0000FF "></view>
                                           <view class="color_item_block purple" data-name="color" data-value="#FF00FF"></view>
                                           <view class="color_item_block black" data-name="color" data-value="#000000"></view>
                                           <view class="color_item_block white" data-name="color" data-value="#ffffff"></view>
                                           <view class="color_item_block skyblue" data-name="color" data-value="#00ffff"></view>
                                       </view>
                                   </view>
   
                                   <view class="toolbar_line_left_label">
                                       文本颜色
                                   </view>
                               </view>
                               <view class="toolbar_line_right">
                                   <view class="color_ball" :style="{'backgroundColor': textColor}">
                                   </view>
                                   <view class="toolbar_line_right_arrow">
                                   </view>
                               </view>
                           </view>
   
                           <!-- 背景颜色选择 -->
                           <view class="toolbar_line_4" @click="show_bg_color">
                               <view class="toolbar_line_left">
                                   <view :class="formats.backgroundColor === '#00ff00' ? 'ql-active' : ''" class="iconfont icon_fontbgcolor position"
                                         data-name="backgroundColor" data-value="#00ff00">
                                       <view class="bg_block" v-if="showBg">
                                           <view class="color_item_block red" data-name="backgroundColor" data-value="#ff0000"></view>
                                           <view class="color_item_block orange" data-name="backgroundColor" data-value="#FF8800"></view>
                                           <view class="color_item_block yellow" data-name="backgroundColor" data-value="#FFFF00"></view>
                                           <view class="color_item_block green" data-name="backgroundColor" data-value="#00FF00"></view>
                                           <view class="color_item_block blue" data-name="backgroundColor" data-value="#0000FF "></view>
                                           <view class="color_item_block purple" data-name="backgroundColor" data-value="#FF00FF"></view>
                                           <view class="color_item_block black" data-name="backgroundColor" data-value="#000000"></view>
                                           <view class="color_item_block white" data-name="backgroundColor" data-value="#ffffff"></view>
                                           <view class="color_item_block skyblue" data-name="backgroundColor" data-value="#00ffff"></view>
                                       </view>
                                   </view>
   
                                   <view class="toolbar_line_left_label">
                                       背景颜色
                                   </view>
                               </view>
                               <view class="toolbar_line_right">
                                   <view class="color_ball" :style="{'backgroundColor': bgColor}">
                                   </view>
                                   <view class="toolbar_line_right_arrow">
                                   </view>
                               </view>
                           </view>
   
   
                           <!-- 字体尺寸 选择 -->
                           <view class="toolbar_line_6">
                               <view class="toolbar_line_left">
                                   <view class="iconfont icon_fontsize"></view>
                                   <view class="toolbar_line_left_label">
                                       字号
                                   </view>
                               </view>
   
                               <view class="toolbar_line_6_right">
                                   <view class="" @click.stop="size_sub">
                                       ➖
                                   </view>
                                   <view class="">
                                       {{text_size}}
                                   </view>
                                   <view class="" @click.stop="size_add">
                                       ➕
                                   </view>
                               </view>
                           </view>
   
                           <!-- 清楚样式 -->
                           <view class="toolbar_line_6" @tap="removeFormat">
                               <view class="toolbar_line_left">
                                   <view class="iconfont icon_clearedformat" @tap="removeFormat"></view>
                                   <view class="toolbar_line_left_label">
                                       清除样式
                                   </view>
                               </view>
                           </view>
                       </scroll-view>
   
   
   
   
   
                       <!-- <view :class="formats.align === 'left' ? 'ql-active' : ''" class="iconfont icon-zuoduiqi" data-name="align"
   data-value="left"></view>
   <view :class="formats.align === 'center' ? 'ql-active' : ''" class="iconfont icon-juzhongduiqi" data-name="align"
   data-value="center"></view>
   <view :class="formats.align === 'right' ? 'ql-active' : ''" class="iconfont icon-youduiqi" data-name="align"
   data-value="right"></view>
   <view :class="formats.align === 'justify' ? 'ql-active' : ''" class="iconfont icon-zuoyouduiqi" data-name="align"
   data-value="justify"></view>
   <view :class="formats.lineHeight ? 'ql-active' : ''" class="iconfont icon-line-height" data-name="lineHeight"
   data-value="2"></view>
   <view :class="formats.letterSpacing ? 'ql-active' : ''" class="iconfont icon-Character-Spacing" data-name="letterSpacing"
   data-value="2em"></view>
   <view :class="formats.marginTop ? 'ql-active' : ''" class="iconfont icon-722bianjiqi_duanqianju" data-name="marginTop"
   data-value="20px"></view>
   <view :class="formats.previewarginBottom ? 'ql-active' : ''" class="iconfont icon-723bianjiqi_duanhouju" data-name="marginBottom"
   data-value="20px"></view>
   <view class="iconfont icon-clearedformat" @tap="removeFormat"></view>
   <view :class="formats.fontFamily ? 'ql-active' : ''" class="iconfont icon-font" data-name="fontFamily" data-value="Pacifico"></view>
   <view :class="formats.fontSize === '24px' ? 'ql-active' : ''" class="iconfont icon-fontsize" data-name="fontSize"
   data-value="24px"></view>
   
   
   <view :class="formats.color === '#0000ff' ? 'ql-active' : ''" class="iconfont icon-text_color position" data-name="color"
   data-value="#0000FF">
   </view>
   
   
   
   <view :class="formats.backgroundColor === '#00ff00' ? 'ql-active' : ''" class="iconfont icon-fontbgcolor position"
   data-name="backgroundColor" data-value="#00ff00">
   
   </view>
   
   <view class="iconfont icon-date" @tap="insertDate"></view>
   <view class="iconfont icon--checklist" data-name="list" data-value="check"></view>
   <view :class="formats.list === 'ordered' ? 'ql-active' : ''" class="iconfont icon-youxupailie" data-name="list"
   data-value="ordered"></view>
   <view :class="formats.list === 'bullet' ? 'ql-active' : ''" class="iconfont icon-wuxupailie" data-name="list"
   data-value="bullet"></view>
   <view class="iconfont icon-undo" @tap="undo"></view>
   <view class="iconfont icon-redo" @tap="redo"></view>
   
   <view class="iconfont icon-outdent" data-name="indent" data-value="-1"></view>
   <view class="iconfont icon-indent" data-name="indent" data-value="+1"></view>
   <view class="iconfont icon-fengexian" @tap="insertDivider"></view>
   <view class="iconfont icon-charutupian" @tap="insertImage"></view>
   <view :class="formats.header === 1 ? 'ql-active' : ''" class="iconfont icon-format-header-1" data-name="header"
   :data-value="1"></view>
   <view :class="formats.script === 'sub' ? 'ql-active' : ''" class="iconfont icon-zitixiabiao" data-name="script"
   data-value="sub"></view>
   <view :class="formats.script === 'super' ? 'ql-active' : ''" class="iconfont icon-zitishangbiao" data-name="script"
   data-value="super"></view>
   <view class="iconfont icon-shanchu" @tap="clear"></view>
   <view :class="formats.direction === 'rtl' ? 'ql-active' : ''" class="iconfont icon-direction-rtl" data-name="direction"
   data-value="rtl"></view> -->
   
                   </view>
               </view>
           </view>
   
       </view>
   </template>
   
   <script src="./editor.js"></script>
   
   <style>
       @import "./editor-icon.css";
   
       .container {
           height: 100%;
       }
   
       .page-body {
           height: 100%;
       }
   
       .wrapper {
           height: 100%;
           display: flex;
           flex-direction: column;
           justify-content: space-between;
       }
   
       .iconfont {
           display: inline-block;
           /* padding: 8rpx 8rpx;
           width: 24rpx;
           height: 24rpx;
           cursor: pointer;
           font-size: 20rpx; */
       }
   
       /* .toolbar {
       box-sizing: border-box;
       border-bottom: 0;
       font-family: 'Helvetica Neue', 'Helvetica', 'Arial', sans-serif;
       } */
   
   
       .ql-container {
           box-sizing: border-box;
           padding: 12rpx 15rpx;
           width: 100%;
           height: auto;
           background: #fff;
           margin-top: 20rpx;
           /* min-height: 30vh; */
           /* font-size: 16rpx; */
           /* line-height: 1.5; */
       }
   
       .ql-active {
           color: #06c !important;
       }
   </style>
   
   ```
   
2. editor.js
   
   ```js
   export default {
   	props: {
   		info_category_list: {
   			default: () => {
   				return []
   			}
   		},
   		categoryName: {
   			default: ""
   		},
   		htmlFile: {
   			default: ""
   		}
   	},
   	watch: {
		text_size() {
   			if (this.text_size < 1) {
   				this.text_size = 1
   			}
   		},
   	},
   	data() {
   		return {
   			
   			token:"",
   			
   			editor_show: false,
   
   			readOnly: false,
   			formats: {
   				align: "left",
   			},
   			showColor: false,
   			showBg: false,
   			textColor: "#000", // 文字颜色
   			bgColor: "#fff", // 背景颜色
   			text_size: 16, // 文字大小
   			scroll_show: false, // 选项的展开 与 合并
   			scroll_tab: 0, // 展示的scroll-view
   
   			show_info_category: false, // 展示分类选择
   			// categoryName: "大圆机", // 选中的分类名字
   			categoryId: 1, // 选中的分类id
   		}
   	},
   	methods: {
   		// 控制富文本 的升降
   		change_stretch() {
   			this.scroll_show = !this.scroll_show
   		},
   
   		choose_align() {
   			if (this.scroll_show === false) {
   				this.change_stretch()
   			} else {
   				this.change_tab_index(1)
   			}
   		},
   
   		// 改变 scroll-view 的页面 索引
   		change_tab_index(index) {
   			if (this.scroll_tab === 1) {
   				this.scroll_tab = 0
   			} else {
   				this.scroll_tab = index
   			}
   		},
   
   		// 展示文字颜色选择
   		show_color(e) {
   			this.showColor = !this.showColor
   			this.showBg = false
   		},
   		// 展示背景颜色选择
   		show_bg_color() {
   			this.showColor = false
   			this.showBg = !this.showBg
   		},
   
   		// 文字尺寸 增加
   		size_add() {
   			this.text_size++
   			this.set_size()
   		},
   
   		// 文字尺寸减少
   		size_sub() {
   			this.text_size--
   			this.set_size()
   		},
   
   		// 尺寸作用于富文本
   		set_size() {
   			let value = `${this.text_size}px`
   			this.editorCtx.format("fontSize", value)
   		},
   
   		// 确定分类
   		confirm_category(id, name) {
   			// this.categoryId = id
   			// this.categoryName = name
   			this.$emit("triggerCategory", id, name)
   			this.show_info_category = false
   		},
   
   
   		readOnlyChange() {
   			this.readOnly = !this.readOnly
   		},
   
   		// 初始化 富文本编辑器
   		onEditorReady() {
   			uni.createSelectorQuery().in(this).select('#editor').context((res) => {
   				this.editorCtx = res.context
   
   				uni.getSystemInfo({
   					success: res => {
   						this.text_size = parseInt((res.screenWidth / 750 * 30))
   					}
   				});
   
   				this.editorCtx.setContents({
   					html: this.htmlFile ? this.htmlFile : ' ',
   					success: function(res) {
   						console.log("数据渲染成功")
   					},
   					fail: function(err) {
   						console.log("渲染失败")
   					}
   				})
   
   			}).exec()
   		},
   
   		// 撤销
   		undo() {
   			this.editorCtx.undo()
   		},
   		// 还原
   		redo() {
   			this.editorCtx.redo()
   		},
   
   		// 监听点击事件
   		format(e) {
   			// console.log(e.target.dataset);
   			let {
   				name,
   				value
   			} = e.target.dataset
   			if (!name) return
   			if (name == "color") {
   				this.textColor = value
   			}
   			if (name == "backgroundColor") {
   				this.bgColor = value
   			}
   			this.editorCtx.format(name, value)
   			this.showColor = false
   			this.showBg = false
   			if (name === "align") {
   				this.scroll_tab = 0
   			}
   		},
   		onStatusChange(e) {
   			const formats = e.detail
   			this.formats = formats
   		},
   		insertDivider() {
   			this.editorCtx.insertDivider({
   				success: function() {
   					console.log('insert divider success')
   				}
   			})
   		},
   		clear() {
   			this.editorCtx.clear({
   				success: function(res) {
   					console.log("clear success")
   				}
   			})
   		},
   		// 移除样式
   		removeFormat() {
   			this.editorCtx.removeFormat()
   			this.textColor = "#000"
   			this.bgColor = "#fff"
   			uni.getSystemInfo({
   				success: res => {
   					this.text_size = parseInt((res.screenWidth / 750 * 30))
   				}
   			});
   		},
   		insertDate() {
   			const date = new Date()
   			const formatDate = `${date.getFullYear()}/${date.getMonth() + 1}/${date.getDate()}`
   			this.editorCtx.insertText({
   				text: formatDate
   			})
   		},
   		onEditorBlur(e) {
   			// console.log(e.detail);
   			this.$emit("triggerBlue", e.detail)
   		},
   		insertImage() {
   			uni.chooseImage({
   				count: 1,
   				success: (res) => {
   					uni.uploadFile({
   						// url: this.$baseUrl + "/companyapi/common/upload",
   						url: this.$baseUrl + "/companyapi/common/upload?token=" + this.token + "&path=" + "/new",
   						header: {},
   						formData: {
   							path: "/new",
   							// token:this.token
   						},
   						filePath: res.tempFilePaths[0],
   						name: "file",
   						complete: (res) => {
   							let data = JSON.parse(res.data)
   							console.log(data);
   							this.editorCtx.insertImage({
   								src: this.$baseImgUrl + data.url,
   								alt: '图像',
   								width: "100%",
   								success: function() {
   									console.log('insert image success')
   								}
   							})
   						}
   					});
   
   					// this.editorCtx.insertImage({
   					// 	src: res.tempFilePaths[0],
   					// 	alt: '图像',
   					// 	success: function() {
   					// 		console.log('insert image success')
   					// 	}
   					// })
   				}
   			})
   		},
   	},
   	created() {
   		setTimeout(() => {
   			this.editor_show = true
   		}, 500)
   		
   		uni.getStorage({
   			key: "loginToken",
   			success: (res) => {
   				this.token = res.data
   			}
   		})
   	},
   	onLoad() {
   		// uni.loadFontFace({
   		// 	family: 'Pacifico',
   		// 	source: 'url("https://sungd.github.io/Pacifico.ttf")'
   		// })
   	},
   }
   ```
   
3. editor.css

   ```css
   /* 详见解决css引入字体跨域问题 */
   @font-face{
       font-family: 'iconfont';
       src: url('data:application/font-woff2;charset=utf-8;base64,......') format('woff2'),
           url('data:application/font-woff;charset=utf-8;base64,.....') format('woff'),
           url('iconfont.ttf') format('truetype');
       /* url('iconfont.ttf') 是iconfont.ttf的文件路径 （相对路径或绝对路径） */
       font-weight: normal;
       font-style: normal;
       font-display: swap;
   }
   
   
   /* 编辑器 区域 */
   #editor {}
   
   #editor /deep/ .ql-editor.ql-blank:before {
   	font-style: normal;
   	font-size: 30rpx;
   }
   
   .wrapper .ql-container_up {
   	margin: 0;
   	padding: 0 22rpx;
   	font-size: 30rpx;
   	/* height: 300rpx; */
   	flex: 1;
   	min-height: 50rpx;
   }
   
   .wrapper .ql-container_down{
   	margin: 0;
   	padding: 0 22rpx;
   	font-size: 30rpx;
   	/* height: 800rpx; */
   	flex: 1;
   	min-height: 50rpx;
   }
   
   
   /* 工具区域 */
   .toolbar {
   	width: 750rpx;
   	background: #F2F4F5;
   	font-size: 28rpx;
   	color: #191919;
   	height: 180rpx;
   	transition-duration: 0.5s;
   	z-index: 9;
   	overflow: hidden;
   }
   
   .up {
   	overflow: hidden;
   	height: calc(40vh + 200rpx);
   	transition-duration: 0.5s;
   }
   
   /* 始终出现得 第一行 */
   .toolbar_line_0 {
   	background-color: #fff;
   	display: flex;
   	justify-content: space-between;
   	align-items: center;
   	padding: 22rpx 28rpx 28rpx;
   }
   
   .toolbar_line_0_left {
   	flex: 2;
   	display: flex;
   	justify-content: space-between;
   	align-items: center;
   }
   
   .toolbar_line_0_right {
   	flex: 1.3;
   	display: flex;
   	justify-content: flex-end;
   }
   
   .push_or_pop{
   	display: flex;
   	justify-content: flex-end;
   }
   
   .toolbar_line_0_right .push_or_pop image {
   	margin-right: 8rpx;
   	width: 44rpx;
   	height: 44rpx;
   }
   
   
   
   
   /* 工具行 */
   .toolbar_line_1,
   .toolbar_line_2,
   .toolbar_line_4,
   .toolbar_line_6 {
   	font-size: 28rpx;
   	color: #191919;
   
   	box-sizing: border-box;
   	height: 82rpx;
   	margin: 24rpx 32rpx 0 32rpx;
   	display: flex;
   	justify-content: space-around;
   	align-items: center;
   	background: #FFFFFF;
   	border: 1rpx solid #E6E8E9;
   	border-radius: 6rpx;
   }
   
   
   .toolbar_line_1 view,
   .toolbar_line_2 view {
   	flex: 1;
   	height: 100%;
   	border-right: 1rpx solid #E6E8E9;
   	display: flex;
   	justify-content: center;
   	align-items: center;
   }
   
   .toolbar_line_4>view {
   	flex: 1;
   	display: flex;
   	align-items: center;
   }
   
   .toolbar_line_left {
   	padding: 0 26rpx;
   }
   
   .toolbar_line_left_label {
   	margin-left: 22rpx;
   }
   
   .toolbar_line_right {
   	padding-right: 36rpx;
   	display: flex;
   	align-items: center;
   	justify-content: flex-end;
   }
   
   .color_ball {
   	width: 40rpx;
   	height: 40rpx;
   	background: #404040;
   	border-radius: 50%;
   	margin-right: 28rpx;
   }
   
   .toolbar_line_right_arrow {
   	width: 12rpx;
   	height: 18rpx;
   	background: url(/static/img/index/right.png);
   	background-size: 100% 100%;
   }
   
   
   /* 字号 */
   .toolbar_line_6>view {
   	flex: 1;
   	display: flex;
   	align-items: center;
   }
   
   .toolbar_line_6 .toolbar_line_6_right {
   	justify-content: space-between;
   	padding: 0 34rpx 0 80rpx;
   }
   
   .position {
   	position: relative;
   }
   
   .color_block {
   	position: absolute;
   	left: 0;
   	top: 60rpx;
   	width: 210rpx;
   	height: 210rpx;
   	display: flex;
   	flex-wrap: wrap;
   	background-color: #C8C7CC;
   	z-index: 9;
   }
   
   .bg_block {
   	position: absolute;
   	left: 0;
   	top: 60rpx;
   	width: 210rpx;
   	height: 210rpx;
   	display: flex;
   	flex-wrap: wrap;
   	background-color: #C8C7CC;
   	z-index: 9;
   }
   
   .color_item_block {
   	margin: 10rpx;
   	width: 50rpx;
   	height: 50rpx;
   }
   
   .red {
   	background-color: #ff0000;
   }
   
   .orange {
   	background-color: #FF8800;
   }
   
   .yellow {
   	background-color: #FFFF00;
   }
   
   .green {
   	background-color: #00FF00;
   }
   
   .blue {
   	background-color: #0000FF;
   }
   
   .purple {
   	background-color: #FF00FF;
   }
   
   .black {
   	background-color: #000000;
   }
   
   .white {
   	background-color: #ffffff;
   }
   
   .skyblue {
   	background-color: #00ffff;
   }
   
   
   
   /* 资讯分类 */
   .info_category {
   	padding: 32rpx;
   	font-size: 30rpx;
   	color: #4278FF;
   	border-bottom: 1rpx solid #EBEDEE;
   	background-color: #fff;
   	display: flex;
   }
   .dark_right {
   	margin-left: 18rpx;
   	display: inline-block;
   	width: 12rpx;
   	height: 20rpx;
   	background: url(/static/img/info/dark_right.png);
   	background-size: 100% 100%;
   }
   
   .dark {
   	position: fixed;
   	height: 100vh;
   	top: 0;
   	left: 0;
   	bottom: 0;
   	right: 0;
   	background: rgba(0, 0, 0, 0.7);
   	z-index: 9999;
   	display: flex;
   	flex-direction: column;
   	justify-content: flex-end;
   }
   
   .dark_view {
   	width: 750rpx;
   	background: #FFFFFF;
   	border-radius: 20rpx 20rpx 0rpx 0rpx;
   	box-sizing: border-box;
   	padding: 56rpx 32rpx 40rpx;
   }
   
   .change_tag_nav {
   	display: flex;
   	justify-content: space-between;
   	align-items: center;
   }
   
   .change_tag_label {
   	font-size: 30rpx;
   	font-weight: bold;
   	color: #191919;
   }
   
   .tag_choose_cancel {
   	font-size: 30rpx;
   	color: #9E9E9E;
   }
   
   .tag_item {
   	display: flex;
   	justify-content: flex-start;
   	align-items: center;
   
   	font-size: 30rpx;
   	color: #191919;
   	margin: 60rpx 16rpx 60rpx 0;
   }
   
   .tag_logo {
   	font-size: 30rpx;
   	font-weight: bold;
   	color: #4278FF;
   	margin-right: 16rpx;
   }
   
   /* 对齐方式 */
   .align_con{
   	display: flex;
   	justify-content: space-between;
   	align-items: center;
   	margin: 40rpx 32rpx;
   	background-color: #fff;
   	border: 1rpx solid #E6E8E9;
   	border-radius: 6rpx;
   }
   .align_con>view{
   	flex: 1;
   	box-sizing: border-box;
   	border-right: 1rpx solid #E6E8E9;
   	display: flex;
   	justify-content: center;
   	align-items: center;
   	padding: 26rpx 0;
   }
   
   
   
   /* icon 集合 */
   .blue_color_icon {
   	width: 40rpx;
   	height: 36rpx;
   	background: url(/static/img/info/aa.png);
   	background-size: 100% 100%;
   }
   
   .active_color_icon {
   	width: 40rpx;
   	height: 36rpx;
   	background: url(/static/img/info/active_aa.png);
   	background-size: 100% 100%;
   }
   
   
   /* 二级页面的 四个对齐 */
   .skyblue{
   	background: #DEEFFD;
   	border-radius: 6rpx 0rpx 0rpx 6rpx;
   }
   
   .icon_zuoduiqi {
   	width: 30rpx;
   	height: 26rpx;
   	background: url(/static/img/info/align_left.png);
   	background-size: 100% 100%;
   }
   
   .active_icon_zuoduiqi {
   	background: url(/static/img/info/active_align_left.png);
   	background-size: 100% 100%;
   }
   
   .icon_juzhongduiqi {
   	width: 30rpx;
   	height: 26rpx;
   	background: url(/static/img/info/align_center.png);
   	background-size: 100% 100%;
   }
   
   .active_icon_juzhongduiqi {
   	background: url(/static/img/info/active_align_left.png);
   	background-size: 100% 100%;
   }
   
   .icon_youduiqi {
   	width: 30rpx;
   	height: 26rpx;
   	background: url(/static/img/info/align_right.png);
   	background-size: 100% 100%;
   }
   
   .active_icon_youduiqi {
   	background: url(/static/img/info/active_align_left.png);
   	background-size: 100% 100%;
   }
   
   .icon_zuoyouduiqi {
   	width: 30rpx;
   	height: 26rpx;
   	background: url(/static/img/info/align_justify.png);
   	background-size: 100% 100%;
   }
   
   .active_icon_zuoyouduiqi {
   	background: url(/static/img/info/active_align_left.png);
   	background-size: 100% 100%;
   }
   
   
   .insert_img {
   	width: 40rpx;
   	height: 38rpx;
   	background: url(/static/img/info/insert_img.png);
   	background-size: 100% 100%;
   }
   
   .icon_text_color {
   	width: 34rpx;
   	height: 32rpx;
   	background: url(/static/img/info/text_color.png);
   	background-size: 100% 100%;
   }
   
   .icon_fontbgcolor {
   	width: 32rpx;
   	height: 34rpx;
   	background: url(/static/img/info/bg_color.png);
   	background-size: 100% 100%;
   }
   
   .icon_fontsize {
   	width: 32rpx;
   	height: 28rpx;
   	background: url(/static/img/info/font_size.png);
   	background-size: 100% 100%;
   }
   
   .icon_clearedformat {
   	width: 32rpx;
   	height: 28rpx;
   	background: url(/static/img/info/clear_format.png);
   	background-size: 100% 100%;
   }
   
   .icon_undo {
   	width: 35rpx;
   	height: 35rpx;
   	background: url(/static/img/info/active_undo.png);
   	background-size: 100% 100%;
   }
   .icon_redo {
   	width: 35rpx;
   	height: 35rpx;
   	background: url(/static/img/info/active_redo.png);
   	background-size: 100% 100%;
   }
   
   
   
   
   
   
   
   
   .iconfont {
   	font-family: "iconfont" !important;
   	/* font-size: 16px; */
   	font-style: normal;
   	-webkit-font-smoothing: antialiased;
   	-moz-osx-font-smoothing: grayscale;
   }
   
   .icon-redo:before {
   	content: "\e627";
   }
   
   .icon-undo:before {
   	content: "\e633";
   }
   
   .icon-indent:before {
   	content: "\eb28";
   }
   
   .icon-outdent:before {
   	content: "\e6e8";
   }
   
   .icon-fontsize:before {
   	content: "\e6fd";
   }
   
   .icon-format-header-1:before {
   	content: "\e860";
   }
   
   .icon-format-header-4:before {
   	content: "\e863";
   }
   
   .icon-format-header-5:before {
   	content: "\e864";
   }
   
   .icon-format-header-6:before {
   	content: "\e865";
   }
   
   .icon-clearup:before {
   	content: "\e64d";
   }
   
   .icon-preview:before {
   	content: "\e631";
   }
   
   .icon-date:before {
   	content: "\e63e";
   }
   
   .icon-fontbgcolor:before {
   	content: "\e678";
   }
   
   .icon-clearedformat:before {
   	content: "\e67e";
   }
   
   .icon-font:before {
   	content: "\e684";
   }
   
   .icon-723bianjiqi_duanhouju:before {
   	content: "\e65f";
   }
   
   .icon-722bianjiqi_duanqianju:before {
   	content: "\e660";
   }
   
   /* 文本颜色icon */
   .icon-text_color:before {
   	content: "\e72c";
   }
   
   .icon-format-header-2:before {
   	content: "\e75c";
   }
   
   .icon-format-header-3:before {
   	content: "\e75d";
   }
   
   .icon--checklist:before {
   	content: "\e664";
   }
   
   .icon-baocun:before {
   	content: "\ec09";
   }
   
   .icon-line-height:before {
   	content: "\e7f8";
   }
   
   .icon-quanping:before {
   	content: "\ec13";
   }
   
   .icon-direction-rtl:before {
   	content: "\e66e";
   }
   
   .icon-direction-ltr:before {
   	content: "\e66d";
   }
   
   .icon-selectall:before {
   	content: "\e62b";
   }
   
   .icon-fuzhi:before {
   	content: "\ec7a";
   }
   
   .icon-shanchu:before {
   	content: "\ec7b";
   }
   
   .icon-bianjisekuai:before {
   	content: "\ec7c";
   }
   
   .icon-fengexian:before {
   	content: "\ec7f";
   }
   
   .icon-dianzan:before {
   	content: "\ec80";
   }
   
   .icon-charulianjie:before {
   	content: "\ec81";
   }
   
   .icon-charutupian:before {
   	content: "\ec82";
   }
   
   .icon-wuxupailie:before {
   	content: "\ec83";
   }
   
   .icon-juzhongduiqi:before {
   	content: "\ec84";
   }
   
   .icon-yinyong:before {
   	content: "\ec85";
   }
   
   .icon-youxupailie:before {
   	content: "\ec86";
   }
   
   .icon-youduiqi:before {
   	content: "\ec87";
   }
   
   .icon-zitidaima:before {
   	content: "\ec88";
   }
   
   .icon-xiaolian:before {
   	content: "\ec89";
   }
   
   .icon-zitijiacu:before {
   	width: 24rpx;
   	height: 28rpx;
   	content: "\ec8a";
   }
   
   .icon-zitishanchuxian:before {
   	width: 32rpx;
   	height: 30rpx;
   	content: "\ec8b";
   }
   
   .icon-zitishangbiao:before {
   	content: "\ec8c";
   }
   
   .icon-zitibiaoti:before {
   	content: "\ec8d";
   }
   
   .icon-zitixiahuaxian:before {
   	width: 25rpx;
   	height: 30rpx;
   	content: "\ec8e";
   }
   
   .icon-zitixieti:before {
   	width: 26rpx;
   	height: 26rpx;
   	content: "\ec8f";
   }
   
   .icon-zitiyanse:before {
   	content: "\ec90";
   }
   
   .icon-zuoduiqi:before {
   	content: "\ec91";
   }
   
   .icon-zitiyulan:before {
   	content: "\ec92";
   }
   
   .icon-zitixiabiao:before {
   	content: "\ec93";
   }
   
   .icon-zuoyouduiqi:before {
   	content: "\ec94";
   }
   
   .icon-duigoux:before {
   	content: "\ec9e";
   }
   
   .icon-guanbi:before {
   	content: "\eca0";
   }
   
   .icon-shengyin_shiti:before {
	content: "\eca5";
   }
   
   .icon-Character-Spacing:before {
   	content: "\e964";
   }
   
   ```
   
   







## 富文本展示

### 1. 使用原生方法

1. 

```js
getImgSrc() {
    if(this.newsInfo.details){
        let imgList = [];
        // this.newsInfo.details 为后台取得的富文本内容
        this.newsInfo.details.replace(/<img [^>]*src=['"]([^'"]+)[^>]*>/g, (match, capture) => {
            imgList.push(capture);
        });

        return imgList;
    }
}
```

2. 处理富文本里的图片宽度自适应

```js
/**
		 * 处理富文本里的图片宽度自适应
		 * 1.去掉img标签里的style、width、height属性
		 * 2.img标签添加style属性：max-width:100%;height:auto
		 * 3.修改所有style里的width属性为max-width:100%
		 * 4.去掉<br/>标签
		 * @param html
		 * @returns {void|string|*}
		 */
formatRichText(html) { //控制小程序中图片大小
    if(html){
        let newContent = html.replace(/<img[^>]*>/gi, function(match, capture) {
            match = match.replace(/style="[^"]+"/gi, '').replace(/style='[^']+'/gi, '');
            match = match.replace(/width="[^"]+"/gi, '').replace(/width='[^']+'/gi, '');
            match = match.replace(/height="[^"]+"/gi, '').replace(/height='[^']+'/gi, '');
            return match;
        });
        newContent = newContent.replace(/style="[^"]+"/gi, function(match, capture) {
            match = match.replace(/width:[^;]+;/gi, 'max-width:100%;').replace(/width:[^;]+;/gi, 'max-width:100%;');
            return match;
        });
        newContent = newContent.replace(/<br[^>]*\/>/gi, '');
        newContent = newContent.replace(/\<img/gi,
                                        '<img style="max-width:100%;height:auto;display:inline-block;margin:10rpx auto;"');
        return newContent;
    }
}
```

### 3.parse插件 富文本插件（支持图片预览等功能）

1. 使用

   ```html
   <!--html 为富文本内容 -->
   <jyf-parser :html="html" ref="article"></jyf-parser>
   <!--也可以使用 uview 中 u-parser 插件 -->
   <u-parser :html="html" ref="article"></u-parser>
   ```

2. 详见 https://ext.dcloud.net.cn/plugin?id=805



## 浅拷贝和深拷贝

1. 浅拷贝---仅仅是指向被复制的内存地址，如果原地址发生改变，那么浅复制出来的对象也会相应的改变。

   ```js
   var a = [1, 2, 3, 4, 5];
   var b = a;
   a[0] = 2
   console.log(a);
   console.log(b);
   
   //因为b浅拷贝a, ab指向同一个内存地址(堆内存中存的值)
   //b会随着a的变化而变化
   //[2, 2, 3, 4, 5]
   //[2, 2, 3, 4, 5]
   ```

   

2. 深拷贝---在计算机中开辟一块**新的内存地址**用于存放复制的对象。

   ```js
   var a = [1, 2, 3, 4, 5];
   var b = [...a];
   a[0] = 2
   console.log(a); //[2, 2, 3, 4, 5]
   console.log(b);	//[1, 2, 3, 4, 5]
   ```

   

## 微信小程序分享朋友圈

1. 在 `onload`中写如下代码：解决了  需要分享给好友，才能分享朋友圈的问题

   ```js
   onLoad(option) {
   	wx.showShareMenu({
   		 withShareTicket: true,
   		menus: ['shareAppMessage', 'shareTimeline']
   	})
   
   },
   ```

   

2. ```js
   // onShareAppMessage 是 onShareTimeline 的前提  必须要有 onShareAppMessage  才能够分享朋友圈
   onShareAppMessage(res) {
       if (res.from === 'button') { // 来自页面内分享按钮
           console.log(res.target)
       }
       return {
           title: this.secondInfo.title,
           path: ""
       }
   },
   //用户点击右上角分享朋友圈  放在methods中 可以自定义标题 图片等
   onShareTimeline: function() {
       return {
           title: '',
           query: {
               key: value
           },
           imageUrl: ''
       }
   },
   ```




## https 和 http

1. https://blog.csdn.net/zhy0509/article/details/81413402



## 微信小程序中 swiper-view 下拉刷新

1. ```html
   <!-- #ifdef MP-WEIXIN -->
   <swiper-item class="swiper-item">
       <scroll-view scroll-y style="height:100vh; width: 100%;" @scrolltolower="onreachBottom" refresher-enabled="true"
                    :refresher-triggered="triggered" :refresher-threshold="100" @refresherrefresh="onRefresh" @refresherrestore="onRestore">
           <productindex :current="current" v-if="productShowIndex === 0"></productindex>
       </scroll-view>
   </swiper-item>
   <!-- #endif -->
   ```
   
2. ```js
   data() {
       return {
           triggered: true, // 刷新状态
       }
   },
       methods: {
           // scroll-view到底部加载更多
           onreachBottom() {
               this.CHANGE_ENTER_BOTTOM(Math.random())
           },
   
           //  scroll-view 下拉刷新
           onRefresh() {
               if (this._freshing) return;
               this._freshing = true;
               setTimeout(() => {
                   this.CHANGE_PULL_REFRESH(Math.random())
                   this.triggered = false;
                   this._freshing = false;
               }, 1500)
           },
           onRestore() {
               this.triggered = null; // 需要重置
           },
   
       },
           onLoad() {
               this.getUsedProductCategory()
               this.getHotProvinces()
   
               this._freshing = false;
               setTimeout(() => {
                   this.triggered = true;
               }, 1000)
           },
   
   ```




## uni.previewImage()  ios 端失效 （解决）

1. 在图片链接加上 `Content-Type：image/jpg`

   ```js
   function swiper_preview(index, imgList) {
   	var b = [...imgList]
   
   	
       // 链接拼接 Content-Type=image/jpg   设置文件类型
   	b.forEach((item, index, arr) => {
   		arr[index] = `${BASE_IMG_URL}${item}?Content-Type=image/jpg`
   	})
   
   	// console.log(b);
   
   	uni.previewImage({
   		current: index,
   		urls: b,
   	})
   	// return
   }
   ```




## 微信小程序的跳转另一个微信小程序

1. ```js
   uni.navigateToMiniProgram({
       appId: "wxb241004ae2cef9e6", // 要跳转的微信小程序appid
       path: "/pages/index/index", // 要跳转的微信小程序的页面路径
       success: res => {
           // 打开成功
           console.log("打开成功", res); // 一定要有success 回调
       },
       fail: err => {
           console.log(err);
       }
   });
   ```




## 微信注册 JS-SDK

### 1.引入 js 文件

```html
<!-- 引入外部js 写你想引入的 script  -->
<script type="text/javascript" src="https://res.wx.qq.com/open/js/jweixin-1.6.0.js"></script>
<script type="text/javascript" src="https://res2.wx.qq.com/open/js/jweixin-1.6.0.js "></script>
```

### 2.weChat.js

```js
//由于企业微信jssdk不是模块化js文件,所以采用自定义index.html模板,并用<script>标签引入
// const jWeixin = require('../static/jweixin-1.2.0.js');
import {
    BASE_IMG_URL,
    BASE_URL
} from "../ajax/ajax.js"


function initJssdk(shareCover,shareTitle,shareDetail) {
    return new Promise((reslove, reject) => {

        uni.request({
            url: `${BASE_URL}/api/getJsJDKconfig`, // 后端接口地址 得到   jWeixin.config  所需要的参数
            method: "POST",
            header: {
                "Content-Type": "application/x-www-form-urlencoded",
            },
            data: {
                url: encodeURIComponent(window.location.href),
            },
            dataType: "json",
            success: (res) => {
                // console.log(res);
				// 默认标题 描述 logo
                let shareDefaultTitle = '针织e家'
                let shareDefaultInfo = '针织行业生态圈'

                let shareDefaultUrl =
                    // 'https://profile.csdnimg.cn/9/E/B/1_qq_24147051'
                    `${BASE_IMG_URL}/ftpnginx/companyundefined/d869f98a68b332f602f8d2508973df36.png`
                // let info = JSON.parse(res.module.config);
                let info = res.data.data
                jWeixin.config({
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
                    openTagList: ['wx-open-launch-weapp']， // 微信开放标签
                })
                jWeixin.ready(function() {
                    // jWeixin.checkJsApi({
                    // 	jsApiList: ['chooseImage', "onMenuShareTimeline", "updateTimelineShareData", "updateAppMessageShareData"], // 需要检测的JS接口列表，所有JS接口列表见附录2,
                    // 	success: function(res) {
                    // 		console.log(res);
                    // 		// 以键值对的形式返回，可用的api值true，不可用为false
                    // 		// 如：{"checkResult":{"chooseImage":true},"errMsg":"checkJsApi:ok"}
                    // 	}
                    // });

                    //自定义“分享给朋友”及“分享到QQ”按钮的分享内容（1.4.0）
                    jWeixin.updateAppMessageShareData({
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
                    jWeixin.updateTimelineShareData({
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
                    //分享到微博
                    jWeixin.onMenuShareWeibo({
                        title: shareTitle || shareDefaultTitle,
                        desc: shareDetail || shareDefaultInfo,
                        link: window.location.href,
                        imgUrl: shareCover || shareDefaultUrl,
                        success: function() {
                            // 用户确认分享后执行的回调函数
                        },
                        cancel: function() {
                            // 用户取消分享后执行的回调函数
                        }
                    })

                    console.log('config初始化成功');
                    reslove(true)
                })


                jWeixin.error((res) => {
                    console.log('config初始化失败', res);
                    reslove(false)
                });
            },
            fail: (err) => {
                reject(err)
                setTimeout(function() {
                    uni.hideLoading();
                }, 1000);
            },
            complete: () => {

            }
        })

    })

}

module.exports = {
    initJssdk
}

```

### 3.main.js  引入

```js
import {
	initJssdk
} from "./utils/common/wechat.js"

// #ifdef H5
initJssdk().then(next => {
	if (next) console.log("initJssdk成功")
})
// #endif

Vue.prototype.$initJssdk = initJssdk
```

### 4.某个页面单独使用 单独注入

```js
// #ifdef H5
this.$initJssdk().then(next => {
    if (next) console.log("initJssdk成功")
})
// #endif
```

### 5. H5 端自定义分享微信好友 QQ好友 朋友圈等

```js
// this.getEcard() 与后端交互 获取数据 $baseImgUrl网络路径域名 "https://m.kintting.cn"
this.getEcard().then(() => {
    let shareCover = `${this.$baseImgUrl}${this.ecard_avatar}`
    let shareTitle = `我是${this.params_obj.nickname}，这是我的新名片，敬请惠存。`
    this.$initJssdk(shareCover, shareTitle).then(next => {
        if (next) console.log("initJssdk成功")
    })
})
```





## 微信内置浏览器 唤醒小程序

1. ```html
   <!-- username: 小程序原始id  path:要打开的页面路径（必须带html后缀）  -->
   <wx-open-launch-weapp id="launch-btn" username="gh_094e9f2515d2" path="pages/index/index.html">
   				<script type="text/wxtag-template">
   					<style>
   						.miniProCon{
   							display: flex;
   							justify-content: space-between;
   							align-items: center;
   							margin: 0 60px;
   						}
   						.btn {
   							background-color: #007AFF;
   							color: #fff;
   							font-size: 30px;
   							padding: 12px;
   							line-height: 80px;
   						}
   					</style>
   					<view class="miniProCon">
   						<view class="miniProLogo">
   							<image src="/static/img/e_logo.png" mode=""></image>
   						</view>
   						<view class="btn">打开小程序</view>
   					</view>
   				</script>
   </wx-open-launch-weapp>
   ```

2. 注意事项

   - 单位使用px
   - 若要引用图片，须得是网络地址
   - 按钮放置位置，可以外部嵌套 `div` 等元素进行定位，以控制按钮显示位置
   - 在微信开发者工具中没有显示是因为微信开发者工具还是无法支持微信开放标签的调试功能，**只能在手机上调试并且是在加了微信安全域名的服务器环境下调试(￣︶￣)**






## 拉取微信快捷登录

### 1.微信内置浏览器拉取微信登录

```html
<button v-if="wx_open == true " class="wx_login_btn" open-type="getUserInfo" @click="wx_login">
    微信一键登录
</button>
```

```js
// 微信小程序 H5 微信登录
wx_login() {
    // #ifdef  H5
    var locationLink = window.location.origin + "/pages/login/login"
	
    // redirect_uri 重定向的页面 appid ：微信小程序id
    
    location.href = "https://open.weixin.qq.com/connect/oauth2/authorize?appid=" + this.appid + "&redirect_uri=" +
        encodeURIComponent(locationLink) + "&response_type=code&scope=snsapi_userinfo&state=STATE#wechat_redirect"
    // #endif

},

    onLoad(){
    
        // #ifdef  H5
        // 判断是否是在微信内置浏览器打开的
        var ua = navigator.userAgent.toLowerCase();
        if (ua.match(/MicroMessenger/i) == "micromessenger") {
            console.log('微信浏览器')
            this.wx_open = true
        } else {
            console.log('其他');
            this.wx_open = false
        }
        // #endif
    }
```



### 2. 微信小程序拉取微信登录并获取用户信息

```html
<!--  open-type="getUserInfo" 是必须的 用户手动触发 -->
<button class="wx_login_btn" open-type="getUserInfo" @click="wx_login">
	微信一键登录
</button>
```

```js
wx_login() {
    // #ifdef MP-WEIXIN
	
    
    // 调用微信登录api 获取code
    // code 是小程序专有，用户登录凭证。开发者需要在开发者服务器后台，使用 code 换取 openid 和 session_key 等信息
    uni.login({
        provider: 'weixin',
        success: (loginRes) => {
            var code = loginRes.code;
            // 获取用户信息
            uni.getUserInfo({
                provider: 'weixin',
                success: (res) => {
                    // console.log(res);
                    /* 后端接口 encryptedData	String	包括敏感数据在内的完整用户信息的加密数据，详细见加密数据解密算法。
                               iv	String	加密算法的初始向量，详细见加密数据解密算法。
                    */
                    this.wechatMinLogin(code, res.encryptedData, res.iv)
                },
                fail: (err) => {
                    // console.log(err)
                }
            });
        }
    });

    // #endif

},
```

### 3. 微信登录获取手机号

```html
<button type="default" open-type="getPhoneNumber" @getphonenumber="decryptPhoneNumber">获取手机号</button>
```

```js
// 获取手机号
decryptPhoneNumber(e){
    uni.login({
        provider: 'weixin',
        success: (loginRes) => {
            // console.log(loginRes);
            var code = loginRes.code;  // 微信用户登录凭证
            // console.log(e.detail); // e.detail 获取手机号 返回的数据 包含 encryptedData iv 
            this.wechatMinLogin(code, e.detail.encryptedData, e.detail.iv)
        }
    });
}
```





## 微信小程序的分包

### 1. 创建一个目录

```json
// 此时的目录结构 如下  packageA 为我们想要创建的 一个分包
├── app.js
├── app.json
├── app.wxss
├── packageA
│   └── pages
│       ├── cat
│       └── dog
│	└── static
│       └── img
├── pages
│   ├── index
│   └── logs
└── utils
```

### 2. 在 pages.json 中 通过 `subpackages` 字段声明项目分包结构

```json
{
    "pages":[
        "pages/index",
        "pages/logs"
    ],
    "subpackages": [
        {
            "root": "packageA/", // 分包根目录
            "pages": [ // 分包页面路径
                "pages/cat",
                "pages/dog"
            ]
        }
    ]
}
```

1. 打包原则
   - 声明 `subpackages` 后，将按 `subpackages` 配置路径进行打包，`subpackages` 配置路径外的目录将被打包到 app（主包） 中
   - app（主包）也可以有自己的 pages（即最外层的 pages 字段）
   - `subpackage` 的根目录不能是另外一个 `subpackage` 内的子目录
   - `tabBar` 页面必须在 app（主包）内
2. 引用原则
   - `packageA` 无法 require `packageB` JS 文件，但可以 require `app`、自己 package 内的 JS 文件
   - `packageA` 无法 import `packageB` 的 template，但可以 require `app`、自己 package 内的 template
   - `packageA` 无法使用 `packageB` 的资源，但可以使用 `app`、自己 package 内的资源



### 3.分包预加载

1. 分包预加载：即进入小程序中的某个页面时，自动预下载所需要的分包，提升进入分钟页面时的启动速度。（暂不支持API调用完成，只支持配置方式使用）

2. 配置

   ```json
   {
       "pages": ["pages/index"],
       "subpackages": [
           {
               "root": "important",
               "pages": ["index"],
           },
           {
               "root": "sub1",
               "pages": ["index"],
           },
           {
               "name": "hello",
               "root": "path/to",
               "pages": ["index"]
           },
           {
               "root": "sub3",
               "pages": ["index"]
           },
           {
               "root": "indep",
               "pages": ["index"],
               "independent": true
           }
       ],
       "preloadRule": {
           "pages/index": {
               "network": "all",
               "packages": ["important"]
           },
           "sub1/index": {
               "packages": ["hello", "sub3"]
           },
           "sub3/index": {
               "packages": ["path/to"]
           },
           "indep/index": {
               "packages": ["__APP__"]
           }
       }
   }
   ```

3. `preloadRule` 中，`key` 是页面路径，`value` 是进入此页面的预下载配置

### 4. 分包优化

1. 在对应平台的配置下添加`"optimization":{"subPackages":true}`开启分包优化

2. 目前只支持`mp-weixin`、`mp-qq`、`mp-baidu`的分包优化

3. 分包优化具体逻辑：

   - 静态文件：分包下支持 static 等静态资源拷贝，即分包目录内放置的静态资源不会被打包到主包中，也不可在主包中使用
   - js文件：当某个 js 仅被一个分包引用时，该 js 会被打包到该分包内，否则仍打到主包（即被主包引用，或被超过 1 个分包引用）
   - 自定义组件：若某个自定义组件仅被一个分包引用时，且未放入到分包内，编译时会输出提示信息

4. ```json
   /* manifest.json 文件下配置 */
   "mp-weixin": {
       "appid": "wxbf9d6324f0dbc238",
       "setting": {
           "urlCheck": true,
           "es6": true,
           "postcss": true,
           "minified": true
       },
       "optimization":{"subPackages":true} // 开启分包优化 就可以存放js文件 自定义组件 静态资源
   },
   ```




## 防抖节流

### 1. 概念

1. 防抖：任务频繁触发下，只有任务触发的时间间隔超过指定的时间间隔的时候，任务才执行。

   ```js
   // 原生JS的实现
   window.onload = function() {
       // 1、获取这个按钮，并绑定事件
       var myDebounce = document.getElementById("debounce");
       myDebounce.addEventListener("click", debounce(sayDebounce));
   }
   
   // 2、防抖功能函数，接受传参
   function debounce(fn) {
       // 4、创建一个标记用来存放定时器的返回值
       let timeout = null;
       return function() {
           // 5、每次当用户点击/输入的时候，把前一个定时器清除
           clearTimeout(timeout);
           // 6、然后创建一个新的 setTimeout，
           // 这样就能保证点击按钮后的 interval 间隔内
           // 如果用户还点击了的话，就不会执行 fn 函数
           timeout = setTimeout(() => {
               fn.apply(this, arguments);
           }, 1000);
       };
   }
   
   // 3、需要进行防抖的事件处理
   function sayDebounce() {
       // ... 有些需要防抖的工作，在这里执行
       console.log("防抖成功！");
   }
   
   ```

2. 节流：在指定的时间间隔之内任务只会执行一次

   ```js
   window.onload = function() {
       // 1、获取按钮，绑定点击事件
       var myThrottle = document.getElementById("throttle");
       myThrottle.addEventListener("click", throttle(sayThrottle));
   }
   
   // 2、节流函数体
   function throttle(fn) {
       // 4、通过闭包保存一个标记
       let canRun = true;
       return function() {
           // 5、在函数开头判断标志是否为 true，不为 true 则中断函数
           if(!canRun) {
               return;
           }
           // 6、将 canRun 设置为 false，防止执行之前再被执行
           canRun = false;
           // 7、定时器
           setTimeout( () => {
               fn.apply(this, arguments);
               // 8、执行完事件（比如调用完接口）之后，重新将这个标志设置为 true
               canRun = true;
           }, 1000);
       };
   }
   
   // 3、需要节流的事件
   function sayThrottle() {
       console.log("节流成功！");
   }
   ```

### 2. uview 插件使用 节流防抖

```js
// 节流
// toNext 是一个方法 一般是api请求
this.$u.throttle(this.toNext, 500)

// 自定义书写
this.$u.throttle(()=>{
    // 函数方法
}, 500)



// 防抖
this.$u.debounce(this.toNext, 500)
```







































