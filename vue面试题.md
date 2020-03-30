#### 一、vue的设计原理

深入响应式原理，数据驱动视图。注：只有在data里面的数据才可实现深入响应式。

#### 二、vue的双向绑定原理

概念阐述：vue的双向绑定是由数据劫持（Observer） + 发布者（compile) + 订阅者（wather）模式实现的。

**【Observer】**:能够对数据对象的所有属性进行监听，如有变动可拿到最新值并通知订阅者

**【compile】**:对每个元素节点的指令进行扫描和解析，根据指令模板替换数据，以及绑定响应的更新函数。

**【Wather】**: 作为链接`Observer`和`Compile`的桥梁，能够订阅并收到每个属性变动的通知，执行指令绑定响应的回调函数，从而更新视图

##### Observer（数据劫持）：

是通过`Obeject.defineProperty()`来监听数据的变动，这个函数内部可以定义`setter`和`getter`，每当数据发生变化，就会触发`setter`。这时候`Observer`就要通知订阅者，订阅者就是`Watcher`。

##### Wather（订阅者）:

是Observer数据与compile视图(模板)桥梁。

1、在自身实例化时往属性订阅器（dep）里面添加自己

2、自身必须有一个`update()`方法

3、待属性变动`dep.notice()`通知时，能调用自身的update()方法，并触发`Compile`中绑定的回调

##### Compile（发布者）：

负责解析模板指令，将模板中的变量替换成数据，然后初始化渲染页面视图，并将每个指令对应的节点绑定更新函数，添加监听数据的订阅者，一旦数据有变动，即更新视图。

![img](https://upload-images.jianshu.io/upload_images/12653633-d82f70569b8c385f.png?imageMogr2/auto-orient/strip|imageView2/2/w/785/format/webp)

**代码剖析：**<https://segmentfault.com/a/1190000006599500>

#### 三、Vue动态挂载路由，实现权限控制

**一、思路：**

- 在vue-router对该项中初始化公共路由的，例如404、login...
- 登录成功后，根据用户角色信息，获取对应权限菜单信息的menuList,并将menuList转换成我们需要的router结构。
- 动态挂载路由，通过vue-router2.2新添的router.addRouter(routes)方法动态挂载路由。并存于vuex和localStorage中。
- 渲染菜单：通过从vuex中获取的全部的路由信息，渲染左侧菜单实现动态路由

**二、实现：**

- 公共路由的定义

```js
import Vue from 'vue'
import Router from 'vue-router'
Vue.use(Router)
// layout
import Layout from '../views/layout/Layout'

export const constantRouterMap = [
  { path: '/login', component: () => import('@/views/login/index'), hidden: true },
  { path: '/404', component: () => import('@/views/404'), hidden: true },
  {
    path: '/',
    component: Layout,
    redirect: '/dashboard',
    name: 'Dashboard',
    hidden: true,
    children: [{
      path: 'dashboard',
      component: () => import('@/views/dashboard/index')
    }]
  },
]
export default new Router({
  scrollBehavior: () => ({ y: 0 }),
  routes: constantRouterMap
})

```

- 获取菜单信息

```js
router.beforeEach((to, from, next) => {
  NProgress.start() // start progress bar
  if (getToken()) { // determine if there has token
    /* has token*/
    if (to.path === '/login') {
      next({ path: '/' })
      NProgress.done() // if current page is dashboard will not trigger afterEach hook, so manually handle it
    } else {
      if (store.getters.roles.length === 0) { // 判断当前用户是否已拉取完user_info信息
        store.dispatch('GetInfo').then(res => { // 拉取user_info
         const roles = res.roles
          store.dispatch("GetMenu").then(data => {
            initMenu(router, data);
          });
          next()
        }).catch((err) => {
          store.dispatch('FedLogOut').then(() => {
            Message.error(err || 'Verification failed, please login again')
            next({ path: '/' })
          })
        })
      } else {
        next()
      }
    }
  } else {
    /* has no token*/
    if (whiteList.indexOf(to.path) !== -1) { // 在免登录白名单，直接进入
      next()
    } else {
      next('/login') // 否则全部重定向到登录页
      NProgress.done() // if current page is login will not trigger afterEach hook, so manually handle it
    }
  }
})

router.afterEach(() => {
  NProgress.done() // finish progress bar
})
```

- 动态加载路由

```js
mport store from '../store'

export const initMenu = (router, menu) => {
  if (menu.length === 0) {
    return
  }
  let menus = formatRoutes(menu);
  // 最后添加
  let unfound = { path: '*', redirect: '/404', hidden: true }
  menus.push(unfound)
  router.addRoutes(menus)
  store.commit('ADD_ROUTERS',menus)
}

export const formatRoutes = (aMenu) => {
  const aRouter = []
  aMenu.forEach(oMenu => {
    const {
      path,
      component,
      name,
      icon,
      childrens
    } = oMenu
    if (!validatenull(component)) {
      let filePath;
      const oRouter = {
        path: path,
        component(resolve) {
          let componentPath = ''
          if (component === 'Layout') {
            require(['../views/layout/Layout'], resolve)
            return
          } else {
            componentPath = component
          }
          require([`../${componentPath}.vue`], resolve)
        },
        name: name,
        icon: icon,
        children: validatenull(childrens) ? [] : formatRoutes(childrens)
      }
      aRouter.push(oRouter)
    }

  })
  return aRouter
}
```

- 渲染菜单

```js
<template>
  <el-scrollbar wrapClass="scrollbar-wrapper">
    <el-menu
      mode="vertical"
      :show-timeout="200"
      :default-active="$route.path"
      :collapse="isCollapse"
      background-color="#304156"
      text-color="#bfcbd9"
      active-text-color="#409EFF"
    >
      <sidebar-item v-for="route in permission_routers" :key="route.name" :item="route" :base-path="route.path"></sidebar-item>
    </el-menu>
  </el-scrollbar>
</template>

<script>
import { mapGetters } from 'vuex'
import SidebarItem from './SidebarItem'
import { validatenull } from "@/utils/validate";
import { initMenu } from "@/utils/util";

export default {
  components: { SidebarItem },
  created() {
  },
  computed: {
    ...mapGetters([
      'permission_routers',
      'sidebar',
      'addRouters'
    ]),
    isCollapse() {
      return !this.sidebar.opened
    }
  }
}
</script>
```

<https://blog.csdn.net/xp541130126/article/details/81513651>

#### <https://blog.csdn.net/qq_29468573/article/details/101515784>

#### 四、什么是Virtual DOM

- Virtual DOM只是一个概念，用来表示DOM节点的信息。通常是以一个树形对象存在于内存中。
- 在数据更新后，会存在两个Virtual DOM,一个是已更新的，一个是前一个的。
- 通过diff算法来找到两个Virtual DOM中数据更改的地方，并对这些节点做更新。

#### 五、Vue中的key起什么作用

**作用：**为了给DOM节点加一个唯一标识，有利于diff算法能更好的识别这个节点，提高性能。

**key值建议写法：**时间戳+自增函数

#### 六、v-if与v-show

- v-if是真正的条件渲染因为塌糊确保在切换过程中条件块内的事件监听器和子组件适当的被销毁和重建
-  v-if也是惰性的，如果在初始渲染时条件为假，则什么都不做，直到条件第一次变为真，才会开始渲染条件块
-  v-show:不管是什么条件，都是会被渲染，只是简单的进行css切换
-  v-if有更高的切换开销，v-show有更高的初始渲染开销。如果需要频繁的切换，使用v-show,如果在运行时条件很少改变，则使用v-if较好

#### 七、Vuex的核心概念

- **state**

  状态（基本数据）存储器

- **getters**

   基本数据的计算属性

- **mutations**

  提交更改数据的方法，同步。使用commit提交数据

- **action**

  包裹mutations提交数据，异步。通常在页面中使用dispatch触发。

- **modules**

  模块化Vuex。

**辅助函数**

- mapState

- mapGetters

- mapMutations

- mapAction

  