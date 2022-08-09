# 源学习

## 1.变化侦测

1. **UI = render(state)**

   - 数据驱动视图： `state`  是状态输入， `ui` 是 页面输出
   - `state` 和   `ui`  其实都是我们自己定义的，而渲染则是 vue 帮我们做的。所以 Vue 扮演的角色就很清晰了，就是 `render()` ，他会帮我们监测 `state` 发生变化，经过一系列加工，最终将变化反应在 `UI` 上。

2. 上文讲到的监测 state 发生变化，就是变化侦测。**变化侦测就是追踪状态，也可以说是数据的的变化。**

3. ### 使Object数据变得“可观测”

   - ```javascript
     // 源码位置：src/core/observer/index.js
     
     /**
      * Observer类会通过递归的方式把一个对象的所有属性都转化成可观测对象
      */
       constructor (value) {
         this.value = value
         // 给value新增一个__ob__属性，值为该value的Observer实例
         // 相当于为value打上标记，表示它已经被转化成响应式了，避免重复操作
         def(value,'__ob__',this)
         if (Array.isArray(value)) {
           // 当value为数组时的逻辑
           // ...
         } else {
           this.walk(value)
         }
       }
     
       walk (obj: Object) {
         const keys = Object.keys(obj) // 枚举 对象的所有属性
         for (let i = 0; i < keys.length; i++) {
           defineReactive(obj, keys[i])
         }
       }
     }
     /**
      * 使一个对象转化成可观测对象
      * @param { Object } obj 对象
      * @param { String } key 对象的key
      * @param { Any } val 对象的某个key的值
      */
     function defineReactive (obj,key,val) {
       // 如果只传了obj和key，那么val = obj[key]
       if (arguments.length === 2) {
         val = obj[key]
       }
       if(typeof val === 'object'){
           new Observer(val)
       }
       Object.defineProperty(obj, key, {
         enumerable: true, // 可枚举的
         configurable: true, // 属性描述符能否被配置(改变) 包括删除
         get(){
           dep.depend()    // 在getter中收集依赖
           console.log(`${key}属性被读取了`);
           return val;
         },
         set(newVal){
           if(val === newVal){
               return
           }
           console.log(`${key}属性被修改了`);
           val = newVal;
           dep.notify()   // 在setter中通知依赖更新
         }
       })
     }
     ```
     
- 定义了一个 `observer` 类，将正常的`object`转化成可以监测的`object`。
  
  并且给 `value`添加一个 `__ob__`属性，值就是 `observer` 的实体。
  
  然后判断数据类型，只有是 object 对象才会执行 `walk` ，将该对象的所有属性转换成 `getter/setter` 的形式来监测变化。最后在 `defineReactive`中判断传入的 `val` 如果依然是一个 `object` ，则递归使用 `new Observer(val)` 来监测对象属性的子属性。至此，我们就可以把一个对象 `obj` 中的所有属性(包括子属性)全都转换成 `getter/setter` 的形式来侦测变化。
     即，**我们只要将一个普通的 `object` 对象传入 `oberver` 中，这个对象就会变成可监测的、响应式的`object`。** 
  
  ```js
     let car = new Observer({
       'brand':'BMW',
       'price':3000
     })
     // 最后举个例子，这样子写 car 的两个属性就都可以被侦测了
  ```
  
4. ### 依赖收集

   - 上面我们已经让一个对象变成可以监测的 `object`了。也就是说，我们已经可以知道 状态什么时候发生了变化，然后谁要对应的更新视图还没有目标。所以引进了 **依赖收集** ，视图中谁使用了这个状态就更新谁。
     优雅的说法就是：谁依赖于这个数据，则更新谁。
     所以我们现在要做的就是收集依赖，我们给每个状态(数据)都建立了一个依赖数组，谁依赖这个状态，就把他放进这个 依赖数组。之后当 这个状态发生改变时，我们就通知它对应的依赖数据，告诉他们：“快更新视图吧”。

   - 何时收集依赖？又何时通知依赖更新？

     - **在getter中收集依赖，在setter中通知依赖更新**。

   - 依赖管理器（Dep)，为每一个数据都建立一个依赖管理器。

     ```js
     // 源码位置：src/core/observer/dep.js
     export default class Dep {
       constructor () {
         this.subs = []  // subs 装的是依赖，依赖：watcher实例
       }
     
       addSub (sub: Watcher) {
         this.subs.push(sub)
       }
       // 删除一个依赖
       removeSub (sub: Watcher) {
         remove(this.subs, sub)
       }
       // 添加一个依赖
       depend () {
           // window.target 指向的是一个watcher
         if (window.target) {
           this.addSub(window.target)
         }
       }
       // 通知所有依赖更新
       notify () {
         const subs = this.subs.slice()
         for (let i = 0, l = subs.length; i < l; i++) {
           subs[i].update()
             // 会调用wacther.update()方法
         }
       }
     }
     
     /**
      * Remove an item from an array
      */
     export function remove (arr, item) {
       if (arr.length) {
         const index = arr.indexOf(item)
         if (index > -1) {
           return arr.splice(index, 1)
         }
       }
}
     ```
     
   - 有了依赖管理器(Dep)之后的 侦测代码
   
     ```js
     function defineReactive (obj,key,val) {
       if (arguments.length === 2) {
         val = obj[key]
       }
       if(typeof val === 'object'){
         new Observer(val)
       }
       const dep = new Dep()  //实例化一个依赖管理器，生成一个依赖管理数组dep
       Object.defineProperty(obj, key, {
         enumerable: true,
         configurable: true,
         get(){
           dep.depend()    // 在getter中收集依赖
           return val;
         },
         set(newVal){
           if(val === newVal){
               return
           }
           val = newVal;
           dep.notify()   // 在setter中通知依赖更新
         }
       })
     }
     ```
   
5. ## 依赖到底是谁

   - 上面一直说收集依赖，那么依赖到底是谁呢？

     于是就有了一个叫做`Watcher`的类，而`Watcher`类的实例就是我们上面所说的那个"谁"。

   - `Watcher`类的具体实现如下：

     ```js
     export default class Watcher {
       constructor (vm,expOrFn,cb) {
           // expOrFn: 被观察的表达式
         this.vm = vm;
         this.cb = cb;
         this.getter = parsePath(expOrFn)
         this.value = this.get()
       }
       get (){
         window.target = this;
         const vm = this.vm
         let value = this.getter.call(vm, vm)
         window.target = undefined;
         return value
       }
       update () {
         const oldValue = this.value
         this.value = this.get()
         this.cb.call(this.vm, this.value, oldValue)
       }
     }
     
     /**
      * Parse simple path.
      * 把一个形如'data.a.b.c'的字符串路径所表示的值，从真实的data对象中取出来
      * 例如：
      * data = {a:{b:{c:2}}}
      * parsePath('a.b.c')(data)  // 2
      */
     const bailRE = /[^\w.$]/
     export function parsePath (path) {
       if (bailRE.test(path)) {
         return
       }
       const segments = path.split('.')
       return function (obj) {
         for (let i = 0; i < segments.length; i++) {
           if (!obj) return
           obj = obj[segments[i]]
         }
         return obj
       }
     }
     ```

   - 当实例化`Watcher`类时，会先执行其构造函数；

   - 在构造函数中调用了`this.get()`实例方法；

   - 在`get()`方法中，首先通过`window.target = this`把实例自身赋给了全局的一个唯一对象`window.target`上，然后通过`let value = this.getter.call(vm, vm)`获取一下被依赖的数据，获取被依赖数据的目的是触发该数据上面的`getter`，上文我们说过，在`getter`里会调用`dep.depend()`收集依赖，而在`dep.depend()`中取到挂载`window.target`上的值并将其存入依赖数组中(**就是将watcher实例存在依赖数组中subs**)，在`get()`方法最后将`window.target`释放掉。

   - 而当数据变化时，会触发数据的`setter`，在`setter`中调用了`dep.notify()`方法，在`dep.notify()`方法中，遍历所有依赖(即watcher实例)，执行依赖的`update()`方法，也就是`Watcher`类中的`update()`实例方法，在`update()`方法中调用数据变化的更新回调函数，从而更新视图。
   
6. 总结：

   - new MVVM()，解析指令 ComPile 时，对数据进行读取，同时触发了数据的 `getter` 方法。在 `getter` 中会从全局唯一的位置读取当前正在读取数据的 `watcher`，并且把这个 `watcher` 添加到 `Dep` 依赖收集器中。
   - 数据变化时，触发 Observer 中的 `setter`，通知 `Dep` 发生变化， `dep`会向每一个 `watcher` 发送通知，`watcher`则会执行 updateed 方法修改数据，从而更新视图。





# 算法

## 二分法

### 1.前提：基于数据先排好顺序

### 2. 使用：

- 递归

  ```js
  // methods
  binarySearch(arr, low, high, key) {
      if (low > high) {
          return -1;
      }
      var mid = Math.floor((low + high) / 2);
      if (key == arr[mid]) {
          return mid;
      } else if (key < arr[mid]) {
          high = mid - 1;
          return this.binarySearch(arr, low, high, key);
      } else {
          low = mid + 1;
          return this.binarySearch(arr, low, high, key);
      }
  }
  
  var arr = [0, 5, 6, 7, 8, 9, 10, 15, 20, 30]
  
  console.log(this.binarySearch(arr,0,8,5));
  ```

- 非递归

  ```js
  function binarySearch(arr,key){
      var low=0; //数组最小索引值
      var high=arr.length-1; //数组最大索引值
      while(low<=high){
          var mid=Math.floor((low+high)/2);
          if(key==arr[mid]){
              return mid;
          }else if(key>arr[mid]){
              low=mid+1;
          }else{
              high=mid-1;
          }
      }
      return -1; //low>high的情况，这种情况下key的值大于arr中最大的元素值或者key的值小于arr中最小的元素值
  }
  
  var arr = [0, 5, 6, 7, 8, 9, 10, 15, 20, 30]
  
  console.log(this.binarySearch(arr,0,8,5));
  ```







# **Vue3.0**

## 常用的Composition API

### 1.setup

- vue3 的一个新的配置

- 原来vue2 中得 data、methods... 等等 都封装在 setup中

- 假如 vue2 和 vue3 同时定义了一个相同 名字得属性，以vue3为准

- setup 有两中返回格式，1：返回一个对象，2：返回一个渲染对象

- 执行时机：如果有`beforeCreate` 配置项，在 `beforeCreate` 之前执行一次，此时 `this` 为 `undefined`

- setup的参数
  - props：值为对象，包含了：组件外部的传参，且在组件内部声明了的属性
  - context：上下文对象**(普通的js对象 可以解构)**
    - attrs：组件外部传递的值，没有在组件内部声明（props没有配置），相当于vue2中 `this.$attrs`
    - emit：分发自定义事件函数，相当于 `this.$emit`
    - slot：收到的插槽内容，相当于 `this.$slot`
    - expose：暴露公共 property (函数) 
  
- ```js
  import { toRefs,toRef  } from 'vue'
  
  export default {
     // 接受外部组件 传递的参数，也可以写成数组格式 props:['xxx','xxx']
    props: {
      title: String
    },
    setup(props,context) {
      // props 是响应式的，无法使用ES6的解构，解构会消除props的响应式
      // 如果需要解构，可以在 setup 函数中使用 toRefs函数来完成此操作
      const { title } = toRefs(props)
    	console.log(title.value)
        
      // 如果 title 是可选的 prop，则传入的 props 中可能没有 title 。
      // 在这种情况下，toRefs 将不会为 title 创建一个 ref 。你需要使用 toRef 替代它  
      const title = toRef(props, 'title')
    	console.log(title.value)
    }
  }
  ```
```
  
- ```js
  // context 运用
  
  
  // 子组件.vue
  export default {
    props: {
      title: String
    },
    emits:[], // 父组件绑定的事件 都需要在这里填写 不然会有 警告
    setup(props,context) {
      context.emit()  // 类比 vue2中 this.$emit()
    }
  }
```

  



### 2.ref函数

- 语法： const 代理对象 = ref(源对象)

- 源对象 ：可以是基本数据类型，也可以是复杂数据类型（对象，数组）

- 作用：把普通得数据 转化为 响应式数据

  - 底层是 通过 `object.defineProperty(obj,key)`

- 代理对象: 

  - 基本数据类型：返回 `refImpl` , 使用时需要 `代理对象.value`

    - ```js
      const name = ref('张三') 
      function changeName() {
      	name.value = '李四' // name 经过ref后 变为 refimpl 对象
      }
      ```

  - 对象或数组：内部求助于 `reactive` 函数，`代理对象.value` 是一个 `proxy` 对象

    - ```js
      const job = ref({
      	type: '类型'
      }) // 响应式  对象类型
      function changeName() {
      	job.value.type = '新类型'
      }
      ```





### 3. reactive 函数

- 语法： const 代理对象 = reactive(源对象)

- 源对象 ：只接收 对象类型的源对象

- 作用：把普通的对象 转化为 响应式对象数据

  - reactive 转化的响应式数据是 “深层次的”
  - 底层是 `proxy` 代理 实现响应式(数据劫持)，再通过 `reflect` 操作源对象内部的数据

- 使用：

  - ```js
    const state = reactive({
        myname: 'kerwin',
        datalist: [
            {
                id: 1,
                count: count
            },
            {
                id: 2,
                count: count
            }	
        ]
    })
    
    const changeList = () => {
        state.datalist[1] = 'hhhh'
        // vue3 使用proxy 数据劫持  可以直接使用数组下标进行响应式更新数据
    }
    ```





### shallowRef 和 shallowReactive

- shallowReactive：只处理对象最外层的响应式(浅层响应式)
- shallowRef：只处理基本数据类型的响应式，不进行对象的响应式处理。
- 使用场景：
  - 如果有一个对象数据，结构深，但是变化时只是外层属性变化 ===》 shallowReactive
  - 如果有一个对象数据，后续功能不会修改对象中的属性，而是生成新的对象，整个替换 ===> shallowRef




### 4.computed函数

- 语法： const 代理对象 = computed(源对象)

- 使用：与 vue2 一样

  - ```js
    const fullInfo = computed({
        get() {
            return state.myname + '-' + state.sex
        },
        set(value) {
            const valueArr = value.split('-')
            state.myname = valueArr[0]
            state.sex = valueArr[1]
        }
    })
    ```



### 5.watch 函数

- 语法 watch(监听对象，handle，监视配置)

- 坑：

  - 如果监听对象是 **ref函数处理的 基本数据类型**，监听对象不用加 `.value`，如果是 **ref函数处理的对象 **需要加 `.value`
  - 监听的是 reactive函数处理的响应式数据：oldValue 无法争取获取，且强制开启深度监听,(配置deep 无效)
  - 监听的 是reactive函数处理的响应式数据中的某个属性(该属性还是对象)，则需要配置 `deep：true`

- 使用：

  - ```js
    const obj = ref({
        value: 'abc'
    })
    const name = ref('张三') 
    // 情况一 
    watch(
        name,
        (newv, oldv) => {
            console.log(newv, oldv)
        },
        { immediate: true }
    )
    // 情况二 (ref对象)
    watch(
        obj.value,
        (newv, oldv) => {
            console.log(newv, oldv)
        },
        { immediate: true }
    )
    
    const state = reactive({
        myname: 'kerwin',
        sex: 'boy',
        datalist: [
            {
                id: 1,
                count: count
            },
            {
                id: 2,
                count: count
            }
        ]
    })
    // 情况三 (reactive对象)
    // 默认开启深度监听，无法正确拿到oldValue
    watch(
        state,
        (newV, oldV) => {
            console.log('state改变了', newV, oldV)
        },
        { immediate: false }
    )
    // 情况四 (reactive属性)
    // 监听 reactive 对象的某个属性
    watch(
        () => state.myname,
        (newV, oldV) => {
            console.log('state改变了', newV, oldV)
        },
        { immediate: false }
    )
    
    // 情况五 (reactive 的多个属性)
    // 监听 reactive 对象的多个属性
    watch(
        [() => state.myname, () => state.sex],
        (newV, oldV) => {
            console.log('state改变了', newV, oldV)
        },
        { immediate: false }
    )
    
    // 特殊情况
    // 监听 reactive 对象的某个属性(属性是对象) 需要开启深度监听
    watch(
        () => state.datalist,
        (newV, oldV) => {
            console.log('state改变了', newV, oldV)
        },
        { deep: true }
    )
    
    
    ```

- 

### watchEffect函数

- 语法：watcheffect(()=>{/////////////})

- 目标：watcheffect 不具体指明监测对象，而是在回调中，使用到了哪些对象，就监测哪些对象。

- 使用：

  - ```js
    // 不指明具体监视哪个属性，回调中使用了哪个属性，就监测哪个属性
    watchEffect(() => {
        const x2 = state.myname
        console.log('watchEffect监测到变化了')
    })
    ```

- 与 computed 的联系：

  - computed：依赖数据发生改变，则对应发生改变，需要返回一个结果。注重结果。
  - watcheffect：监测数据发生改变，进行回调，不需要返回。更注重过程。



### 6.toRef与toRefs

- 语法：`const name = toRef(state,'name')`

- 作用：创建一个 ref 对象，其 value 值指向另一个对象中的某个属性

- 应用：要将响应式对象中的某一个属性单独提供给外部使用

- 扩展：`toRefs` 可以批量创建多个 ref 对象，语法 `toRefs(state)`

- 使用：

  - ```vue
    <template>
        <h2>姓名:{{ myname }}</h2>
        <h2>性别:{{ sex }}</h2>
    </template>
    
    
    <script>
        const state = reactive({
            myname: 'kerwin',
            sex: 'boy',
        })
    
        return {
           myname:toRef(state,'myname'),
           sex:toRef(state,'sex'),
        }
    </script>
    
    
    ```
<template>
      <div class="img-verify">
        {{height}}
          {{width}}
      </div>
    </template>

    <script>
    import { reactive, onMounted, ref, toRefs } from 'vue'
    export default {
      setup() {
      	const state = reactive({
          pool: 'ABCDEFGHIJKLMNOPQRSTUVWXYZ1234567890', // 字符串
          width: 120,
          height: 40,
          imgCode: ''
        })
        
         return {
          ...toRefs(state),
        }
      }
     }
    </script>
    ```


​    

 

## Proxy实现数据劫持

- ```js
  observe(data) {
      const that = this;
      let handler = {
          get(target, property) {
              // return target[property];
  
              // proxy 也不支持嵌套，因此需要逐层遍历。Proxy 的写法是在 get 里面递归调用 Proxy 并返回
              if (typeof target[property] === 'object' && target[property] !== null) {
                  return new Proxy(target[property], handler)
              }
  
              return Reflect.get(target, property)
          },
          set(target, key, value) {
              // Reflect 反射
              let res = Reflect.set(target, key, value);
              // subscribe 是一个主题/事件通道，  介于订阅者对象和发布者对象之间 允许代码定义应用程序的特定事件 目的是避免订阅者和发布者之间产生依赖
              that.subscribe[key].map(item => {
                  item.update();
              }); // 通知依赖进行更新
              return res;
          }
      }
      // data：拦截的目标对象， handler：定制拦截行为
      this.$data = new Proxy(data, handler);
      // $data 就是 vue中的 data
  }
  
  这段代码里把代理器返回的对象代理到this.$data，即this.$data是代理后的对象，外部每次对this.$data进行操作时，实际上执行的是这段代码里handler对象上的方法。
  
  
  
  
  function reactive(obj) {
      if (typeof obj !== 'object' && obj != null) {
          return obj
      }
      // Proxy相当于在对象外层加拦截
      const observed = new Proxy(obj, {
          get(target, key, receiver) {
              const res = Reflect.get(target, key, receiver)
              console.log(`获取${key}:${res}`)
              return res
          },
          set(target, key, value, receiver) {
              const res = Reflect.set(target, key, value, receiver)
              console.log(`设置${key}:${value}`)
              return res
          },
          deleteProperty(target, key) {
              const res = Reflect.deleteProperty(target, key)
              console.log(`删除${key}:${res}`)
              return res
          }
      })
      return observed
  }
  ```
  
- 为什么要替换 Object.defineProperty

  - 在Vue中，Object.defineProperty无法监控到数组下标的变化，导致直接通过数组的下标给数组设置值，不能实时响应。

  - Object.defineProperty只能劫持对象的属性,因此我们需要对每个对象的每个属性进行遍历，甚至深度遍历。

  - ```html
    Proxy是 ES6 中新增的一个特性，翻译过来意思是"代理"，用在这里表示由它来“代理”某些操作。 Proxy 让我们能够以简洁易懂的方式控制外部对对象的访问。其功能非常类似于设计模式中的代理模式。
    
    Proxy 可以理解成，在目标对象之前架设一层“拦截”，外界对该对象的访问，都必须先通过这层拦截，因此提供了一种机制，可以对外界的访问进行过滤和改写。
    
    使用 Proxy 的核心优点是可以交由它来处理一些非核心逻辑（如：读取或设置对象的某些属性前记录日志；设置对象的某些属性值前，需要验证；某些属性的访问控制等）。 从而可以让对象只需关注于核心逻辑，达到关注点分离，降低对象复杂度等目的。
    ```



## Reflect

与 `Proxy` 对象一样，也是为了操作对象而生的新API。 不可以是  new Reflect () 对象，并且 参数的target 必须是一个对象。



## Vue应用实例

1. 每个 Vue 应用都是通过用 `createApp` 函数创建一个新的**应用实例**开始的：

   ```js
   // 3.0
   const app = Vue.createApp({ /* 选项 */ })
   const vm = app.mount('#app')
   // 2.x
   const app = new Vue({
   	...App,
   	store
   })
   app.$mount()
   ```

   



## 生命周期钩子函数

1. ```js
   import { onBeforeMount, onMounted, onBeforeUpdate, onUpdated, onBeforeUnmount, onUnmounted, onActivated, onDeactivated, onErrorCaptured } from 'vue'
   
   export default {
     setup() {
       onBeforeMount(() => {
         // ... 
       })
       onMounted(() => {
         // ... 
       })
       onBeforeUpdate(() => {
         // ... 
       })
       onUpdated(() => {
         // ... 
       })
       onBeforeUnmount(() => {
         // ... 
       })
       onUnmounted(() => {
         // ... 
       })
       onActivated(() => {
         // ... 
       })
       onDeactivated(() => {
         // ... 
       })
       onErrorCaptured(() => {
         // ... 
       })
     }
   }
   
   // 应该是这么使用的
   beforeCreate -> use setup()
   created -> use setup()
   beforeMount -> onBeforeMount
   mounted -> onMounted
   beforeUpdate -> onBeforeUpdate
   updated -> onUpdated
   beforeDestroy -> onBeforeUnmount
   destroyed -> onUnmounted
   errorCaptured -> onErrorCaptured
   ```

### 生命周期函数的实现 （粗糙）

- ```js
  Vue.prototype._init = function (options?: Object) {
  	...
      // expose real self
      vm._self = vm
      initLifecycle(vm)
      initEvents(vm)
      initRender(vm)
      callHook(vm, 'beforeCreate')
      initInjections(vm) // resolve injections before data/props
      initState(vm)
      initProvide(vm) // resolve provide after data/props
      callHook(vm, 'created')
      ...
      if (vm.$options.el) {
        vm.$mount(vm.$options.el)
      }
   }
  ```

- `Vue先调用了initLifecycle(vm)、initEvents(vm)、initRender(vm)`这三个方法，用于初始化`生命周期、事件、渲染函数`，这些过程发生在`Vue初始化的过程(_init方法)中`，并在调用`beforeCreate钩子`之前。

- 然后Vue通过`callHook (vm: Component, hook: string)`方法来去调用`钩子函数(hook)`，它接收`vm（Vue实例对象），hook（钩子函数名称）`来去执行`生命周期函数`。在Vue中**几乎所有的钩子（`errorCaptured`除外）函数执行都是通过`callHook (vm: Component, hook: string)`来调用的**。

- ```js
  // callHook.js
  export function callHook (vm: Component, hook: string) {
    // #7573 disable dep collection when invoking lifecycle hooks
    pushTarget()
    const handlers = vm.$options[hook]
    if (handlers) {
      for (let i = 0, j = handlers.length; i < j; i++) {
        try {
          handlers[i].call(vm)
        } catch (e) {
          handleError(e, vm, `${hook} hook`)
        }
      }
    }
    if (vm._hasHookEvent) {
      vm.$emit('hook:' + hook)
    }
    popTarget()
  }
  ```

- 模板解析 



# 浏览器

## 浏览器渲染

### 1. 浏览器的内核

- Chrome：之前是 Webkit现在是 Blink
- Firefox: Gecko
- IE : Trident
- Safari: Webkit
- Opera:  以前是 Webkit 现在是Blink

### 2.浏览器渲染流程

- 构建DOM树
  - 浏览器从本地磁盘或者网络中读写HTML的原始字节，并根据指定编码(一般是UFT-8)将他们转化为字符串。（原始字节内容 就是 0 和 1这些字节数据，当浏览器接收到这些数据以后，会将这些字节数据转化成字符串，也就是我们所写的代码）
  - 再将字符串转换成 TOKEN（标签），例如 <html> <body> 等，**token会标识当前token是开始标签还是结束标签亦或者是文本等信息，token也会识别出节点之间的父子关系或兄弟关系**
  - 生成节点对象 并构建DOM
- 构建CSSOM
  - DOM捕获了页面内容，但是浏览器还不知道页面内容该如何展示，所以需要构建CSSOM。构建过程与构建DOM的过程相似。
  - 当浏览器接收到一段css时，浏览器首先识别出token，然后构建节点生成CSSOM（这个过程是很消耗资源的，因为样式除了自己设置的还有继承祖先的，这样子就需要递归遍历CSSOM树，然后才能确定每一个节点元素具体的样式）
- 生成渲染树
  - 并不是简单的将两者合并，渲染树只会包括需要显示的节点和这些节点的样式信息，如果是 display：none 的，那么就不会在渲染树中显示。
- 布局（回流）
  - 浏览器生成渲染树之后，就会根据渲染树来进行布局（也可以叫回流）。这一阶段浏览器要做的事情就是要弄清楚每个节点在页面中确切的位置和大小。通常这一行为也可以叫自动重排。
  - 布局流程的输出是一个盒模型，它会精确的捕获每个元素在视口内的确切位置和尺寸，所有相对测量值都将转换成屏幕上的绝对像素。
- 绘制
  - 布局完成之后，浏览器就会立即发出Paint Setup 和Paint事件，将渲染后转换成屏幕上的像素。



### 3.回流和重绘

- 回流
  - 当render tree 的一部分或全部的元素因改变了自身的宽高，布局，显示或隐藏，或者元素内部的文字结构发生变化 导致需要重新构建页面的时候，回流就产生了。
- 重绘
  - 当一个元素自身的宽高，布局，及显示或隐藏没有改变，而只是改变了元素的外观风格的时候，就会产生重绘。例如你改变了元素的background-color。
- 结论
  - 回流一定会触发重绘，重绘不一定触发回流

### 4.一些小问题

- 渲染时遇到JS文件 要怎么处理？
  - JS 的加载、解析与执行过程会阻塞DOM 的构建，即在构建DOM时，HTML解析器遇到了 JS 文件，就会暂停DOM的构建，将控制权交给 JS 引擎。等JS引擎运行完毕，浏览器再从终端恢复DOM的构建。
  - JS 不只阻塞了DOM的构建，也会阻塞CSSOM的构建，因为 JS 不只是可以更改 DOM 元素， 也可以修改样式，而不完成的CSSOM 是不能够被使用的，所以浏览器将延迟脚本执行和DOM构建，直到CSSOM 下载和构建完毕。
  - 总结：先下载构建 CSSOM，然后执行 JS  脚本文件，最后构建DOM。（倒是首屏渲染 白屏  时间慢 等问题）



### 5. 一些小建议

- 减少回流和重绘

  - 减少DOM样式操作，不如预先定好 class ，再修改 className
  - 尽量不使用table布局
  - 为动画的html元件使用fixed 或 absolute 的position， 

- 首屏渲染 

  -  将 JS 文件放在底部

  - 使用 defer 属性（只有IE 支持）

  - async 属性

    ```html
    <script type="text/javascript" src="demo_async.js" async="async"></script>
    ```

    



## 输入一个url到看到页面的过程

1、输入url

2、查看浏览器缓存，看是否有缓存，如果有缓存，继续查看缓存是否过期，如果没有过期，直接返回缓存页面，如果没有缓存或者缓存过期，发送一个请求。

3、浏览器解析url地址，获取协议、主机名、端口号和路径。

4、获取主机ip地址过程

（1）浏览器缓存

（2）主机缓存

（3）hosts文件

（4）路由器缓存

（5）DNS缓存

（6）DNS递归查询

5、浏览器发起和服务器的TCP连接，执行三次握手（你吃饭了吗？ 我吃饭了，你呢？ 我也吃完了。）

	-	三次握手：建立连接，客户端发送 syn包(syn=j) 到服务器，并进入 syn_send ,  等待服务器确认。
	-	服务器收到 syn 包之后，确认是客户的SYN（ack=j+1），同时自己也发送一个 SYN包（syn=k）， 即 syn+ack包，此时服务器进入 syn_recv状态
	-	客户端接收到了服务器发送的 syn+ack包，向服务器发送确认包 ack(ack= k+1)，此包发送完毕，客户端和服务端进入 established 状态，完成三次握手。

6、三次握手连接后，浏览器发送一个http请求（这部分是重点，请查询相关资料，详细了解http协议关于请求格式和重要的几个请求头字段含义）。

7、服务器收到请求，转到相关的服务程序，期间可能需要连接并操作数据库（主要分get和post请求）。

8、服务器看是否需要缓存，服务器处理完请求，发出一个响应（这部分也是重点，请查询资料了解http响应头各个字段的含义）

9、服务器并根据请求头包含信息决定是否需要关闭TCP连接（如需关闭，则需要四次挥手过程）

10、浏览器对接收到的响应进行解码

11、浏览器解析收到的响应并根据响应的内容（假如是HTML文件）进行构建DOM树，构建render树，渲染render树等过程

12、处理嵌入的其他资源如css文件、js文件、图片文件、音视频等文件，处理过程类似上面的步骤在此不详述。





# JS

## Canvas 的使用

1. 一些基础知识点

   - 定义对齐方式：`ctx.textBaseline = 'top'`
   - 填充不同颜色：`ctx.fillstyle= 颜色、渐变`
   - 保存当前状态,入栈：`ctx.save()`
   - 平移：`ctx.translate(x,y)`
   - 旋转：`ctx.rotate(deg)`
   - 填充文字：`ctx.filltext(text,x,y)`
   - restore 出栈：`ctx.restore()`
   - 开始一条路径：`ctx.beginPath()`
   - 移动到某点，初始点：`ctx.moveTo(x,y)`
   - 添加新点，连接该点：`ctx.lineTo(x,y)`
   - 从当前点回初始点：`ctx.closePath()'`
   - 绘制已定义的路径：`ctx.stroke()`
   - 画圆，有弧度的都能画：`ctx.arc(x,y,r半径,起始角度,结束角度)`
   - 填充当前绘图（路径）。：`ctx.fill()`

2. 验证码的使用

   ```vue
   <template>
     <div class="img-verify">
       <canvas ref="verify" :width="width" :height="height" @click="handleDraw"></canvas>
     </div>
   </template>
   
   <script type="text/ecmascript-6">
   import { reactive, onMounted, ref, toRefs } from 'vue'
   export default {
     setup() {
       const verify = ref(null)
       const state = reactive({
         pool: 'ABCDEFGHIJKLMNOPQRSTUVWXYZ1234567890', // 字符串
         width: 120,
         height: 40,
         imgCode: ''
       })
       onMounted(() => {
         // 初始化绘制图片验证码
         state.imgCode = draw()
       })
   
       // 点击图片重新绘制
       const handleDraw = () => {
         state.imgCode = draw()
       }
   
       // 随机数
       const randomNum = (min, max) => {
         return parseInt(Math.random() * (max - min) + min)
       }
       // 随机颜色
       const randomColor = (min, max) => {
         const r = randomNum(min, max)
         const g = randomNum(min, max)
         const b = randomNum(min, max)
         return `rgb(${r},${g},${b})`
       }
   
       // 绘制图片
       const draw = () => {
         // 3.填充背景颜色，背景颜色要浅一点
         const ctx = verify.value.getContext('2d')
         // 填充颜色
         ctx.fillStyle = randomColor(180, 230)
         // 填充的位置
         ctx.fillRect(0, 0, state.width, state.height)
         // 定义paramText
         let imgCode = ''
         // 4.随机产生字符串，并且随机旋转
         for (let i = 0; i < 4; i++) {
           // 随机的四个字
           const text = state.pool[randomNum(0, state.pool.length)]
           imgCode += text
           // 随机的字体大小
           const fontSize = randomNum(18, 40)
           // 字体随机的旋转角度
           const deg = randomNum(-30, 30)
           /*
            * 绘制文字并让四个文字在不同的位置显示的思路 :
            * 1、定义字体
            * 2、定义对齐方式
            * 3、填充不同的颜色
            * 4、保存当前的状态（以防止以上的状态受影响）
            * 5、平移translate()
            * 6、旋转 rotate()
            * 7、填充文字
            * 8、restore出栈
            * */
           ctx.font = fontSize + 'px Simhei'
           ctx.textBaseline = 'top'
           ctx.fillStyle = randomColor(80, 150)
           /*
            * save() 方法把当前状态的一份拷贝压入到一个保存图像状态的栈中。
            * 这就允许您临时地改变图像状态，
            * 然后，通过调用 restore() 来恢复以前的值。
            * save是入栈，restore是出栈。
            * 用来保存Canvas的状态。save之后，可以调用Canvas的平移、放缩、旋转、错切、裁剪等操作。 restore：用来恢复Canvas之前保存的状态。防止save后对Canvas执行的操作对后续的绘制有影响。
            *
            * */
           ctx.save()
           ctx.translate(30 * i + 15, 15)
           ctx.rotate((deg * Math.PI) / 180)
           // fillText() 方法在画布上绘制填色的文本。文本的默认颜色是黑色。
           // 请使用 font 属性来定义字体和字号，并使用 fillStyle 属性以另一种颜色/渐变来渲染文本。
           // context.fillText(text,x,y,maxWidth);
           ctx.fillText(text, -15 + 5, -15)
           ctx.restore()
         }
         // 5.随机产生5条干扰线,干扰线的颜色要浅一点
         for (let i = 0; i < 5; i++) {
           ctx.beginPath()
           ctx.moveTo(randomNum(0, state.width), randomNum(0, state.height))
           ctx.lineTo(randomNum(0, state.width), randomNum(0, state.height))
           ctx.closePath()
           ctx.stroke()
           ctx.strokeStyle = randomColor(180, 230)
         }
         // 6.随机产生40个干扰的小点
         for (let i = 0; i < 40; i++) {
           ctx.beginPath()
           ctx.arc(randomNum(0, state.width), randomNum(0, state.height), 1, 0, 2 * Math.PI)
           ctx.closePath()
           ctx.fillStyle = randomColor(150, 200)
           ctx.fill()
         }
         return imgCode
       }
   
       return {
         ...toRefs(state),
         verify,
         handleDraw
       }
     }
   }
   </script>
   <style type="text/css">
   .img-verify canvas {
     cursor: pointer;
   }
   </style>
   ```

   



## JS 事件循环

1. 首先  js 是单线程执行任务。 任务又分为 同步和异步任务。  异步任务又包含（宏任务和微任务）

2. ![eventloop](C:\Users\cws\Pictures\Saved Pictures\eventloop.png)

   - 解读：JS单线程执行任务，遇到任务时会自动将其分类，进入不同“场所”，同步进入主线程执行，异步任务则进入 Event Table 并注册回调函数，等待执行。
   - 主线程中的任务执行完毕为空的时候，则会开始读取 Event Queue 中 注册的回调函数，进入主线程进行执行。
   - 重复以上步骤，即为 Event Loop

   

3. 宏任务 和微任务

   - ![task](C:\Users\cws\Pictures\Saved Pictures\task.png)
   - 宏任务一般：script, setTimeout, setInterval,setImmediate
   - 微任务：promise.then(有些实现的promise将then方法放到了宏任务中), process.nextTick,MutationObserver

4. 实例分析

   - ```js
     console.log('1');
     
     setTimeout(function() {
         console.log('2');
         process.nextTick(function() {
             console.log('3');
         })
         new Promise(function(resolve) {
             console.log('4');
             resolve();
         }).then(function() {
             console.log('5')
         })
     })
     process.nextTick(function() {
         console.log('6');
     })
     new Promise(function(resolve) {
         console.log('7');
         resolve();
     }).then(function() {
         console.log('8')
     })
     
     setTimeout(function() {
         console.log('9');
         process.nextTick(function() {
             console.log('10');
         })
         new Promise(function(resolve) {
             console.log('11');
             resolve();
         }).then(function() {
             console.log('12')
         })
     })
     
     // 1 7 6 8 2 4 3 5 9 11 10 12
     
     // 如果遇到 async 和 await  转成 new Promise 同理可得出来
     
     ```

   - 解析：

     - 整体script 进入主线程，遇到 `console.log('1');` ， 直接执行 输入 `1`，
     - 接着遇到第一个 `setTimeout` 将它的回调函数 放到 宏任务 event Queue。暂时记为 `setTimeout1`
     - 遇到 `process.nextTick（）`   将之 放置 微任务 event Queue。 暂时记为 `process1`
     - 遇到 `promise`  `new Promise` 直接执行， 输入 `7`，then 被放置在微任务 event Queue 中，暂时记为 `then1`
     - 再次遇到一个 `setTimeout` 将他的回调放到 宏任务 event Queue， 暂时记为 `setTimeout2`
     -  ---------  此时已经输入了 1 和 7，  微任务中存在 `process1` 和 `then1` ， 执行输入 6 和 8
     - ----------  第一轮 循环 结束输出 1 7 6 8
     - 接着执行宏任务的 第一个 `setTimeout1`  ，首先遇到 `console.log('2');`  输出2 ，遇到process，将之放置在微任务 event Queue 中， 暂时记为 `process2` ,  接着遇到 `new Promise` 立即执行 `console.log('4');`，把then 函数放置在微任务 event Queue 中， 暂时记为 `then2`
     - 宏任务 `setTimeout1 `  执行完毕，微任务中存在 `process2` 和 `then2` ， 执行输入 3 和 5 
     - ----------  第二轮 循环结束 输出 1 7 6 8  2  4 3 5 
     - 继续执行 宏任务中 第二个 `setTimeout2` , 同理 可得  
     - ----------  第二轮 循环结束 输出 1 7 6 8  2  4 3 5  9 11 10 12





## 虚拟DOM

### 1.什么是虚拟DOM？

- 虚拟Dom，就是用一个 `JS` 对象来描述一个 `DOM` 节点。例如：

  - ```js
    <div class="a" id="b">我是内容</div>
    
    {
      tag:'div',        // 元素标签
      attrs:{           // 属性
        class:'a',
        id:'b'
      },
      text:'我是内容',  // 文本内容
      children:[]       // 子元素
    }
    ```

- 为什么需要虚拟Dom？

  - `Vue`是数据驱动视图的，数据发生变化视图就要随之更新，在更新视图的时候难免要操作`DOM`,而操作真实`DOM`又是非常耗费性能的，这是因为浏览器的标准就把 `DOM` 设计的非常复杂，所以一个真正的 `DOM` 元素是非常庞大的。我们可以用`JS`模拟出一个`DOM`节点，称之为虚拟`DOM`节点。当数据发生变化时，我们对比变化前后的虚拟`DOM`节点，通过`DOM-Diff`算法计算出需要更新的地方，然后去更新需要更新的视图。说白了，实际上就是用`JS`的计算性能来换取操作真实`DOM`所消耗的性能。

- `VNode` 类，vue中的 虚拟节点

  - ```js
    // 源码位置：src/core/vdom/vnode.js
    
    export default class VNode {
      constructor (
        tag?: string,
        data?: VNodeData,
        children?: ?Array<VNode>,
        text?: string,
        elm?: Node,
        context?: Component,
        componentOptions?: VNodeComponentOptions,
        asyncFactory?: Function
      ) {
        this.tag = tag                                /*当前节点的标签名*/
        this.data = data        /*当前节点对应的对象，包含了具体的一些数据信息，是一个VNodeData类型，可以参考VNodeData类型中的数据信息*/
        this.children = children  /*当前节点的子节点，是一个数组*/
        this.text = text     /*当前节点的文本*/
        this.elm = elm       /*当前虚拟节点对应的真实dom节点*/
        this.ns = undefined            /*当前节点的名字空间*/
        this.context = context          /*当前组件节点对应的Vue实例*/
        this.fnContext = undefined       /*函数式组件对应的Vue实例*/
        this.fnOptions = undefined
        this.fnScopeId = undefined
        this.key = data && data.key           /*节点的key属性，被当作节点的标志，用以优化*/
        this.componentOptions = componentOptions   /*组件的option选项*/
        this.componentInstance = undefined       /*当前节点对应的组件的实例*/
        this.parent = undefined           /*当前节点的父节点*/
        this.raw = false         /*简而言之就是是否为原生HTML或只是普通文本，innerHTML的时候为true，textContent的时候为false*/
        this.isStatic = false         /*静态节点标志*/
        this.isRootInsert = true      /*是否作为跟节点插入*/
        this.isComment = false             /*是否为注释节点*/
        this.isCloned = false           /*是否为克隆节点*/
        this.isOnce = false                /*是否有v-once指令*/
        this.asyncFactory = asyncFactory
        this.asyncMeta = undefined
        this.isAsyncPlaceholder = false
      }
    
      get child (): Component | void {
        return this.componentInstance
      }
    }
    ```

### 2. DOM-Diff 算法

- patch:
  - 本质上都是对比新旧两份 `VNode` 的过程。
  - 思想：所谓旧的`VNode`(即`oldVNode`)就是数据变化之前视图所对应的虚拟`DOM`节点，而新的`VNode`是数据变化之后将要渲染的新的视图所对应的虚拟`DOM`节点，所以我们要以生成的新的`VNode`为基准，对比旧的`oldVNode`，如果新的`VNode`上有的节点而旧的`oldVNode`上没有，那么就在旧的`oldVNode`上加上去；如果新的`VNode`上没有的节点而旧的`oldVNode`上有，那么就在旧的`oldVNode`上去掉；如果某些节点在新的`VNode`和旧的`oldVNode`上都有，那么就以新的`VNode`为准，更新旧的`oldVNode`，从而让新旧`VNode`相同。
    - 创建节点：新的`VNode`中有而旧的`oldVNode`中没有，就在旧的`oldVNode`中创建。
    - 删除节点：新的`VNode`中没有而旧的`oldVNode`中有，就从旧的`oldVNode`中删除。
    - 更新节点：新的`VNode`和旧的`oldVNode`中都有，就以新的`VNode`为准，更新旧的`oldVNode`。

### 3. 创建节点

- 实际上只有3种类型的节点能够被创建并插入到 `DOM` 中，分别是：元素节点、文本节点、注释节点

- ```js
  // 源码位置: /src/core/vdom/patch.js
  function createElm (vnode, parentElm, refElm) {
      const data = vnode.data
      const children = vnode.children
      const tag = vnode.tag
      // 元素节点 判断是否有 tag
      // 注释节点 判断 isComment 属性是否 为 true
      // 既不是元素节点 也不是注释节点 就认为是文本节点
      if (isDef(tag)) {
          vnode.elm = nodeOps.createElement(tag, vnode)   // 创建元素节点
          createChildren(vnode, children, insertedVnodeQueue) // 递归遍历 创建元素节点的子节点
          insert(parentElm, vnode.elm, refElm)       // 插入到DOM中
      } else if (isTrue(vnode.isComment)) {
          vnode.elm = nodeOps.createComment(vnode.text)  // 创建注释节点
          insert(parentElm, vnode.elm, refElm)           // 插入到DOM中
      } else {
          vnode.elm = nodeOps.createTextNode(vnode.text)  // 创建文本节点
          insert(parentElm, vnode.elm, refElm)           // 插入到DOM中
      }
  }
  ```



### 4.删除节点

- 如果 新 `VNode` 中没有而 旧 `VNode` 中有，只需在要删除节点的父元素上调用`removeChild`方法即可

  ```javascript
  function removeNode (el) {
      const parent = nodeOps.parentNode(el)  // 获取父节点
      if (isDef(parent)) {
        nodeOps.removeChild(parent, el)  // 调用父节点的removeChild方法
      }
    }
  ```



### 5.更新节点

- 更新节点的时候我们需要对以下3种情况进行判断并分别处理：
  - 如果`VNode`和`oldVNode`均为静态节点，则直接跳过。
  - 如果 `VNode` 是文本节点即表示这个节点内只包含纯文本，那么只需看 `oldVNode` 是否也是文本节点，如果是，那就比较两个文本是否不同，如果不同则把 `oldVNode` 里的文本改成跟 `VNode` 的文本一样。如果 `oldVNode` 不是文本节点，那么不论它是什么，直接调用 `setTextNode` 方法把它改成文本节点，并且文本内容跟 `VNode `相同。





## 防抖节流

1. 定义

   - 节流: n 秒内只运行一次，若在 n 秒内重复触发，只有一次生效

   - 防抖: n 秒后在执行该事件，若在 n 秒内被重复触发，则重新计时

   - 一个经典比喻：想象每天上班大厦底下的电梯。把电梯完成一次运送，类比为一次函数的执行和响应

     假设电梯有两种运行策略 `debounce` 和 `throttle`，超时设定为15秒，不考虑容量限制

     电梯第一个人进来后，15秒后准时运送一次，这是节流

     电梯第一个人进来后，等待15秒。如果过程中又有人进来，15秒等待重新计时，直到15秒后开始运送，这是防抖

2. 代码实现

   - 节流：使用时间戳与定时器的写法

     - ```js
       // 闭包 在内存中维持一个变量，可以做缓存（但使用多了同时也是一项缺点，消耗内存）
       function throttled(fn, delay) {
           let timer = null
           let starttime = Date.now() // 这个不会被销毁，产生内存消耗
           return function () {
               let curTime = Date.now() // 当前时间
               let remaining = delay - (curTime - starttime)  // 从上一次到现在，还剩下多少多余时间
               let context = this
               let args = arguments
               clearTimeout(timer)
               if (remaining <= 0) {
                   fn.apply(context, args)
                   starttime = Date.now()
               } else {
                   timer = setTimeout(fn, remaining);
               }
           }
       }
       ```

   - 防抖：

     - ```js
       function debounce(func, wait) {
           let timeout;
       
           return function () {
               let context = this; // 保存this指向
               let args = arguments; // 拿到event对象
       
               clearTimeout(timeout)
               timeout = setTimeout(function(){
                   func.apply(context, args)
               }, wait);
           }
       }
       ```

3. 应用场景

   - 防抖在连续的事件，只需触发一次回调的场景有：
     - 搜索框搜索输入。只需用户最后一次输入完，再发送请求
     - 手机号、邮箱验证输入检测
     - 窗口大小`resize`。只需窗口调整完成后，计算窗口大小。防止重复渲染。
   - 节流在间隔一段时间执行一次回调的场景有：
     - 滚动加载，加载更多或滚到底部监听
     - 搜索框，搜索联想功能

   





# **webpack**

### 介绍：

- webpack  是一个前端项目构建工具，他是基于 Node.js 开发出来的一个前端工具。
- 能够处理 JS 文件的互相依赖关系



### 构建流程：

- `初始化参数`：解析webpack配置参数，合并shell脚本语言命令和webpack.config.js文件配置的参数,形成最后的配置结果；
- `开始编译`：上一步得到的参数构建compiler对象，注册所有配置的插件(WebpackOptionsApply首先会初始化几个基础插件，然后把options中对应的选项进行require)，插件 监听webpack构建生命周期的事件节点，做出相应的反应，执行对象的run方法开始执行编译；
- `确定入口`：从配置的entry入口，开始解析文件构建AST语法树，找出依赖，递归下去
- `编译模块`：递归中根据文件类型和loader配置，调用所有配置的loader对文件进行转换，再找出该模块依赖的模块，再递归本步骤直到所有入口依赖的文件都经过了本步骤的处理；
  - `完成模块编译并输出`：递归完事后，得到每个文件结果，包含每个模块以及他们之间的依赖关系，根据entry或分包配置生成代码块chunk; 逐次对每一个module和chunk进行整理，生成编译后的源码，合并拆分，每一个chunk对应一个入口文件。
- `输出完成`：输出所有的chunk到文件系统(output中的path)



### 1. Webpack中Loader和Plugin的区别？编写Loader，Plugin的思路？	

1. 区别：
   - loader：文件加载器，能够加载资源文件，并进行一些处理，比如：编译，压缩，最终打包到置顶为文件中。从运行时机上来看，loader是运行在打包文件之前。对于`loader`，实质是一个转换器，将A文件进行编译形成B文件，操作的是文件，比如将`A.scss`或`A.less`转变为`B.css`，单纯的文件转换过程。
   - plugin：赋予了 webpack 各种灵活的功能，例如打包优化、资源管理、环境变量注入等。目的是解决 loader 无法实现的其他事情。从运行时机上来看，plugin 是整个编译周期都在起到了作用。
   
2. 编写
   - loader 的编写：
     
     - loader：本质上是一个函数，函数中 `this` 作为上下文被 `webapck` 填充，所以不能将 `loader` 设为一个箭头函数。
     
     - ```js
       
       // 导出一个函数，source为webpack传递给loader的文件源内容
       module.exports = function(source) {
           const content = doSomeThing2JsString(source);
           
           // 如果 loader 配置了 options 对象，那么this.query将指向 options
           const options = this.query;
           
           // 可以用作解析其他模块路径的上下文
           console.log('this.context');
           
           /*
            * this.callback 参数：
            * error：Error | null，当 loader 出错时向外抛出一个 error
            * content：String | Buffer，经过 loader 编译后需要导出的内容
            * sourceMap：为方便调试生成的编译后内容的 source map
            * ast：本次编译生成的 AST 静态语法树，之后执行的 loader 可以直接使用这个 AST，进而省去重复生成 AST 的过程
            */
           this.callback(null, content); // 异步
           return content; // 同步
       }
       ```
     
       
     
   - plugin 的编写：
   
     - plugin 规范：
   
       - 插件必须是一个函数或者是一个包含 `apply` 方法的对象，这样才能访问`compiler`实例
       - 传给每个插件的 `compiler` 和 `compilation` 对象都是同一个引用，因此不建议修改
       - 异步的事件需要在插件处理完任务时调用回调函数通知 `Webpack` 进入下一个流程，不然会卡住
   
     - 模板
   
       - ```js
         class MyPlugin {
         	// webpack 会使用 MyPlugin 实例的 apply 方法给插件实例传入 compiler 对象
         	apply(compiler){
         		compiler.hooks.emit.tap('MyPlugin',compilation => {
         			
         			console.log(compilation)
         			
         			// do something
         		})
         	}
         }
         ```
   
   

### 2. 为什么使用webpack? vite?

1. 模块打包工具 webpack， 首先要清楚为什么要打包？
   - 我们的html页面会引入多个js文件，多个js文件可能还存在依赖关系，所以浏览器需要发送多次http请求来获取这些js文件。但是如果其中有一个文件因为网络问题而延误了时间，那么就会延误整个页面的显示。这个我们就想合并成一个js文件，只发送一次http请求不就好了，这样子就可以减少http请求数量。而 **在开发完成后这个合并的过程就是打包** 。webpack在打包过程中会分析各个文件之间的依赖关系，然后生成一个依赖图并用文件的形式保存下来。
2. 知道了打包之后，就应该知道什么是模块？
   - 首先想一个问题，没有模块化的代码是什么样子的？
     - 变量和方法不容易维护，容易污染全局变量
     - 加载资源的方式通过<script> 标签从上而下
     - 依赖的环境主观逻辑偏重，代码较多就会变得复杂
     - 大型项目资源难以维护，特别是多人合作的情况下，资源的引入会让人崩溃
   - 模块化的作用：
     - 可以解决命名冲突
     - 管理依赖
     - 提高代码的可读性
     - 代码解耦，提高代码的复用性
   



### 3. 有哪些常见的Loader？他们是解决什么问题的？

1. file-loader：把文件输出到一个文件夹中，在代码中通过相对 URL 去引用输出的文件
2. url-loader：和 file-loader 类似，但是能在文件很小的情况下以 base64 的方式把文件内容注入到代码中去
3. source-map-loader：加载额外的 Source Map 文件，以方便断点调试
4. image-loader：加载并且压缩图片文件
5. babel-loader：把 ES6 转换成 ES5
6. css-loader：加载 CSS，支持模块化、压缩、文件导入等特性
7. style-loader：把 CSS 代码注入到 JavaScript 中，通过 DOM 操作去加载 CSS。
8. eslint-loader：通过 ESLint 检查 JavaScript 代码



#  前端工程化、模块化、组件化

1. 前端工程化

   - 工程化是一种思想而不是某种技术(当然为了实现工程化我们会用一些技术)
   - 再用一句通俗的话来概括前端工程化:前端工程化就是用做工程的思维看待和开发自己的项目,而不再是直接撸起袖子一个页面一个页面开写
   - 从工具入手，工程化包括
     - 模块化与组件化: npm, es6,seajs, react/angularjs/Vue
     - 代码版本管理: git
     - 代码风格管理: jscs, editorconfig
     - 代码编译: babel, less,sass,scss, imgmin, csssprit, inline-svg
     - 代码质量管理 (QA): eslint, mocha
     - 代码构建: webpack
     - 项目脚手架: yeoman
     - 持续集成/持续交付/持续部署: jenkins
     - 本地化与国际化
   - 执行工程化:
     - 在配置初始项目文件结构和基本文件依靠命令行（工具）自动生成。
     - 确定代码规范，缩进，换行，以及各种预编译工具less，coffee，保证输出代码的标准一致
     - 对JS文件是否规范化，进行单元测试，不用手动复制到jslint上去检测，现在配置grunt监听文件变动自动检验显示检验结果还可以通过配置构建工具自动刷新浏览器实现文件实时变动监听。
     - 以前压缩合并文件用手工复制到压缩工具然后复制到一个文件里面，现在配置一下webpack，webpack可以做自动任务，实时编译，并且监测文件改变而做出响应。
     - 以前发布到服务器上，要手动使用 FTP 软件上传，现在也可以用工具自动打包上传。

   

2. 模块化开发

   - **一个模块就是一个实现特定功能的文件，有了模块我们就可以更方便的使用别人的代码，要用什么功能就加载什么模块。**

   

3. 组件化开发

   - **组件化将页面视为一个容器,页面上各个独立部分例如:头部、导航、焦点图、侧边栏、底部等视为独立组件,不同的页面根据内容的需要,去盛放相关组件即可组成完整的页面。**
   - ①页面上的每个独立的、可视/可交互区域视为一个组件;
     ②每个组件对应一个工程目录,组件所需的各种资源都在这个目录下就近维护;
     ③由于组件具有独立性,因此组件与组件之间可以 自由组合;
     ④页面只不过是组件的容器,负责组合组件形成功能完整的界面;
     ⑤当不需要某个组件,或者想要替换组件时,可以整个目录删除/替换。
   - 组件化开发能大幅提高应用开发效率、测试性、复用性
   - 常用得组件化技术：属性、自定义事件、插槽
   - 降低更新范围，值重新渲染变化的组件
   - 高内聚、低耦合、单向数据流(父组件传递数据给子组件，子组件不要修改)





# Git 常用命令

## 1. 配置

1.  **第一件事情设置你得用户名和邮件地址** 设置提交代码时的用户信息命令如下：
   - git config [--global] user.name "[name]"
   - git config [--global] user.email "[email address]"



## 2. 启动

1. 初始化的两个操作
   - git init [project-name]：创建或在当前目录初始化一个git代码库
   - git clone url：下载一个项目和它的整个代码历史

## 3. 日常基本操作

1. 代码常用操作
   - git init 初始化仓库，默认为 master 分支
   - git add . 提交全部文件修改到缓存区
   - git add <具体某个文件路径+全名> 提交某些文件到缓存区
   - git diff  查看当前代码 add后，会 add 哪些内容
   - git diff --staged查看现在 commit 提交后，会提交哪些内容
   - git status 查看当前分支状态
   - git pull <远程仓库名> <远程分支名> 拉取远程仓库的分支与本地当前分支合并
   - git pull <远程仓库名> <远程分支名>:<本地分支名> 拉取远程仓库的分支与本地某个分支合并
   - git commit -m "<注释>" 提交代码到本地仓库，并写提交注释
   - git commit -v 提交时显示所有diff信息
   - git commit --amend [file1] [file2] 重做上一次commit，并包括指定文件的新变化
2. 关于提交信息的格式，可以遵循以下的规则：
   - feat: 新特性，添加功能
   - fix: 修改 bug
   - refactor: 代码重构
   - docs: 文档修改
   - style: 代码格式修改, 注意不是 css 修改
   - test: 测试用例修改
   - chore: 其他修改, 比如构建流程, 依赖管理



## 4. 分支操作

1. 
   - git branch 查看本地所有分支
   - git branch -r 查看远程所有分支
   - git branch -a 查看本地和远程所有分支
   - git merge <分支名> 合并分支
   - git merge --abort 合并分支出现冲突时，取消合并，一切回到合并前的状态
   - git branch <新分支名> 基于当前分支，新建一个分支
   - git checkout --orphan <新分支名> 新建一个空分支（会保留之前分支的所有文件）
   - git branch -D <分支名> 删除本地某个分支
   - git push <远程库名> :<分支名> 删除远程某个分支
   - git branch <新分支名称> <提交ID> 从提交历史恢复某个删掉的某个分支
   - git branch -m <原分支名> <新分支名> 分支更名
   - git checkout <分支名> 切换到本地某个分支
   - git checkout <远程库名>/<分支名> 切换到线上某个分支
   - git checkout -b <新分支名> 把基于当前分支新建分支，并切换为这个分支



## 5. 远程同步

1. 远程操作常见的命令：
   - git fetch [remote] 下载远程仓库的所有变动
   - git remote -v 显示所有远程仓库
   - git pull [remote] [branch] 拉取远程仓库的分支与本地当前分支合并
   - git fetch 获取线上最新版信息记录，不合并
   - git push [remote] [branch] 上传本地指定分支到远程仓库
   - git push [remote] --force 强行推送当前分支到远程仓库，即使有冲突
   - git push [remote] --all 推送所有分支到远程仓库



## 6. 撤销

1. 

   - git checkout [file] 恢复暂存区的指定文件到工作区
   - git checkout [commit] [file]  恢复某个commit的指定文件到暂存区和工作区
   - git checkout . 恢复暂存区的所有文件到工作区
   - git reset [commit] 重置当前分支的指针为指定commit，同时重置暂存区，但工作区不变
   - git reset --hard 重置暂存区与工作区，与上一次commit保持一致
   - git reset [file] 重置暂存区的指定文件，与上一次commit保持一致，但工作区不变
   - git revert [commit]  后者的所有变化都将被前者抵消，并且应用到当前分支

2. 补充说明

   - `reset`：真实硬性回滚，目标版本后面的提交记录全部丢失了

     `revert`：同样回滚，这个回滚操作相当于一个提价，目标版本后面的提交记录也全部都有



## 7. 存储操作

1. 你正在进行项目中某一部分的工作，里面的东西处于一个比较杂乱的状态，而你想转到其他分支上进行一些工作，但又不想提交这些杂乱的代码，这时候可以将代码进行存储
   - git stash 暂时将未提交的变化移除
   - git stash pop 取出储藏中最后存入的工作状态进行恢复，会删除储藏
   - git stash list 查看所有储藏中的工作
   - git stash apply <储藏的名称>  取出储藏中对应的工作状态进行恢复，不会删除储藏
   - git stash clear 清空所有储藏中的工作
   - git stash drop <储藏的名称>  删除对应的某个储藏



# echarts

1. 引入 Echarts

   - ```npm
     npm install echarts --save  // npm 安装 ECharts
     ```

   - ```
     import * as echarts from 'echarts'; // 全部引入 或者 按需引入均可
     ```

     



# 题



### 深层次遍历，改变所有的key

```js
eachReplaceMerchantCateKey(cate) {
    let item = [];
    cate.map((list) => {
        let newData = {};
        newData.label = list.cateName;
        newData.id = list.merchantProductCateUuid;
        if (list.children.length > 0) {
            newData.expand = true
            newData.children = this.eachReplaceMerchantCateKey(list.children);
        }
        item.push(newData);
    });
    return item;
},
```



### web 移动端自适应

1. ```
   npm install lib-flexible --save
   npm install postcss-pxtorem --save-dev
   ```

2. main.js

   ```js
   import 'lib-flexible/flexible'
   ```

3.  新建 postcss.config.js 文件(与package.json同级)

   ```js
   // postcss.config.js
   // 用 vite 创建项目，配置 postcss 需要使用 post.config.js，之前使用的 .postcssrc.js 已经被抛弃
   // 具体配置可以去 postcss-pxtorem 仓库看看文档
   module.exports = {
     "plugins": {
       "postcss-pxtorem": {
         rootValue: 37.5, // Vant 官方根字体大小是 37.5  // 设计图的百分之十
         propList: ['*'],
         selectorBlackList: ['.norem'] // 过滤掉.norem-开头的class，不进行rem转换
       }
     }
   }
   ```

   



### 1.什么样的 a 可以满足 (a === 1 && a === 2 && a === 3) === true 呢？(注意是 3 个 =，也就是严格相等)

```js
// 解法一 数据劫持  defineProperty
let current = 0
Object.defineProperty(window, 'a', {
  get () {
    current++
    // console.log(current)
    return current
  }
})
console.log(a === 1 && a === 2 && a === 3) // true
```

```js
// 解法二 proxy 代理

let current = {
    a: 0
}
let handler = {
    get(target, key, receiver) {
        // 递归创建并返回
        if (typeof target[key] === 'object' && target[key] !== null) {
            return new Proxy(target[key], handler)
        }
        target[key]++
        console.log(target[key]);
        return Reflect.get(target, key, receiver)
    },
    set(target, key, value, receiver) {
        console.log('set', key, value, target)
        return Reflect.set(target, key, value, receiver)
    }
}
let proxy = new Proxy(current, handler)
console.log(proxy.a === 1 && proxy.a === 2 && proxy.a === 3) // true

```

### 2.将数组arr=[[1,2,2],[3,4,5,5],[6, 7, 8, 9, [11, 12, [12, 13, [14]]]], 10]  扁平化 去重 排序

```js
var arr = [
    [1, 2, 2],
    [3, 4, 5, 5],
    [6, 7, 8, 9, [11, 12, [12, 13, [14]]]], 10
];
// 扁平化
let flatArr = arr.flat(Infinity)
// 去重  Array.from 可以 将类数组对象转换为真正数组  将Set结构的数据转换为真正的数组  将字符串转换为数组
let disArr = Array.from(new Set(flatArr))
// 排序
let result = disArr.sort((a, b) => {
    return a - b
})
console.log(result)
```



### 3.实现双向绑定Proxy比defineproperty优劣如何?

1. `Object.defineProperty`只能遍历对象属性进行劫持

   - ```js
     function observe(obj) {
         if (typeof obj !== 'object' || obj == null) {
             return
         }
         Object.keys(obj).forEach(key => {
             defineReactive(obj, key, obj[key])
         })
     }
     ```

2. `Proxy`直接可以劫持整个对象，并返回一个新对象，我们可以只操作新的对象达到响应式目的

   - ```js
     function reactive(obj) {
         if (typeof obj !== 'object' && obj != null) {
             return obj
         }
         // Proxy相当于在对象外层加拦截
         const observed = new Proxy(obj, {
             get(target, key, receiver) {
                 const res = Reflect.get(target, key, receiver)
                 console.log(`获取${key}:${res}`)
                 return res
             },
             set(target, key, value, receiver) {
                 const res = Reflect.set(target, key, value, receiver)
                 console.log(`设置${key}:${value}`)
                 return res
             },
             deleteProperty(target, key) {
                 const res = Reflect.deleteProperty(target, key)
                 console.log(`删除${key}:${res}`)
                 return res
             }
         })
         return observed
     }
     
     ```

3.  区别

   - vue2
     - 由于 JavaScript 的限制，Vue 不能检测以下变动的数组： 当你利用索引直接设置一个项时，例如：vm.items[indexOfItem] = newValue 当你修改数组的长度时，例如：vm.items.length = newLength，会导致严重的性能问题
     - vue 对于一个对象进行删除与添加属性操作，无法被劫持到
     - vue 中 无法监听  通过`index ` 对数组进行改变 与 `.length` 操作数组
     - 在`Vue2`中 增加了`set`、`delete` API，并且对数组`api`方法进行一个重写 

   - vue3
     - `Proxy`的监听是针对一个对象的，那么对这个对象的所有操作会进入监听操作，这就完全可以代理所有属性了
     - `Proxy`直接可以劫持整个对象，并返回一个新对象，我们可以只操作新的对象达到响应式目的
     - `Proxy`可以直接监听数组的变化（`push`、`shift`、`splice`）
     - `Proxy` 不兼容IE，也没有 `polyfill`, `defineProperty` 能支持到IE9

### 4.跨域问题

1. 跨域是什么？
   - 跨域本质是浏览器基于**同源策略**的一种安全手段
   - 同源：协议，主机（域名），端口都相同
   - 一定要注意跨域是浏览器的限制，你用抓包工具抓取接口数据，是可以看到接口已经把数据返回回来了，只是浏览器的限制，你获取不到数据。用postman请求接口能够请求到数据。这些再次印证了跨域是浏览器的限制。



1. proxy 正向代理

   ```js
   /* webpack.js */
   const path = require("path");
   const HtmlWebpackPlugin = require("html-webpack-plugin");
   
   module.exports = {
       entry: {
           index: "./index.js"
       },
       output: {
           filename: "bundle.js",
           path: path.resolve(__dirname, "dist")
       },
       devServer: {
           host: '127.0.0.1',
           port: 8084,
           open: true,// vue项目启动时自动打开浏览器
           proxy: {
               "/api": {
                   target: "http://localhost:8080" //这里面是你要访问的IP地址 即后台ip地址
                   changeOrigin: true,     //开启代理
               }
           }
       },
       plugins: [
           new HtmlWebpackPlugin({
               filename: "index.html",
               template: "webpack.html"
           })
       ]
   };
   ```

   ```js
   /* vue-cli 2.x */
   /* config/index.js */
   
   ...
   proxyTable: {
       '/api': {
           target: 'http://localhost:8080', //这里面是你要访问的IP地址 即后台ip地址
       }
       },
           ...
   ```

   - 服务端实现代理请求转发

     - ```js
       /* express 框架为例 */
       var express = require('express');
       const proxy = require('http-proxy-middleware')
       const app = express()
       app.use(express.static(__dirname + '/'))
       app.use('/api', proxy({ target: 'http://localhost:4000', changeOrigin: false
                             }));
       module.exports = app
       ```

   - 通过配置 nginx 实现代理

     - ```
       server {
           listen    80;
           # server_name www.josephxia.com;
           location / {
               root  /var/www/html;
               index  index.html index.htm;
               try_files $uri $uri/ /index.html;
           }
           location /api {
               proxy_pass  http://127.0.0.1:3000;
               proxy_redirect   off;
               proxy_set_header  Host       $host;
               proxy_set_header  X-Real-IP     $remote_addr;
               proxy_set_header  X-Forwarded-For  $proxy_add_x_forwarded_for;
           }
       }
       ```

2. CORS

   1. 跨域资源共享([CORS](https://developer.mozilla.org/zh-CN/docs/Glossary/CORS)) 是一种机制，它使用额外的 [HTTP](https://developer.mozilla.org/zh-CN/docs/Glossary/HTTP) 头来告诉浏览器 让运行在一个 origin (domain) 上的 Web 应用被准许访问来自不同源服务器上的指定的资源。当一个资源从与该资源本身所在的服务器**「不同的域、协议或端口」**请求一个资源时，资源会发起一个**「跨域 HTTP 请求」**。而在 cors 中会有 `简单请求` 和 `复杂请求`的概念。

   2. Node 中的解决方案

      ```js
      const express = require('express')
      const app = express()
      
      app.all('*', function(req, res, next) {
      	res.header('Access-Control-Allow-Origin', '*')
      	// res.header("Access-Control-Allow-Headers", "X-Requested-With");
      	res.header(
      		'Access-Control-Allow-Headers',
      		'Content-Type, Content-Length, Authorization, Accept, X-Requested-With , yourHeaderFeild'
      	)
      	res.header('Access-Control-Allow-Methods', 'PUT,POST,GET,DELETE,OPTIONS')
      	res.header('X-Powered-By', ' 3.2.1')
      	res.header('Content-Type', 'application/json;charset=utf-8')
      	next()
      })
      ```



### 5.BFC

1. BFC 就是块级格式上下文，是页面盒模型布局中的一种 CSS 渲染模式，相当于一个独立的容器，里面的元素和外部的元素相互不影响。

2. 创建BFC的方式

   1. html 根元素
   2. float 浮动
   3. 绝对定位
   4. overflow 不为 visiable
   5. display 为表格布局或者弹性布局
   6. 行内块元素 display:inline-block;
   7. contain值为layout、content或 strict的元素  .....等等

3. BFC特性：

   1. 内部box会在垂直方向，一个接一个地放置。
   2. Box垂直方向的距离由margin决定，在一个BFC中，两个相邻的块级盒子的垂直外边距会产生折叠。
   3. 在BFC中，每一个盒子的左外边缘（margin-left）会触碰到容器的左边缘(border-left)（对于从右到左的格式来说，则触碰到右边缘）
   4. 形成了BFC的区域不会与float box重叠
   5. 计算BFC高度时，浮动元素也参与计算

4. BFC 用途

   ```vue
   /* 利用特性5可以解决浮动元素造成的父元素高度塌陷问题： */
   <div class='parent'>
       <div class='float'>浮动元素</div>
   </div>
   
   <style>
       .parent {
           overflow:hidden;
       }
       .float {
           float:left;
       }
   </style>
   ```

   

   ```html
   <!-- 利用特性4可实现左图右文之类的效果 -->
   <template>
       <img src='image.png'>
       <p>我是超长的文字<p>
   </template>
   <style>
       img {
           float:left
       }
       p {
           overflow:hidden
       }
   </style>
   ```

   

### 6：作用域 变量提升 IIFE(立即执行函数)

```js
var a = 10;
(function () {
    console.log(a)
    a = 5
    console.log(window.a)
    var a = 20;
    console.log(a)
})()

// undefined 10 20

var a = 10;
(function () {
    console.log(a)
    a = 5
    console.log(window.a)
    a = 20;
    console.log(a)
})()

// 10 5 20
```

- var 了   块级上下文  局部变量
- 没var ，  上下文查找，往上找



### 7.用 JS 实现一个无限累加的函数 `add`

```js
function add(a) {
    function sum(b) { // 使用闭包
        a = a + b; // 累加
        return sum;
    }
    sum.toString = function() { // 重写toString()方法
        return a;
    }
    return sum; // 返回一个函数
}

add(1); // 1
add(1)(2);  // 3
add(1)(2)(3)； // 6
add(1)(2)(3)(4)； // 10 
```



### 8. 防抖 节流

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



### 9.webSocket

1. http协议有一个缺陷：通信只能由客户端发起。这种单项的请求，导致了如果服务器有连续不断的状态变化，客户端想要获取，就需要使用“轮询”；每隔一段时间，就发起一个询问，了解服务器有没有新的信息。轮询的效率很低，非常浪费资源（必须不停的进行请求连接，或者保持http连接）。因此webSocket应运而生。

2. 特点
   - 最大的特点：服务器可以主动向客户端推送信息，客户端也可以主动向服务器发送信息，是真正的双向平等对话，属于[服务器推送技术](https://en.wikipedia.org/wiki/Push_technology)的一种。
   - 没有同源策略，客户端可以与任意的服务器通信。
   - 协议标识符是 `wx` (如果是加密，`则为wss`)，服务器网址就是URL
   
3. readyState

   ```
   connnecting 0   正在连接
   open 1 连接成功
   closing 2 连接正在关闭
   closed  3 连接已经关闭 或 失败
   ```

4. uniapp 使用

   ```js
   // 建立一个webSocket 连接
   var socketTask = uni.connectSocket({
       url: 'ws://47.104.202.152:8080/websocket/' + 'scout', //仅为示例，并非真实接口地址。
       success: () => {
           uni.onSocketOpen(function(res) {
               console.log('WebSocket连接已打开！');
           });
       },
       fail: () => {
   
       }
   });
   
   // socketTask.onOpen(function(res) {
   // 	console.log('WebSocket连接已打开！');
   // })
   
   socketTask.onMessage(function(res) {
       console.log('收到服务器内容：' + res.data);
   })
   
   
   socketTask.onClose(function(res) {
       console.log('WebSocket 已关闭！');
   })
   
   socketTask.onError(function(res) {
       console.log('WebSocket连接打开失败，请检查！');
   })
   // 
   
   this.scoket = socketTask
   console.log(socketTask);
   ```




### 10. 为什么说HTTPS比HTTP安全？HTTPS是如何保证安全的？

1. `http` 通信过程中存在的问题：
   - 通信使用 明文传输(不加密)，内容可能被窃听
   - 不验证通信方的身份，可能遭遇伪装
   
2. `https`的出现解决了 `http` 的不足， https 是建立在 `SSL` 之上的，其安全性由`SSL`保证
   - SSL(Secure Sockets Layer 安全套接字协议),及其继任者传输层安全（Transport Layer Security，TLS）是为网络通信提供安全及数据完整性的一种安全协议
   - SSL 的实现：
     - 对称加密：采用协商的密钥对数据加密
     - 非对称加密：实现身份认证和密钥协商
     - 摘要算法：验证信息的完整性
     - 数字签名：身份验证
   
3. 混合加密

   - 在`HTTPS`通信过程中，采用的是对称加密+非对称加密，也就是混合加密
   - 具体做法是发送密文的一方使用对方的公钥进行加密处理“对称的密钥”，然后对方用自己的私钥解密拿到“对称的密钥”
   - 举例：服务器将公钥发送给客户端，客户端拿到公钥后，对 密钥 进行公钥加密，再发送给服务器，服务器用私钥解密，得到 客户端的密钥。之后双方使用 对称密钥 进行通信。

4. 摘要算法：

   - 实现完整性的手段主要是摘要算法，也就是常说的散列函数、哈希函数

     可以理解成一种特殊的压缩算法，它能够把任意长度的数据“压缩”成固定长度、而且独一无二的“摘要”字符串，就好像是给这段数据生成了一个数字“指纹”

5. 数字签名

   - CA 对公钥的签名认证要求包括序列号、用途、颁发者、有效时间等等，把这些打成一个包再签名，完整地证明公钥关联的各种信息，形成“数字证书”



### 11.父子组件声明周期

1. 父beforeCreate => 父created => 父beforeMount => 子beforeCreate => 子created =>子beforeMount => 子Mounted=>父Mounted



### Vue声明周期

1. 初始化阶段：为vue实力初始化属性、事件、数据观测等。
2. 模板编译阶段：将模板字符串编译成渲染函数(该阶段在runtime版本中不存在，因为已经编译好了)
3. 挂载阶段：将vue实例挂载到指定的dom上，即将模板渲染到真是dom中。
4. 销毁阶段：解绑指令，移除事件监听器，销毁子实例。



### 12.普通函数 箭头函数的区别* 构造函数

1. 箭头函数没有原型 原型是undefined
2. 箭头函数this指向全局对象 而函数指向引用对象
3. call，apply，bind方法改变不了箭头函数的指向

### 13.let var  const

1. var定义的变量，没有块的概念，可以跨块访问, 不能跨函数访问，存在变量提升，允许在相同作用域内`重复声明同一个变量`的
2. let定义的变量，只能在块作用域里访问，不能跨块访问，也不能跨函数访问
3. const用来定义常量，使用时必须初始化(即必须赋值)，只能在块作用域里访问，而且不能修改（对象属性可以修改）



### 14. typeof,instanceof

1. typeof：可以判断基本数据类型，引用数据类型返回的都是 objcet
2. instanceof：可以判断引用数据类型，
3. 最好的方法是用  `Object.prototype.toString().call(判断的对象)`



### 15.原型链



### 闭包

1. **闭包就是一个函数引用另一个函数内部的变量，因为变量被引用着，所以当另外一个函数执行结束，其相应的执行上下文弹出栈时， 变量并不会被回收，因此可以用来封装一个私有变量。不正当地使用闭包可能会造成内存泄漏。**

   - 全局作用域
   - 函数作用域

2. 一个函数结束

   - 本地执行上下文从执行栈中弹出
   - 本地执行上下文被销毁，

3. 闭包：

   - ```js
     function createCounter() {
         let counter = 0
         const myFunction = function() {
             counter = counter + 1
             return counter
         }
         return myFunction
     }
     const increment = createCounter()
     const c1 = increment()
     const c2 = increment()
     const c3 = increment()
     console.log('example increment', c1, c2, c3)
     ```

   - 函数背着一个小背包 就是闭包。

### 

### 16.Vue的组件data为什么必须是一个函数?

1. `new Vue`是一个单例模式，不会有任何的合并操作，所以根实例不必校验data一定是一个函数。 **组件的data必须是一个函数，是为了防止两个组件的数据产生污染。** 如果都是对象的话，会在合并的时候，指向同一个地址。 而如果是函数的时候，合并的时候调用，会产生两个空间。



### 17.v-for和v-if哪个优先级更高？

1.  v-for 优先级高于 v-if
2. 同时遇到的时候会浪费性能，v-for 会循环所有节点，v-if 再次判断时候，会出现渲染无用节点
3. 更好的解决方法：computed 处理数据，再进行 v-for 循环



### 18.nextTick原理

1. dom更新之后执行的回调，可以用来获取更新后的dom元素
2. 粗浅的认为：数据更新了，在DOM渲染的时候，自动的执行该函数。
3. 原理：Vue是异步执行dom的更新，一旦观察到数据变化，Vue就会开启一个队列，然后把在同一个事件循环 (event loop) 当中观察到数据变化的 watcher 推送进这个队列。（**不理解**)



### 19. vue3 diff  与 vue2的区别 

1. 增加patchFlag静态标识，只对比有静态标识的dom元素
2. 很多文本节点提升 只定义一次，渲染时不需要再次定义，vue2每次都需要重新定义



### 20. 插槽

1. 匿名插槽

   - ```vue
     //父
     <child>
     		<!-- 自定义内容 -->  
     </child>
     
     
     //子
     <templete>
         <slot></slot>
     </templete>
     ```

     

2. 具名插槽

   - ```vue
     //父
     <child>
         <templete v-slot:自定义命名>
     		<!-- 自定义内容 -->        
         </templete>
     </child>
     
     
     //子
     <templete>
         <slot name="自定义命名"></slot>
     </templete>
     ```

     

3. 作用域插槽

   - ```vue
     //子组件
     <div>
     	<!-- 设置默认值：{{user.lastName}}获取 Jun -->
     	<!-- 设置一个 usertext 然后把user绑到设置的 usertext 上 -->
     	<slot name="自定义命名" v-bind:usertext="user">{{user.lastName}}</slot>
     </div>
     
     //定义内容
     data(){
       return{
     	user:{
     	  firstName:"Fan",
     	  lastName:"Jun"
     	}
       }
     }
     
     
     //父
     <div>
       <test v-slot:自定义命名="slotProps">
         {{slotProps.usertext.firstName}}
       </test>
     </div>
     
     ```

     



### 21. v-for中 key的作用

1. 主要作用是给节点作 唯一标识，以便高效的更新虚拟 DOM。
2. 标识，为diff算法利用，方便复用
3. Vue在patch过程中，通过key可以判断两个虚拟节点是否是相同节点。
4. 没有key会导致更新的时候出问题
5. 尽量不要采用索引作为key





### 22. $router 和 $route

1. $router: 路由对象
   - router.push()  replace() go()
2. $route:路由信息
   - route.path  fullpath params query

### 23. 路由守卫

1.  beforeEach(to,from,next)
2. afterEach()



### 24. get 和 post 的区别



### 25.vue的data为什么是函数返回而不是直接一个对象

1. 组件中的 data 如果直接返回的是对象，如果多次引入该组件，映射地址是相同的，则会造成数据互相污染。
2. 如果是函数返回一个对象，每次注册组件都是一个生成一个新的对象，他们的映射地址就不同。



### vue 为什么要使用虚拟Dom

- 虚拟dom是用js对象来描述真实Dom，是对真实Dom的抽象化
- 由于直接操作真实Dom消耗大性能低，但是js层的效率高，所以将真实Dom转化成js对象进行操作，最终通过diff算法比对差异更新Dom （简而言之，实际上就是用`JS`的计算性能来换取操作真实`DOM`所消耗的性能。）
- 虚拟Dom不依赖真实平台环境，可以跨平台



### 27. Vue2 和 Vue3的区别

- vue2 面向对象编程，vue3函数式编程
- 生命周期：
  - vue3：setup,onBeforeMount,onMounted,onBeforeUnMount,onUnMounted,onBeforeUpdat,onUpdated,
- diff 算法
- v-for 和 v-if 的优先级
  - vue3:  v-if > v-for  （报错，此时还没有dom）
  - vue2:  v-for > v-if  (先循环再判断)

























基于vue全家桶和iview框架开发；

利用qrcodejs插件，绘制二维码以及轮询验证，实现商家入驻押金缴纳并跳转

使用vuex存储用户登录的相关信息状态，以及登录、退出等方法



利用flex弹性布局实现页面布局。对公用样式进行抽离。

使用subpackages分包解决小程序项目包超过2M。

对商品介绍页、店铺介绍页面等使用骨架屏，优化用户体验。





暂存

```
let obj = {
			info: {
				name: 'eason',
				blogs: ['webpack', 'babel', 'cache']
			}
		}
		let handler = {
			get(target, key, receiver) {
				console.log('target', target);
				console.log('get', key)
				// 递归创建并返回
				if (typeof target[key] === 'object' && target[key] !== null) {
					return new Proxy(target[key], handler)
				}
				// console.log(Reflect.get(target, key, receiver));
				return Reflect.get(target, key, receiver)
			},
			set(target, key, value, receiver) {
				console.log('set', key, value, target)
				return Reflect.set(target, key, value, receiver)
			}
		}

		let proxy = new Proxy(obj, handler)
		// proxy.info.blogs.push('proxy')
		// proxy.info.name = "luxi"
		// proxy.age = 18
		// console.log(proxy.info.blogs);



		let current = {
			a: 0
		}
		let handler1 = {
			get(target, key, receiver) {
				// 递归创建并返回
				if (typeof target[key] === 'object' && target[key] !== null) {
					return new Proxy(target[key], handler)
				}
				target[key]++
				console.log(target[key]);
				return Reflect.get(target, key, receiver)
			},
			set(target, key, value, receiver) {
				console.log('set', key, value, target)
				return Reflect.set(target, key, value, receiver)
			}
		}
		
		let proxy1 = new Proxy(current, handler1)
		
		// proxy1.a = 2
		console.log(proxy1.a === 1 && proxy1.a === 2 && proxy1.a === 3) // true


		// let current = 0
		// Object.defineProperty(window, 'a', {
		// 	get() {
		// 		current++
		// 		console.log(current)
		// 		return current
		// 	}
		// })
		// console.log(a === 1 && a === 2 && a === 3) // true
```



