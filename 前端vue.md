## 一、Vue3.0环境搭建

- 使用 vite 创建 Vue(3.2.30)项目

```bash
npm install yarn -g
yarn create vite vue3-project --template vue
cd vue3-project // 进入项目根目录
yarn // 安装依赖包
yarn dev // 启动本地服务
```

- 安装 vue-router、vuex全家桶

```bash
yarn add vue-router@latest // v4.0.14
yarn add vuex@latest // v4.0.2
```

- 安装 UI 组件库：在Vue3环境中，一定找支持 Vue3的组件库，那些 Vue2的组件库是无法使用的。

```text
yarn add ant-design-vue@next // v2.2.8
yarn add vite-plugin-components --dev // 支持ant-design-vue按需引入
```

- 支持 ant-design-vue 组件按需引入

```js
# vite.config.ts

import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'
import ViteComponents, { AntDesignVueResolver } from 'vite-plugin-components'

// https://vitejs.dev/config/
export default defineConfig({
  plugins: [
    vue(),
    ViteComponents({
      customComponentResolvers: [AntDesignVueResolver()],
    })
  ]
})
```

- 支持 Sass 样式语法

```bash
yarn add sass // v1.49.9
```

**1、入口文件 main.js**

```js
import { createApp } from 'vue'
import router from './router.ts'
import store from './store'
import App from './App.vue'
// 导入UI样式表
import "ant-design-vue/dist/antd.css"

const app = createApp(App)
// 配置全局属性（这里不能再使用Vue.prototype了）
app.config.globalProperties.$http = ''

app.use(router) // 注册路由系统
app.use(store) // 注册状态管理
// 全局指令
app.directive('highlight', {
  beforeMount(el, binding, vnode) {
    el.style.background = binding.value
  }
})
app.mount('#app') // 挂载
```

**2、Vue-Router (v4) 详解**

- 注意：在vue3环境中，必须要使用vue-router(v4)
- 创建router，使用createRouter()
- 指定路由模式，使用history属性：createWebHashHistory/createWebHistory()
- 路由注册，在mian.js中 app.use(router)
- 如果当前项目严格使用组合式API进行开发，必须使用 useRoute、userRouter等Hooks API进行开发。
- <router-link>已经没有tag属性的，可以用custom和插槽实现自定义。
- <router-view>新增了"插槽"功能，极其强大，参见路由中的伪代码，它在实现<keep-alive>和<transition>动画将变得更简单，还可以Suspense实现Loading。
- 新增了几个组合API：useRoute/useRouter/useLink。
- 查询vue-router(v3)和vue-router(v4)的变化：[https://next.router.vuejs.org/zh/guide/migration/index.html](https://link.zhihu.com/?target=https%3A//next.router.vuejs.org/zh/guide/migration/index.html)

在Vue3环境中编写路由配置，参考代码如下：

```js
import { createRouter, createWebHashHistory } from 'vue-router'

const Home = () => import('@/pages/study/Home.vue')
const Find = () => import('@/pages/study/Find.vue')
const User = () => import('@/pages/study/User.vue')
const Cnode = () => import('@/pages/cnode/index.vue')

export default createRouter({
  history: createWebHashHistory(), // Hash路由
  routes: [
    {
      path: '/home',
      component: Home,
      meta:{transition: 'fade', isAlive: true }
    },
    { path: '/find', component: Find },
    { path: '/user', component: User },
    { path: '/cnode', component: Cnode }
  ]
})
```

**3、Vuex 根store 代码示例**

- 版本：在vue3环境中，必须要使用 vuex(4)
- 注意：在组件中使用 vuex数据时，哪怕是在setup中，也要使用 computed 来访问状态管理中的数据，否则数据不响应。

在Vue3环境中编写 Vuex代码示例如下：

```js
# src/store/index.ts

import { createStore } from 'vuex'
import cnode from './modules/cnode'

export default createStore({
  getters: {},
  modules: { cnode }
})
```

**4、Vuex 子store 代码示例**

```js
# src/store/modules/cnode.ts

import { fetchList } from '@/utils/api'
export default {
  namespaced: true,
  state: {
    msg: 'cnode',
    list: [],
    cates: [
      { id: 1, tab: '', label: '全部' },
      { id: 2, tab: 'good', label: '精华' },
      { id: 3, tab: 'share', label: '分享' },
      { id: 4, tab: 'ask', label: '问答' },
      { id: 5, tab: 'job', label: '招聘' }
    ]
  },
  mutations: {
    updateList (state, payload) {
      state.list = payload
    }
  },
  actions: {
    getList ({commit}, params) {
      fetchList(params).then(list=>{
        console.log('文章列表', list)
        commit('updateList', list)
      })
    }
  }
}
```

**5、App 根组件代码示例**

```html
<template>
  <!-- 路由菜单 -->
  <router-link to='/home'>首页</router-link>
  <router-link to='/find'>发现</router-link>
  <router-link to='/user'>我们</router-link>
  <!-- 视图容器 -->
  <router-view />
</template>

<script setup></script>

<style lang='scss'>
  html, body { padding:0; margin:0; }
</style>

<style lang='scss' scoped>
  a { display:inline-block; padding:5px 15px; }
</style>
```



## **二、组合API 详解**



**为什么要使用setup组合?**

- Vue3 中新增的 setup，目的是为了解决 Vue2 中“数据和业务逻辑不分离”的问题。

**Vue3中使用 setup 是如何解决这一问题的呢？**

- 第1步: 用setup组合API 替换 vue2 中的data/computed/watch/methods等选项；
- 第2步: 把setup中相关联的功能封装成一个个可独立可维护的hooks。

**1、ref**

- 作用：一般用于定义基本数据类型数据，比如 String / Boolean / Number等。
- 背后：ref 的背后是使用 reactive 来实现的响应式.
- 语法：const x = ref(100)
- 访问：在 setup 中使用 .value 来访问。

```html
<template>
  <h1 v-text='num'></h1>
  <!-- 在视图模板中，无须.value来访问 -->
  <button @click='num--'>自减</button>
  <!-- 在setup中，要使用.value来访问 -->
  <button @click='add'>自增</button>
</template>

<script setup>
  import { ref } from 'vue'
  const num = ref(100)
  const add = () => num.value++
</script>
```

**2、isRef**

- 作用：判断一个变量是否为一个 ref 对象。
- 语法：const bol = isRef(x)

```html
<template>
  <h1 v-text='hello'></h1>
</template>

<script setup>
  import { ref, isRef, reactive } from 'vue'
  const hello = ref('Hello')
  const world = reactive('World')
  console.log(isRef(hello))  // true
  console.log(isRef(world))  // false
</script>
```

**3、unref**

- 作用：用于返回一个值，如果访问的是 ref变量，就返回其 .value值；如果不是 ref变量，就直接返回。
- 语法：const x = unref(y)

```html
<template>
  <h1 v-text='hello'></h1>
</template>

<script setup>
  import { ref, unref } from 'vue'
  const hello = ref('Hello')
  const world = 'World'
  console.log(unref(hello))  // 'Hello'
  console.log(unref(world))  // 'World'
</script>
```

**4、customRef**

作用：自定义ref对象，把ref对象改写成get/set，进一步可以为它们添加 track/trigger。

```html
<template>
  <h1 v-text='num'></h1>
  <button @click='num++'>自增</button>
</template>

<script setup>
  import { customRef, isRef } from 'vue'
  const num = customRef((track, trigger)=>{
    let value = 100
    return {
      get () {
        track()
        return value
      },
      set (newVal) {
        value = newVal
        trigger()
      }
    }
  })
  console.log(isRef(num)) // true
</script>
```

**5、toRef**

- 作用：把一个 reactive对象中的某个属性变成 ref 变量。
- 语法：const x = toRef(reactive(obj), 'key') // x.value

```html
<template>
  <h1 v-text='age'></h1>
</template>

<script setup>
  import { toRef, reactive, isRef } from 'vue'
  let user = { name:'张三', age:10 }

  let age = toRef(reactive(user), 'age')
  console.log(isRef(age))  // true
</script>
```

**6、toRefs**

- 作用：把一个reactive响应式对象变成ref变量。
- 语法：const obj1 = toRefs(reactive(obj))
- 应用：在子组件中接收父组件传递过来的 props时，使用 toRefs把它变成响应式的。

```html
<template>
  <h1 v-text='info.age'></h1>
</template>

<script setup>
  import { toRefs, reactive, isRef } from 'vue'
  let user = { name:'张三', age:10 }
  let info = toRefs(reactive(user))
  
  console.log(isRef(info.age))  // true
  console.log(isRef(info.name))  // true
  console.log(isRef(info))  // true
</script>
```

**7、shallowRef**

- 作用：对复杂层级的对象，只将其第一层变成 ref 响应。 (性能优化)
- 语法：const x = shallowRef({a:{b:{c:1}}, d:2}) 如此a、b、c、d变化都不会自动更新，需要借助 triggerRef 来强制更新。

```html
<template>
  <h1 v-text='info.a.b.c'></h1>
  <button @click='changeC'>更新[c]属性</button>

  <h1 v-text='info.d'></h1>
  <button @click='changeD'>更新[d]属性</button>
</template>

<script setup>
  import { shallowRef, triggerRef, isRef } from 'vue'

  let info = shallowRef({a:{b:{c:1}}, d:2})

  console.log(isRef(info.value.a.b.c)) // false
  console.log(isRef(info)) // true
  console.log(isRef(info.a)) // false
  console.log(isRef(info.d)) // false

  const changeC = () => {
    info.value.a.b.c++
    triggerRef(info) // 强制渲染更新
  }

  const changeD = () => {
    info.value.d++
    triggerRef(info) // 强制渲染更新
  }
</script>
```

**8、triggerRef**

- 作用：强制更新一个 shallowRef对象的渲染。
- 语法：triggerRef(shallowRef对象)
- 参考代码：见上例。

**9、reactive**

- 作用：定义响应式变量，一般用于定义引用数据类型。如果是基本数据类型，建议使用ref来定义。
- 语法：const info = reactive([] | {})

```html
<template>
  <div v-for='(item,idx) in list'>
    <span v-text='idx'></span> -
    <span v-text='item.id'></span> -
    <span v-text='item.label'></span> -
    <span v-text='item.tab'></span>
  </div>
  <button @click='addRow'>添加一行</button>
</template>

<script setup>
  import { reactive } from 'vue'
  const list = reactive([
    { id:1, tab:'good', label:'精华' },
    { id:2, tab:'ask', label:'问答' },
    { id:3, tab:'job', label:'招聘' },
    { id:4, tab:'share', label:'分享' }
  ])
  const addRow = () => {
    list.push({id:Date.now(),tab:'test',label:'测试'})
  }
</script>
```

**10、readonly**

- 作用：把一个对象，变成只读的。
- 语法：const rs = readonly(ref对象 | reactive对象 | 普通对象)

```html
<template>
  <h1 v-text='info.foo'></h1>
  <button @click='change'>改变</button>
</template>

<script setup>
  import { reactive, readonly } from 'vue'
  const info = readonly(reactive({bar:1, foo:2}))
  const change = () => {
    info.foo++  // target is readonly
  }
</script>
```

**11、isReadonly**

- 作用: 判断一个变量是不是只读的。
- 语法：const bol = isReadonly(变量)

```html
<script setup>
  import { reactive, readonly, isReadonly } from 'vue'

  const info = readonly(reactive({bar:1, foo:2}))
  console.log(isReadonly(info)) // true

  const user = readonly({name:'张三', age:10})
  console.log(isReadonly(user)) // true
</script>
```

**12、isReactive**

- 作用：判断一变量是不是 reactive的。
- 注意：被 readonly代理过的 reactive变量，调用 isReactive 也是返回 true的。

```html
<script setup>
  import { reactive, readonly, isReactive } from 'vue'

  const user = reactive({name:'张三', age:10})
  const info = readonly(reactive({bar:1, foo:2}))

  console.log(isReactive(info)) // true
  console.log(isReactive(user)) // true
</script>
```

**13、isProxy**

作用：判断一个变量是不是 readonly 或 reactive的。

```html
<script setup>
  import { reactive, readonly, ref, isProxy } from 'vue'

  const user = readonly({name:'张三', age:10})
  const info = reactive({bar:1, foo:2})
  const num = ref(100)

  console.log(isProxy(info)) // true
  console.log(isProxy(user)) // true
  console.log(isProxy(num))  // false
</script>
```

**14、toRaw**

- 作用：得到返回 reactive变量或 readonly变量的"原始对象"。
- 语法:：const raw = toRaw(reactive变量或readonly变量)
- 说明：reactive(obj)、readonly(obj) 和 obj 之间是一种代理关系，并且它们之间是一种浅拷贝的关系。obj 变化，会导致reactive(obj) 同步变化，反之一样。

```html
<script setup>
  import { reactive, readonly, toRaw } from 'vue'

  const uu = {name:'张三', age:10}
  const user = readonly(uu)
  console.log(uu === user) // false
  console.log(uu === toRaw(user)) // true

  const ii = {bar:1, foo:2}
  const info = reactive(ii)
  console.log(ii === info) // false
  console.log(ii === toRaw(info)) // true
</script>
```

**15、markRaw**

- 作用：把一个普通对象标记成"永久原始"，从此将无法再变成proxy了。
- 语法：const raw = markRaw({a,b})

```html
<script setup>
  import { reactive, readonly, markRaw, isProxy } from 'vue'

  const user = markRaw({name:'张三', age:10})
  const u1 = readonly(user)  // 无法再代理了
  const u2 = reactive(user)  // 无法再代理了

  console.log(isProxy(u1))  // false
  console.log(isProxy(u2))  // false
</script>
```

**16、shallowReactive**

- 作用：定义一个reactive变量，只对它的第一层进行Proxy,，所以只有第一层变化时视图才更新。
- 语法：const obj = shallowReactive({a:{b:9}})

```html
<template>
  <h1 v-text='info.a.b.c'></h1>
  <h1 v-text='info.d'></h1>
  <button @click='change'>改变</button>
</template>

<script setup>
  import { shallowReactive, isProxy } from 'vue'
  const info = shallowReactive({a:{b:{c:1}}, d:2})

  const change = () => {
    info.d++  // 只改变d，视图自动更新
    info.a.b.c++ // 只改变c，视图不会更新
    // 同时改变c和d，二者都更新
  }

  console.log(isProxy(info))  // true
  console.log(isProxy(info.d))  // false
</script>
```

**17、shallowReadonly**

- 作用：定义一个reactive变量，只有第一层是只读的。
- 语法：const obj = shallowReadonly({a:{b:9}})

```html
<template>
  <h1 v-text='info.a.b.c'></h1>
  <h1 v-text='info.d'></h1>
  <button @click='change'>改变</button>
</template>

<script setup>
  import { reactive, shallowReadonly, isReadonly } from 'vue'
  const info = shallowReadonly(reactive({a:{b:{c:1}}, d:2}))

  const change = () => {
    info.d++  // d是读的，改不了
    info.a.b.c++ // 可以正常修改，视图自动更新
  }
  console.log(isReadonly(info))  // true
  console.log(isReadonly(info.d))  // false
</script>
```

**18、computed**

- 作用：对响应式变量进行缓存计算。
- 语法：const c = computed(fn / {get,set})

```html
<template>
  <div class='page'>
    <span
      v-for='p in pageArr'
      v-text='p'
      @click='page=p'
      :class='{"on":p===page}'
    >
    </span>
  </div>

  <!-- 在v-model上使用computed计算属性 -->
  <input v-model.trim='text' /><br>
  你的名字是：<span v-text='name'></span>
</template>

<script setup>
  import { ref, computed } from 'vue'
  const page = ref(1)
  const pageArr = computed(()=>{
    const p = page.value
    return p>3 ? [p-2,p-1,p,p+1,p+2] : [1,2,3,4,5]
  })

  const name = ref('')
  const text = computed({
    get () { return name.value.split('-').join('') },
    // 支持计算属性的setter功能
    set (val) {
      name.value = val.split('').join('-')
    }
  })
</script>

<style lang='scss' scoped>
  .page {
    &>span {
      display:inline-block; padding:5px 15px;
      border:1px solid #eee; cursor:pointer;
    }
    &>span.on { color:red; }
  }
</style>
```

**19、watch**

- 作用：用于监听响应式变量的变化，组件初始化时，它不执行。
- 语法：const stop = watch(x, (new,old)=>{})，调用stop() 可以停止监听。
- 语法：const stop = watch([x,y], ([newX,newY],[oldX,oldY])=>{})，调用stop()可以停止监听。

```html
<template>
  <h1 v-text='num'></h1>
  <h1 v-text='usr.age'></h1>
  <button @click='change'>改变</button>
  <button @click='stopAll'>停止监听</button>
</template>

<script setup>
  import { ref, reactive, watch, computed } from 'vue'

  // watch监听ref变量、reactive变量的变化
  const num = ref(1)
  const usr = reactive({name:'张三',age:1})
  const change = () => {
    num.value++
    usr.age++
  }
  const stop1 = watch([num,usr], ([newNum,newUsr],[oldNum,oldUsr]) => {
    // 对ref变量，newNum是新值，oldNum是旧值
    console.log('num', newNum === oldNum) // false
    // 对reactive变量，newUsr和oldUsr相等，都是新值
    console.log('usr', newUsr === oldUsr) // true
  })

  // watch还可以监听计算属性的变化
  const total = computed(()=>num.value*100)
  const stop2 = watch(total, (newTotal, oldTotal) => {
    console.log('total', newTotal === oldTotal) // false
  })

  // 停止watch监听
  const stopAll = () => { stop1(); stop2() }
</script>
```

**20、watchEffect**

- 作用：相当于是 react中的 useEffect()，用于执行各种副作用。
- 语法：const stop = watchEffect(fn)，默认其 flush:'pre'，前置执行的副作用。
- watchPostEffect，等价于 watchEffect(fn, {flush:'post'})，后置执行的副作用。
- watchSyncEffect，等价于 watchEffect(fn, {flush:'sync'})，同步执行的副作用。
- 特点：watchEffect 会自动收集其内部响应式依赖，当响应式依赖发变化时，这个watchEffect将再次执行，直到你手动 stop() 掉它。

```html
<template>
  <h1 v-text='num'></h1>
  <button @click='stopAll'>停止掉所有的副作用</button>
</template>

<script setup>
  import { ref, watchEffect } from 'vue'
  let num = ref(0)

  // 等价于 watchPostEffect
  const stop1 = watchEffect(()=>{
    // 在这里你用到了 num.value
    // 那么当num变化时，当前副作用将再次执行
    // 直到stop1()被调用后，当前副作用才死掉
    console.log('---effect post', num.value)
  }, { flush:'post'} )

  // 等价于 watchSyncEffect
  const stop2 = watchEffect(()=>{
    // 在这里你用到了 num.value
    // 那么当num变化时，当前副作用将再次执行
    // 直到stop2()被调用后，当前副作用才死掉
    console.log('---effect sync', num.value)
  }, { flush:'sync'})

  const stop3 = watchEffect(()=>{
    // 如果在这里用到了 num.value
    // 你必须在定时器中stop3()，否则定时器会越跑越快！
    // console.log('---effect pre', num.value)
    setInterval(()=>{
      num.value++
      // stop3()
    }, 1000)
  })

  const stopAll = () => {
    stop1()
    stop2()
    stop3()
  }
</script>
```

**21、生命周期钩子**

- 选项式的 beforeCreate、created，被setup替代了。setup表示组件被创建之前、props被解析之后执行，它是组合式 API 的入口。
- 选项式的 beforeDestroy、destroyed 被更名为 beforeUnmount、unmounted。
- 新增了两个选项式的生命周期 renderTracked、renderTriggered，它们只在开发环境有用，常用于调试。
- 在使用 setup组合时，不建议使用选项式的生命周期，建议使用 on* 系列 hooks生命周期。

```html
<template>
  <h1 v-text='num'></h1>
  <button @click='num++'>自增</button>
</template>

<script setup>
  import {
    ref, onBeforeMount, onMounted,
    onBeforeUpdate, onUpdated,
    onBeforeUnmount, onUnmounted,
    onRenderTracked, onRenderTriggered,
    onActivated, onDeactivated,
    onErrorCaptured
  } from 'vue'

  console.log('---setup')
  const num = ref(100)
  // 挂载阶段
  onBeforeMount(()=>console.log('---开始挂载'))
  onRenderTracked(()=>console.log('---跟踪'))
  onMounted(()=>console.log('---挂载完成'))

  // 更新阶段
  onRenderTriggered(()=>console.log('---触发'))
  onBeforeUpdate(()=>console.log('---开始更新'))
  onUpdated(()=>console.log('---更新完成'))

  // 销毁阶段
  onBeforeUnmount(()=>console.log('---开始销毁'))
  onUnmounted(()=>console.log('---销毁完成'))

  // 与动态组件有关
  onActivated(()=>console.log('---激活'))
  onDeactivated(()=>console.log('---休眠'))
  
  // 异常捕获
  onErrorCaptured(()=>console.log('---错误捕获'))
</script>
```

**22、provide / inject**

- 作用：在组件树中自上而下地传递数据.
- 语法：provide('key', value)
- 语法：const value = inject('key', '默认值')

```html
# App.vue

<script setup>
  import { ref, provide } from 'vue'
  const msg = ref('Hello World')
  // 向组件树中注入数据
  provide('msg', msg)
</script>

# Home.vue

<template>
  <h1 v-text='msg'></h1>
</template>
<script setup>
  import { inject } from 'vue'
  // 消费组件树中的数据，第二参数为默认值
  const msg = inject('msg', 'Hello Vue')
</script>
```

**23、getCurrentInstance**

- 作用：用于访问内部组件实例。请不要把它当作在组合式 API 中获取 this 的替代方案来使用。
- 语法：const app = getCurrentInstance()
- 场景：常用于访问 app.config.globalProperties 上的全局数据。

```html
<script setup>
  import { getCurrentInstance } from 'vue'
  const app = getCurrentInstance()
  // 全局数据，是不具备响应式的。
  const global = app.appContext.config.globalProperties
  console.log('app', app)
  console.log('全局数据', global)
</script>
```

**24、关于setup代码范式（最佳实践）**

- 只使用 setup 及组合API，不要再使用vue选项了。
- 有必要封装 hooks时，建议把功能封装成hooks，以便于代码的可维护性。
- 能用 vite就尽量使用vite，能用ts 就尽量使用ts。



## 三、Vue3 组件通信



**1、第一个组件（品类组件）**

- 使用 setup 及组件API ，自定义封装 Vue3 组件。
- defineProps 用于接收父组件传递过来的自定义属性。
- defineEmits 用于声明父组件传递过来的自定义事件。
- useStore，配合 computed 实现访问 Vuex中的状态数据。

```html
# 文件名 CnCate.vue
 
<template>
  <div class='cates'>
    <span
      v-for='item in cates'
      v-text='item.label'
      :class='{"on": tab===item.tab}'
      @click='change(item.tab)'
    >
    </span>
  </div>
</template>

<script setup>
  import { defineProps, defineEmits, computed } from 'vue'
  import { useStore } from 'vuex'
  // 接收自定义属性
  const props = defineProps({
    tab: { type: String, default: '' }
  })
  const emit = defineEmits(['update:tab'])
  // 从vuex中访问cates数据
  const store = useStore()
  const cates = computed(()=>store.state.cnode.cates)

  const change = (tab) => {
    emit('update:tab', tab) // 向父组件回传数据
  }
</script>

<style lang="scss" scoped>
.cates {
  padding: 5px 20px;
  background-color: rgb(246, 246, 246);
}
.cates span {
  display: inline-block; height: 24px;
  line-height: 24px; margin-right: 25px;
  color: rgb(128, 189, 1); font-size: 14px;
  padding: 0 10px; cursor: pointer;
}
.cates span.on {
  background-color: rgb(128, 189, 1);
  color: white; border-radius: 3px;
}
</style>
```



**2、第二个组件（分页组件）**

- 使用 toRefs 把 props 变成响应式的。在Vue3中，默认情况下 props是不具备响应式的，即父组件中的数据更新了，在子组件中却是不更新的。
- 使用 computed 实现动态页码结构的变化。
- defineProps、defineEmits，分别用于接收父组件传递过来的自定义属性、自定义事件。

```html
# 文件名 CnPage.vue
 
<template>
  <div class='pages'>
    <span @click='prev'>&lt;&lt;</span>
    <span v-if='page>3'>...</span>
    <span
      v-for='i in pages'
      v-text='i'
      :class='{"on":i===page}'
      @click='emit("update:page", i)'
    >
    </span>
    <span>...</span>
    <span @click='emit("update:page", page+1)'>>></span>
  </div>
</template>

<script setup>
  import { defineProps, defineEmits, computed, toRefs } from 'vue'
  let props = defineProps({
    page: { type: Number, default: 1 }
  })
  const { page } = toRefs(props)
  const emit = defineEmits(['update:page'])
  const pages = computed(()=>{
    // 1  1 2 3 4 5 ...
    // 2  1 2 3 4 5 ...
    // 3  1 2 3 4 5 ...
    // 4  ... 2 3 4 5 6 ...
    // n  ... n-2 n-1 n n+1 n+2 ...
    const v = page.value
    return v<=3 ? [1,2,3,4,5] : [v-2,v-1,v,v+1,v+2]
  })
  const prev = () => {
    if (page.value===1) alert('已经是第一页了')
    else emit('update:page', page.value-1)
  }
</script>

<style lang="scss" scoped>
.pages {
  line-height: 50px; text-align: right;
}
.pages>span {
  cursor: pointer; display: inline-block;
  width: 34px; height: 30px; margin: 0;
  line-height: 30px; text-align: center;
  font-size: 12px; border: 1px solid #ccc;
}
.pages>span.on {
  background: rgb(128, 189, 1); color: white;
}
</style>
```

**3、在父级组件中使用 自定义组件**

- v-model:tab='tab' 是 :tab 和 @update:tab 的语法糖简写；
- v-model:page='page' 是 :page 和 @update:page 的语法糖简写；
- 使用 watch 监听品类和页面的变化，然后触发调接口获取新数据。

```html
# 文件名 Cnode.vue

<template>
  <div class='app'>
    <!-- <CnCate :tab='tab' @update:tab='tab=$event' /> -->
    <CnCate v-model:tab='tab' />
    <!-- <CnPage :page='page' @update:page='page=$event' /> -->
    <CnPage v-model:page='page' />
  </div>
</template>

<script setup>
  import { ref, watch } from 'vue'
  import CnCate from './components/CnCate.vue'
  import CnPage from './components/CnPage.vue'
  const tab = ref('')
  const page = ref(1)
  const stop = watch([tab, page], ()=>{
    console.log('当品类或页码变化时，调接口')
  })
</script>
```

## 四、Hooks 封装

**1、为什么要封装 Hooks ？**

我们都知道，在Vue2中，在同一个.vue组件中，当 data、methods、computed、watch 的体量较大时，代码将变得臃肿。为了解决代码臃肿问题，我们除了拆分组件外，别无它法。

在Vue3中，同样存在这样的问题：当我们的组件开始变得更大时，逻辑关注点将越来越多，这会导致组件难以阅读和理解。但是，在Vue3中，我们除了可以拆分组件，还可以使用 Hooks封装来解决这一问题。

所谓 Hooks封装，就是把不同的逻辑关注点抽离出来，以达到业务逻辑的独立性。这一思路，也是Vue3 对比Vue2的最大亮点之一。

**2、如何封装 Hooks 呢？**

在 setup 组合的开发模式下，把具体某个业务功能所用到的 ref、reactive、watch、computed、watchEffect 等，提取到一个以 use* 开头的自定义函数中去。

封装成 use* 开头的Hooks函数，不仅可以享受到封装带来的便利性，还有利于代码逻辑的复用。Hooks函数的另一个特点是，被复用时可以保持作用域的独立性，即，同一个Hooks函数被多次复用，彼此是不干扰的。

**3、在哪些情况下需要封装 Hooks呢？**

我总结了两种场景：一种是功能类Hooks，即为了逻辑复用的封装；另一种是业务类Hooks，即为了逻辑解耦的封装。下面我给两组代码，说明这两种使用场景。

**4、示例：功能类 Hooks封装**

```js
import { computed } from 'vue'
import { useRoute } from 'vue-router'
import { useStore } from 'vuex'

// 返回路由常用信息
export function useLocation() {
  const route = useRoute()
  const pathname = route.fullPath
  return { pathname }
}

// 用于方便地访问Vuex数据
export function useSelector(fn) {
  const store = useStore()
  // Vuex数据要用computed包裹，处理响应式问题
  return computed(()=>fn(store.state))
}

// 用于派发actions的
export function useDispatch() {
  const store = useStore()
  return store.dispatch
}
```

**5、示例：业务类 Hooks封装**

```js
import { ref, computed, watch, watchEffect } from 'vue'
import { useDispatch, useSelector } from '@/hooks'

// 业务逻辑封装
export function useCnode() {

  let tab = ref('')  // tab.value
  let page = ref(1)  // page.value

  const dispatch = useDispatch()
  // 使用 store数据
  const cates = useSelector(state=>state.cnode.cates)
  const list = useSelector(state=>state.cnode.list)

  // 用于处理list列表数据
  const newList = computed(()=>{
    const result = []
    list.value.forEach(ele1=>{
      const cate = cates.value.find(ele2=>ele2.tab===ele1.tab)
      ele1['label'] = ele1.top ? '置顶' : ( ele1.good ? '精华' : (cate?.label||'问答'))
      ele1['first'] = tab.value===''
      result.push(ele1)
    })
    return result
  })

  // 相当于react中useEffect(fn, [])
  // watchEffect，它可以自动收集依赖项
  watchEffect(()=>{
    dispatch('cnode/getList',{tab:tab.value,limit:5,page:page.value})
  })

  // 当品类发生变化时，页码重置为第一页
  watch(tab, ()=>page.value=1)

  return [tab, page, newList]
}
```

最后想说的是，不能为了封装Hooks而封装。要看具备场景：是否有复用的价值？是否有利于逻辑的分离？是否有助提升代码的可阅读性和可维护性？

## 五、Vue3 新语法细节

1、在Vue2中，v-for 和 ref 同时使用，这会自动收集 $refs。当存在嵌套的v-for时，这种行为会变得不明确且效率低下。在Vue3中，v-for 和 ref 同时使用，这不再自动收集$refs。我们可以手动封装收集 ref 对象的方法，将其绑定在 ref 属性上。

```html
<template>
  <div class='grid' v-for="i in 5" :ref='setRef'>
    <span v-for='j in 5' v-text='(i-1)*5+j' :ref='setRef'>
    </span>
  </div>
</template>

<script setup>
  import { onMounted } from 'vue'
  // 用于收集ref对象的数组
  const refs = []
  // 定义手动收集ref的方法
  const setRef = el => {
    if (el) refs.push(el)
  }
  onMounted(()=>console.log('refs', refs))
</script>

<style lang='scss' scoped>
  .grid {
    width: 250px; height: 50px; display: flex;
    text-align: center; line-height: 50px;
    &>span { flex: 1; border: 1px solid #ccc; }
  }
</style>
```

2、在Vue3中，使用 defineAsyncComponent 可以异步地加载组件。需要注意的是，这种异步组件是不能用在Vue-Router的路由懒加载中。

```html
<script setup>
  import { defineAsyncComponent } from 'vue'
  // 异步加载组件
  const AsyncChild = defineAsyncComponent({
    loader: ()=>import('./components/Child.vue'),
    delay: 200,
    timeout: 3000
  })
</script>
```

3、Vue3.0中的 $attrs，包含了父组件传递过来的所有属性，包括 class 和 style 。在Vue2中，$attrs 是接到不到 class 和 style 的。在 setup 组件中，使用 useAttrs() 访问；在非 setup组件中，使用 this.$attrs /setupCtx.attrs 来访问。

```html
<script setup>
  // 在非setup组件中，使用this.$attrs/setupCtx.attrs
  import { useAttrs } from 'vue'
  const attrs = useAttrs()
  // 能够成功访问到class和style
  console.log('attrs', attrs) 
</script>
```

4、Vue3中，移除了 $children 属性，要想访问子组件只能使用 ref 来实现了。在Vue2中，我们使用 $children 可以方便地访问到子组件，在组件树中“肆意”穿梭。

5、Vue3中，使用 app.directive() 来定义全局指令，并且定义指令时的钩子函数们也发生了若干变化。

```js
app.directive('highlight', {
  // v3中新增的
  created() {},
  // 相当于v2中的 bind()
  beforeMount(el, binding, vnode, prevVnode) {
    el.style.background = binding.value
  },
  // 相当于v2中的 inserted()
  mounted() {},
  // v3中新增的
  beforeUpdate() {},
  // 相当于v2中的 update()+componentUpdated()
  updated() {},
  // v3中新增的
  beforeUnmount() {},
  // 相当于v2中的 unbind()
  unmounted() {}
})
```

6、data 选项，只支持工厂函数的写法，不再支持对象的写法了。在Vue2中，创建 new Vue({ data }) 时，是可以写成对象语法的。

```html
<script>
  import { createApp } from 'vue'

  createApp({
    data() {
      return {
        msg: 'Hello World'
      }
    }
  }).mount('#app')
</script>
```

7、Vue3中新增了 emits 选项。在非<script setup>写法中，使用 emits选项 接收父组件传递过来的自定义，使用 ctx.emit() / this.$emit() 来触发事件。在<script setup>中，使用 defineEmits 来接收自定义事件，使用 defineProps 来接收自定义事件。

```html
<template>
  <h1 v-text='count' @click='emit("update:count", count+1)'></h1>
</template>

<script setup>
  import { defineProps, defineEmits } from 'vue'
  // 接收父组件传递过来的自定义属性
  const props = defineProps({
    count: { type: Number, default: 100 }
  })
  // 接收父组件传递过来的自定义事件
  // emit 相当于 vue2中的 this.$emit()
  const emit = defineEmits(['change','update:count'])
</script>
```

8、Vue3中 移除了 $on / $off / $once 这三个事件 API，只保留了 $emit 。

9、Vue3中，移除了全局过滤器（Vue.filter）、移除了局部过滤器 filters选项。取而代之，你可以封装自定义函数或使用 computed 计算属性来处理数据。

10、Vue3 现在正式支持了多根节点的组件，也就是片段，类似 React 中的 Fragment。使用片段的好处是，当我们要在 template 中添加多个节点时，没必要在外层套一个 div 了，套一层 div 这会导致多了一层 DOM结构。可见，片段 可以减少没有必要的 DOM 嵌套。

```html
<template>
  <header>...</header>
  <main>...</main>
  <footer>...</footer>
</template>
```

11、函数式组件的变化：在Vue2中，要使用 functional 选项来支持函数式组件的封装。在Vue3中，函数式组件可以直接用普通函数进行创建。如果你在 vite 环境中安装了 `@vitejs/plugin-vue-jsx` 插件来支持 JSX语法，那么定义函数式组件就更加方便了。

```js
# Counter.tsx

export default (props, ctx) => {
  // props是父组件传递过来的属性
  // ctx 中有 attrs, emit, slots
  const { value, onChange } = props

  return (
    <>
      <h1>函数式组件</h1>
      <h1 onClick={()=>onChange(value+1)}>
        { value }
      </h1>
    </>
  )
}
```

12、Vue2中的Vue构造函数，在Vue3中已经不能再使用了。所以Vue构造函数上的静态方法、静态属性，比如 Vue.use/Vue.mixin/Vue.prototype 等都不能使用了。在Vue3中新增了一套实例方法来代替，比如 app.use()等。

```js
import { createApp } from 'vue'
import router from './router'
import store from './store'
import App from './App.vue'

const app = createApp(App)

// 相当于 v2中的 Vue.prototype
app.config.globalProperties.$http = ''
// 等价于 v2中的 Vue.use
app.use(router) // 注册路由系统
app.use(store)  // 注册状态管理
```

13、在Vue3中，使用 getCurrentInstance 访问内部组件实例，进而可以获取到 app.config 上的全局数据，比如 $route、$router、$store 和自定义数据等。这个 API 只能在 setup 或 生命周期钩子 中调用。

```html
<script>
  // 把使用全局数据的功能封装成Hooks
  import { getCurrentInstance } from 'vue'
  function useGlobal (key) {
    return getCurrentInstance().appContext.config.globalProperties[key]
  }
</script>

<script setup>
  import { getCurrentInstance } from 'vue'
  const global = getCurrentInstance().appContext.config.globalProperties
  // 得到 $route、$router、$store、$http ...

  使用自定义Hooks方法访问全局数据
  const $store = useGlobal('$store')
  console.log('store', $store)
</script>
```

14、我们已经知道，使用 provide 和 inject 这两个组合 API 可以组件树中传递数据。除此之外，我们还可以应用级别的 app.provide() 来注入全局数据。在编写插件时使用 app.provide() 尤其有用，可以替代app.config.globalProperties。

```js
# main.ts

const app = createApp(App)
app.provide('global', {
  msg: 'Hello World',
  foo: [1,2,3,4]
})

# 在组件中使用

<script setup>
  import { inject } from 'vue'
  const global = inject('global')
</script>
```

15、在Vue2中，Vue.nextTick() / this.$nextTick 不能支持 Webpack 的 Tree-Shaking 功能的。在 Vue3 中的 nextTick ，考虑到了对 Tree-Shaking 的支持。

```html
<template>
  <div v-html='content'></div>
</template>

<script setup>
  import { ref, watchPostEffect, nextTick } from 'vue'
  const content = ref('')
  watchPostEffect(()=>{
    content.value = `<div id='box'>动态HTML字符串</div>`
    // 在nextTick中访问并操作DOM
    nextTick(()=>{
      const box = document.getElementById('box')
      box.style.color = 'red'
      box.style.textAlign = 'center'
    })
  })
</script>
```

16、Vue3中，对于 `v-if/v-else/v-else-if`的各分支项，无须再手动绑定 key了， Vue3会自动生成唯一的key。因此，在使用过渡动画 <transition>对多个节点进行显示隐藏时，也无须手动加 key了。

```html
<template>
  <!-- 使用<teleport>组件，把animate.css样式插入到head标签中去 -->
  <teleport to='head'>
    <link
      rel="stylesheet"
      href="https://cdn.jsdelivr.net/npm/animate.css@4.1.1/animate.css"
    />
  </teleport>
  <!-- 使用<transition>过渡动画，无须加key了 -->
  <transition
    enter-active-class='animate__animated animate__zoomIn'
    leave-active-class='animate__animated animate__zoomOutUp'
    mode='out-in'
  >
    <h1 v-if='bol'>不负当下</h1>
    <h1 v-else>不畏未来</h1>
  </transition>
  <button @click='bol=!bol'>Toggle</button>
</template>

<script setup>
  import { ref} from 'vue'
  const bol = ref(true)
</script>
```

17、在Vue2中，使用 Vue.config.keyCodes 可以修改键盘码，这在Vue3 中已经淘汰了。

18、Vue3中，$listeners 被移除了。因此我们无法再使用 $listeners 来访问、调用父组件给的自定义事件了。

19、在Vue2中，根组件挂载 DOM时，可以使用 el 选项、也可以使用 $mount()。但，在 Vue3中只能使用 $mount() 来挂载了。并且，在 Vue 3中，被渲染的应用会作为子元素插入到 <div id='app'> 中，进而替换掉它的innerHTML。

20、在Vue2中，使用 propsData 选项，可以实现在 new Vue() 时向根组件传递 props 数据。在Vue3中，propsData 选项 被淘汰了。替代方案是：使用createApp的第二个参数，在 app实例创建时向根组件传入 props数据。

```js
# main.ts 

import { createApp } from 'vue'
import App from './App.vue'
// 使用第二参数，向App传递自定义属性
const app = createApp(App, { name:'vue3' })
app.mount('#app') // 挂载

# App.vue
<script setup>
  import { defineProps } from 'vue'
  // 接收 createApp() 传递过来的自定义属性
  const props = defineProps({
    name: { type: String, default: '' }
  })
  console.log('app props', props)
</script>
```

21、在Vue2中，组件有一个 render 选项（它本质上是一个渲染函数，这个渲染函数的形参是 h 函数），h 函数相当于 React 中的 createElement()。在Vue3中，render 函数选项发生了变化：它的形参不再是 h 函数了。h 函数变成了一个全局 API，须导入后才能使用。

```js
import { createApp, h } from 'vue'
import App from './App.vue'

const app = createApp({
  render () {
    return h(App)
  }
}, { name: 'vue3'} )
app.$mount('#app')
```

22、Vue3中新增了实验性的内置组件 <suspense>，它类似 React.Suspense 一样，用于给异步组件加载时，指定 Loading指示器。需要注意的是，这个新特征尚未正式发布，其 API 可能随时会发生变动。

```html
<template>
  <suspense>
    <!-- 用name='default'默认插槽加载异步组件 -->
    <AsyncChild />
    <!-- 异步加载成功前的loading 交互效果 -->
    <template #fallback>
      <div> Loading... </div>
    </template>
  </suspense>
</template>

<script setup>
  import { defineAsyncComponent } from 'vue'
  const AsyncChild = defineAsyncComponent({
    loader: ()=>import('./components/Child.vue'),
    delay: 200,
    timeout: 3000
  })
</script>
```

23、Vue3中，过渡动画<transition>发生了一系列变化。之前的 v-enter 变成了现在的 v-enter-from ， 之前的 v-leave 变成了现在的 v-leave-from 。另一个变化是：当使用<transition>作为根结点的组件，从外部被切换时将不再触发过渡效果。

```html
<template>
  <transition name='fade'>
    <h1 v-if='bol'>但使龙城飞将在，不教胡马度阴山！</h1>
  </transition>
  <button @click='bol=!bol'>切换</button>
</template>

<script setup>
  import { ref } from 'vue'
  const bol = ref(true)
</script>

<style lang='scss' scoped>
  .fade-enter-from { opacity: 0; color: red; }
  .fade-enter-active { transition: all 1s ease; }
  .fade-enter-to { opacity: 1; color: black; }

  .fade-leave-from { opacity: 1; color: black; }
  .fade-leave-active { transition: all 1.5s ease; }
  .fade-leave-to { opacity: 0; color: blue; }
</style>
```

24、在Vue3中，v-on的.native修饰符已被移除。

25、同一节点上使用 v-for 和 v-if ，在Vue2中不推荐这么用，且v-for优先级更高。在Vue3中，这种写法是允许的，但 v-if 的优秀级更高。

26、在Vue2中，静态属性和动态属性同时使用时，不确定最终哪个起作用。在Vue3中，这是可以确定的，当动态属性使用 :title 方式绑定时，谁在前面谁起作用；当动态属性使用 v-bind='object'方式绑定时，谁在后面谁起作用。

```html
<template>
  <!-- 这种写法，同时绑定静态和动态属性时，谁在前面谁生效！ -->
  <div id='red' :id='("blue")'>不负当下</div>
  <div :title='("hello")' title='world'>不畏未来</div>
  <hr>
  <!-- 这种写法，同时绑定静态和动态属性时，谁在后面谁生效！ -->
  <div id='red' v-bind='{id:"blue"}'>不负当下</div>
  <div v-bind='{title:"hello"}' title='world'>不畏未来</div>
</template> 
```

27、当使用watch选项侦听数组时，只有在数组被替换时才会触发回调。换句话说，在数组被改变时侦听回调将不再被触发。要想在数组被改变时触发侦听回调，必须指定deep选项。

```html
<template>
  <div v-for='t in list' v-text='t.task'></div>
  <button @click.once='addTask'>添加任务</button>
</template>

<script setup>
  import { reactive, watch } from 'vue'
  const list = reactive([
    { id:1, task:'读书', value:'book' },
    { id:2, task:'跑步', value:'running'}
  ])
  const addTask = () => {
    list.push({ id:3, task:'学习', value:'study' })
  }
  // 当无法监听一个引用类型的变量时
  // 添加第三个选项参数 { deep:true }  
  watch(list, ()=>{
    console.log('list changed', list)
  }, { deep:true })
</script>
```

28、在Vue2中接收 props时，如果 prop的默认值是工厂函数，那么在这个工厂函数里是有 this的。在Vue3中，生成 prop 默认值的工厂函数不再能访问this了。

```html
<template>
  <!-- v-for循环一个对象 -->
  <div v-for='(v,k,i) in info'>
    <span v-text='i'></span> -
    <span v-text='k'></span> -
    <span v-text='v'></span>
  </div>

  <!-- v-for循环一个数组 -->
  <div v-for='n in list' v-text='n'></div>
</template>

<script setup>
  import { defineProps, inject } from 'vue'
  // 为该 prop 指定一个 default 默认值时，
  // 如果是对象或数组类型，默认值必须从一个工厂函数返回。
  const props = defineProps({
    info: { type: Object, default () {
      // 在Vue3中，这里是没有this的，但可以访问inject
      console.log('this', this) // null
      return inject('info', { name:'张三', age:10 })
    }},
    list: { type: Array, default () {
      return inject('list', [1,2,3,4])
    }}
  })
</script>
```

29、Vue3中，新增了 <teleport>组件，这相当于 ReactDOM.createPortal()，它的作用是把指定的元素或组件渲染到任意父级作用域的其它DOM节点上。上面第 16个知识点中，用到了 <teleport> 加载 animate.css 样式表，这算是一种应用场景。

除此之外，<teleport>还常用于封装 Modal 弹框组件，示例代码如下：

```html
# Modal.vue

<template>
  <!-- 当Modal弹框显示时，将其插入到<body>标签中去 -->
  <teleport to='body'>
    <div
      class='layer' v-if='visibled'
      @click.self='cancel'
    >
      <div class='modal'>
        <header></header>
        <main><slot></slot></main>
        <footer></footer>
      </div>
    </div>
  </teleport>
</template>

<script setup>
  import { defineProps, defineEmits, toRefs } from 'vue'
  const props = defineProps({
    visibled: { type: Boolean, default: false }
  })
  const emit = defineEmits(['cancel'])
  const cancel = () => emit('cancel')
</script>

<style lang="scss">
.layer {
  position:fixed; bottom: 0;
  top: 0; right: 0; left: 0;
  background-color: rgba(0,0,0,0.7);
  .modal {
    width: 520px; position: absolute;
    top: 100px; left: 50%; margin-left: -260px;
    box-sizing: border-box; padding: 20px;
    border-radius: 3px; background-color: white;
  }
}
</style>
```

在业务页面组件中使用自定义封装的 Modal 弹框组件：

```html
<template>
  <Modal :visibled='show' @cancel='show=!show'>
    <div>弹框主体内容</div>
  </Modal>
  <button @click='show=!show'>打开弹框</button>
</template>

<script setup>
  import { ref, watch } from 'vue'
  import Modal from './components/Modal.vue'
  const show = ref(false)
</script>
```

30、在Vue3中，移除了 model 选项，移除了 v-bind 指令的 .sync 修饰符。在Vue2中，v-model 等价于 :value + @input ；在Vue3中，v-model 等价于 :modelValue + @update:modelValue 。在Vue3中，同一个组件上可以同时使用多个 v-model。在Vue3中，还可以自定义 v-model 的修饰符。

封装带有多个 v-model的自定义组件：

```html
# GoodFilter.vue

<template>
  <span>请选择商家（多选）：</span>
  <span v-for='s in shopArr'>
    <input
      type='checkbox'
      :value='s.value'
      :checked='shop.includes(s.value)'
      @change='shopChange'
    />
    <span v-text='s.label'></span>
  </span><br>

  <span>请选择价格（单选）：</span>
  <span v-for='p in priceArr'>
    <input
      type='radio'
      :value='p.value'
      :checked='p.value===price'
      @change='priceChange'
    />
    <span v-text='p.label'></span>
  </span>
</template>

<script setup>
  import { reactive, defineProps, defineEmits, toRefs } from 'vue'

  const props = defineProps({
    shop: { type: Array, default: [] },
    // 接收v-model:shop的自定义修饰符
    shopModifiers: { default: () => ({}) },
    price: { type: Number, default: 500 },
    // 接收v-model:price的自定义修饰符
    priceModifiers: { default: () => ({}) }
  })
  const { shop, price } = toRefs(props)
  // 接收v-model的自定义事件
  const emit = defineEmits(['update:shop', 'update:price'])

  const shopArr = reactive([
    { id: 1, label: '华为', value: 'huawei' },
    { id: 2, label: '小米', value: 'mi' },
    { id: 3, label: '魅族', value: 'meizu' },
    { id: 4, label: '三星', value: 'samsung' }
  ])
  const priceArr = reactive([
    { id:1, label:'1000以下', value: 500 },
    { id:2, label:'1000~2000', value: 1500 },
    { id:3, label:'2000~3000', value: 2500 },
    { id:4, label:'3000以上', value: 3500 }
  ])

  // 多选框
  const shopChange = ev => {
    const { checked, value } = ev.target
    // 使用v-model:shop的自定义修饰符
    const { sort } = props.shopModifiers
    let newShop = (
      checked
        ? [...shop.value, value]
        : shop.value.filter(e=>e!==value)
    )
    if (sort) newShop = newShop.sort()
    emit('update:shop', newShop)
  }
  // 单选框
  const priceChange = ev => {
    emit('update:price', Number(ev.target.value))
  }
</script>

<style lang='scss' scoped>
  .nav {
    &>span { display:inline-block; padding:5px 15px; }
    &>span.on { color:red; }
  }
</style>
```

使用带有多个 v-model 的自定义组件：

```html
<template>
  <GoodFilter
    v-model:shop.sort='shop'
    v-model:price='price'
  />
</template>

<script setup>
  import { ref, reactive, watch } from 'vue'
  import GoodFilter from './components/GoodFilter.vue'
  const shop = ref([])
  const price = ref(500)

  watch([shop, price], ()=>{
    console.log('changed', shop.value, price.value)
  })
</script>
```



## 六、写在最后



后续继续分享 **Vue3响应式原理、Vite构建工具、Pinia(2)、ElementPlus、Vant(3)** 等的使用。Vue3全家桶值得深入学习与关注，为Vue开发者带来全新的开发体验。