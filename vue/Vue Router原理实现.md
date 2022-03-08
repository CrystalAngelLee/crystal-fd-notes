# Vue Router 基础

## 基础使用步骤

**1. 创建项目**

使用 Vue-cli 创建项目的时候选择 Router，就会生成 Vue Router 的基础代码结构

**2. 使用**

2.1 创建一些路由视图 

2.2 在 router/index.js 页面引入 VueRouter 并注册路由插件

```js
import Vue from 'vue'
import VueRouter from 'vue-router'
import Index from '../view/Index.vue'

// 1. 注册路有插件
// Vue.use 是用来注册插件，会调用传入对象的 install 方法
Vue.use(VueRouter)

// 创建路由规则
const routes = [
  {
    path: '/',
    name: 'Index',
    component: Index
  },
  {
    path: '/blog',
    name: 'Blog',
    component: () => import(/* webpackChunkName: 'blog' */ '../view/Blog.vue')
  }
]

// 2. 创建 router 对象
const router = new VueRouter({ routes })
export default router
```

2.3 在 main.js 中导入 router 对象 并注册它

```js
import Vue from 'vue'
import App from './App.vue'
import router from './router'

...
new Vue({
  // 3. 注册 router 对象
  router,
  render: h => h(App)
}).$mount('#app')
```

2.4 在 App.vue 中创建路由组件的占位

```vue
<template>
	<div id="app">
    <div id='nav'>
      <!-- 5. 创建链接 -->
      <router-link to='/blog'>blog</router-link>
      <router-link to='/'>index</router-link>
  	</div>
    <!-- 4. 创建路由组件的占位 -->
    <router-view />
  </div>
</template>
```

**附：**

注册 router 对象会给 Vue 实例注册 $route(路由规则) 和 $router(VueRouter 的实例) 属性

## 动态路由

router/index.js

```vue
...
const routes = [
  ...,
	{
		path: '/detail/:id',
    name: 'Detail',
		// 开启 props，会把 URL 中的参数传递给组件，在组件中通过 props 来接收 URL 参数
		props: true,
    component: () => import(/* webpackChunkName: 'detail' */ '../view/Detail.vue')
	}
]
...
```

**获取路由参数的两种方式**

```vue
<template>
	<div>
    <!-- 方式一： 通过当前路由规则，获取数据 -->
    通过当前路由规则获取： {{ $route.params.id }}
    
    <!-- 方式二： 路由规则中开启 props 传参（推荐！！） -->
    通过开启 props 获取：{{ id }}
  </div>
</template>

<script>
	export default {
    name: 'Detail',
    props: ['id'] // 父子组件传值
  }
</script>
```

## 嵌套路由

router/index.js

```vue
...
const routes = [
	{
		path: '/login',
    name: 'Login',
    component: Login
	},
	{
		path: '/',
		component: Layout,
		children: [
			{
				name: 'index',
        path: '',
        component: Index
			},
      {
        name: 'detail',
        path: 'detail/:id',
        props: true,
        componet: () => import('../views/Detail.vue')
      }
		]
	}
]
...
```

## [编程式导航](https://router.vuejs.org/zh/guide/essentials/navigation.html#%E7%BC%96%E7%A8%8B%E5%BC%8F%E7%9A%84%E5%AF%BC%E8%88%AA)

```js
// 字符串路径
router.push('/users/eduardo')

// 带有路径的对象
router.push({ path: '/users/eduardo' })

// 命名的路由，并加上参数，让路由建立 url
router.push({ name: 'user', params: { username: 'eduardo' } })

// 带查询参数，结果是 /register?plan=private
router.push({ path: '/register', query: { plan: 'private' } })

// 带 hash，结果是 /about#team
router.push({ path: '/about', hash: '#team' })
```



# Hash 模式和 History 模式

## 表现形式的区别

> 要用好 History 模式需要服务端配置的支持
> Hash 模式 会有与数据无关的 特殊符号出现

| Hash 模式                       | History 模式               |
| ------------------------------- | -------------------------- |
| https://yumin.com/#/list?id=123 | https://yumin.com/list/123 |

## 原理区别

- Hash 模式基于锚点，以及 onhashchange 事件
  通过锚点的值作为路由地址，当地址发生变化后触发 onhashchange 事件，根据路径决定页面呈现的内容
- History 模式基于 HTML5 中的 History API
  - history.pushState()  // IE 10 以后才支持
    和 push 方法的区别是：当调用 history.push 时路径会发生变化，要像服务器发送请求；
    调用 history.pushState 不会像服务器发送请求，只会改变浏览器地址栏中的地址，并且将地址记录到历史记录中（可以实现客户端路由的依据）
  - history.replaceState()

## history 模式的使用

**需要服务器的支持**
reason： 单页应用中，只有一个 index.html 页面，服务端不存在 https://yumin.com/list 这样的地址，服务端不存在 `/list` 的页面，如果正常访问单页应用不会有任何问题，如果浏览器地址栏中的地址是  `/list`  刷新浏览器回像服务器发送请求（请求 `/list` 页面）服务器不存在该页面就会返回找不到该页面（404），所以在服务端应该配置除了静态资源外都返回单页应用的 index.html

# 模拟实现

