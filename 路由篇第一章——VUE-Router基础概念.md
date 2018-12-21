# VUE-Router基础概念

## 安装
因为本篇内容都是基于vue cli3框架的，所以这里就结合vue cli3框架解读。

vue-router在搭建vue项目基础的时候可以选取自动安装（具体可参考基础篇第一章），项目搭建完毕后在main.js文件中我们会看到如下代码：
```
import Vue from "vue";   //引入vue
import App from "./App.vue";   
import router from "./router";   //引入vue-router
import store from "./store";

Vue.config.productionTip = false;

new Vue({
  router,   //在new一个新的vue时引入router即可
  store,
  render: h => h(App)
}).$mount("#app");
```

当然，也可以在搭建完毕后npm手动安装，之后参照以上方式手动引入
```
npm install vue-router
```

## vue-cli3中路由配置示例

这里我们先给出一段总览的的路由配置代码，来逐步剖析路由的部分常用属性：
```
import Vue from "vue";
import Router from "vue-router";
import Home from "./views/Home.vue";

Vue.use(Router);

export default new Router({
  mode: "history",
  base: process.env.BASE_URL,
  routes: [{
      path: "/",      //默认展示的路由
      name: "home",
      component: Home
    },
    {
      path: "/login",
      name: "login",
      component: () => import("./views/Login.vue")
    },
    {
      path: "/about",
      name: "about",
      redirect: "/about/default",
      component: () => import("./views/About.vue"),
      children: [
        {
          path: "/about/default",
          name: "router1",
          component: () => import("./views/about/router1.vue")
        },
        {
          path: "/about/router2/:username/:password",
          name: "router2",
          component: () => import("./views/about/router2.vue")
        },
        {
          path: "/about/router3/:id(\\d+)",
          name: "router3",
          component: () => import("./views/about/router3.vue")
        },
        {
          path: "/about/router4",
          name: "router4",
          component: () => import("./views/about/router4.vue")
        }
      ]
    },
    {
      path: "/Index/:username",
      name: "index",
      component: () => import("./views/Index.vue"),
      children: [{
          path: "/charts",
          name: "chartshow",
          component: () => import("./views/chartArea/ChartsHome.vue")
        },
        {
          path: "/tables",
          name: "tableshow",
          component: () => import("./views/TableHome.vue")
        },
        {
          path: "/view",
          name: "viewshow",
          component: () => import("./views/ViewHome")
        }
      ]
    }
  ]
});
```
对应文件主要目录如下：

![文件目录](/IMG/vue11.png)

### 针对上述路由配置做几点解析
#### 1.组件的引入方式分两类：

一类即在页面头部采用import引入，比如上述router配置中home页的引入
```
import Home from "./views/Home.vue";
```
另一类就是所说的路由懒加载，即在单个路由配置项中直接引入：
```
component: () => import("./views/Login.vue")
```
原因：当打包构建应用时，Javascript 包会变得非常大，影响页面加载。如果我们能把不同路由对应的组件分割成不同的代码块，然后当路由被访问的时候才加载对应组件，这样就更加高效了。所以推荐使用第二类组件加载方式---懒加载
#### 2.单个路由配置属性意义分析
path: 路径  注意：这里的路径除了默认首路径为'/'外，其余路径形式都是自定义的，也就是说不一定非要按照对应组件所在路径来配置，如上图中的./views/Login.vue组件，path就可以写作"/login"或者其他的都可以。

name: 即所谓的命名路由，用途跟path一样，主要是用来进行路由跳转的时候用的，相对来说使用起来更方便。至于具体的区别，这个在后面会详述  

component: 组件  这里指路由指向的组件，此处引入路径是否正确是确定组件能否正常引入的标准

redirect: 重定向路径  重定向顾名思义即重新确定指向的路由（组件），主要在嵌套路由中使用，用以重新定向二级乃至三级的默认展示路由

children: 子路由配置  配置下级路由的对象数组，嵌套路由属性大体与一级路由相同。

### 常用知识点

#### 动态路由匹配

所谓动态路由，简单的说就是指向同一个组件，但可以携带不同的参数的路由。
这里以二级路由router2示例：
```
{
  path: "/about/router2/:username/:password",
  name: "router2",
  component: () => import("./views/about/router2.vue")
}
```
我们可以看到该路由可匹配（或携带）2个参数username和password,每个路径参数用“：”标记。这样我们可以使用路由时，不管是"/about/router2/admin/pass"还是"/about/router2/MrsLi/123456"，它们都会指向同一个路由。

那么有一个问题，必须是动态路由匹配才能使路由携带参数吗？  
答案显然是否定的，这里我们先不解释，先做几个概念的解析
##### route和router
router指代的是什么不必多说，即vue-router实例对象，相当于一个路由管理器。形式如下：

![router对象](/IMG/vue1_3.png)  
route指当前激活的路由对象，形式如下：

![route对象](/IMG/vue1_2.png)  
其包含当前激活路由的相关参数，比如基本的name，path.  
这里我们还发现两个参数对象params和query。由图可见params包含2个键值对，显然应该是当前路由携带的参数,而query中则是空的。这显然意味着二者是有区别的，在编程式导航中我们在做详述。

##### 编程式导航
路由导航有两种，一种是基本的声明式导航，这个在官网查看下即可明白，这里不多提，主要来描述下编程式导航，类似以下形式：

```
router.push()
```
在实际项目开发中声明式导航用起来并不方便，且在面临多级路由时会出现一些莫名的问题，大部分情况下推荐使用编程式导航。
基本用法示例：
```
this.$router.push({path: "/login"})  
this.$router.push({name: "login"})  
this.$router.push({name: "router4",params:{username:"丑",password: 12345621}})  
this.$router.push({path: "/about/router4",query:{username:"丑",password: 123454321}});
```
由上述我们可以看到，编程式导航的导航形式有两种，一种是命名路由name,一种是路径path。那么这两者有什么区别呢？

在route的介绍中我们知道了激活的路由对象中包含query和params两个对象，正对应编程式导航传参的2种形式，即name-params和path-query。（这两者是一一对应的，不能混搭），这就很明了了，params和query都是路由携带的参数对象。而二者同时只能存在一个。

我们通过以下示例来做下验证。
这里来进行router2和router4之间的相互跳转，router2和router4的区别是前者给定了动态路由的匹配模式/:username/:password.

由router2跳转到router4，采用以下三种push形式

```
this.$router.push({name: "router4",params:{username:"丑",password: 12345621}}) 成功

this.$router.push({path: "/about/router4",query:{username:"丑",password: 123454321}}) 成功

this.$router.push({path: "/about/router4?username=丑&password=12581"}) 成功
```
由router4跳转到router2，采用以下两种push形式

```
this.$router.push({name: "router2",params: { username: "卯", password: 12345621}})成功  
this.$router.push({path: "/about/router2",query:{username:"卯",password: 123454321}})失败
```

对比发现name-params模式在二者间执行路由跳转都是成功的，但path-query模式在由router4跳转到router2时是失败的，这是为什么呢？

这里我们先看下name-params和query-path两种模式在跳转url上的体现上有什么不同：

1.name-params模式

![router对象](/IMG/vue1_4.png) 

2.path-query模式

![router对象](/IMG/vue1_5.png) 

通过比较我们可以看到，name-params模式执行的路由url与路由配置中动态路由匹配的形式你一致，而path-query则是另一种？查询模式。这样原因就找到了，因为路由配置中给定了动态路由的匹配模式，所以只能采用name-params模式来执行编程式导航，path-query模式则会失效。

除此之外我们还会发现，路由router4并没有给定的动态匹配模式，但以上采用的三种push方式，都成功的跳转了路由，且name—params模式的路由字符串中也没有包含所传的参数（指的是url中没有携带），但在跳转成功后激活的route对象中的params仍是携带了所传参数的。

所以动态路由匹配并不是传参必须的，鉴于避免在url中暴露所携参数，我们大可以不给定匹配模式，而是直接采用name-params模式进行路由信息传递，相对来说会安全点。

##### 拓展
上面针对动态匹配路由只剖析了最常用的一种传参形式，除此之外，动态路由还有一些高级用法，比如参数的正则验证,例如二级路由router3，针对参数id添加了正则验证——传参须是一个或多个1到9的整数。如果你用以下导航，则是不能顺利切换路由的：

```
this.$router.push({name:"router3",params:{id:a12}}) //路由验证不通过
```

#### 嵌套路由

嵌套路由，顾名思义就是一个路由组件中还包含其他的路由（或者说子路由），而子路由也可包含子路由，这样就形成了路由的嵌套。

从文章开头给出的路由配置我们可以看到about和index两个路由都嵌套有子路由，而配置子路由需要的就是一个children属性。那么当进入about路由时，怎么确保会展示指定的一个子路由呢？这里就用到了路由重定向（redirect）这个属性。

##### 重定向

拿官网的一个简单的示例来看：
```
routes: [
    { path: '/a', redirect: '/b' }
  ]
```
重定向就是说当进入路由"/a"的组件后，重新定向跳转到路由"/b",这样就实现了上面说的如何进入二级路由组中指定的一个子路由。
```
{
      path: "/about",
      name: "about",
      redirect: "/about/default",  //重定向到默认子路由
      component: () => import("./views/About.vue"),
      children: [
        {
          path: "/about/default",  //默认子路由
          name: "router1",
          component: () => import("./views/about/router1.vue")
        },
        {
          path: "/about/router2/:username/:password",
          name: "router2",
          component: () => import("./views/about/router2.vue")
        },
        {
          path: "/about/router3/:id(\\d+)",
          name: "router3",
          component: () => import("./views/about/router3.vue")
        },
        {
          path: "/about/router4",
          name: "router4",
          component: () => import("./views/about/router4.vue")
        }
      ]
}
```


重定向的目标可以是示例中的路径，也可以是命名路由，比如
```
{
  path:"/a",
  redirect: {name: "b"}
}
```

