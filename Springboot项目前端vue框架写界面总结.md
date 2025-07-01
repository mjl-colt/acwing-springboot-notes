# Springboot项目前端vue框架写界面总结

# 1.实现关于路由

在App这个vue里面template导入**`<router-view />` ， `<router-view />` **是 Vue Router 提供的占位符组件，通俗理解为它是管理链接，可以跳转链接。比如说从http://localhost:8080/pk/跳转到http://localhost:8080/record/。它实际上是占据这个位置，等链接到对应网址的对应网页。就会显示对应网页的内容。

- 它会动态地渲染与当前路径（URL）匹配的组件。
- 例如，当路径为 `/pk/` 时，`router-view` 会加载 `PkIndexView` 组件。
- 它的作用是将路由系统中定义的页面组件插入到模板中。

```vue
<template>
  <NavBar />
  <!-- 这个是用来导航的，写在router文件夹的index.js里面 -->
  <router-view/>
</template>
```

再来看此时router文件夹中的index.js文件。他才是路由系统核心文件。其实可以理解为写跳转链接的地方。index.js文件是 Vue Router 的配置文件，定义了路径（`path`）、名称（`name`）和对应的组件（`component`）。一般都写成这样`path`表示路径。`name`表示路由唯一名称。`component`表示对应的页面组件。Vue Router不是自带的，是创建项目的前期导入的依赖。一般他会在1.在 `src/router/index.js` 文件中定义路由2.在主入口文件 `main.js` 中引入路由

**那么输入 URL 后，系统如何找到对应的页面？**

 **流程分析：**

1. **输入路径**（如 `http://localhost:8080/pk/`）。

2. 路由系统匹配路径

   - Vue Router 在 `routes` 中查找与 `path` 相匹配的配置项。

   - 在这里，

     ```javascript
     /pk/
     ```

      匹配的是：

     ```javascript
     {
       path: "/pk/",
       name: "pk_index",
       component: PkIndexView,
     }
     ```

3. 加载组件

   - Vue Router 将 `PkIndexView` 组件动态地插入到 `<router-view />` 的位置。

4. 页面渲染

   - 渲染完成后，浏览器显示对应的内容。

 **关键点：`component`**

- `component` 指定了匹配路径后加载的 Vue 组件。
- 例如，`path: "/pk/"` 的 `component` 是 `PkIndexView`，所以访问 `/pk/` 时，会渲染 `PkIndexView`。

```javascript
import { createRouter, createWebHistory } from 'vue-router'
import PkIndexView from '../views/pk/PkIndexView'
import RanklistIndexView from '../views/ranklist/RanklistIndexView'
import RecordIndexView from '../views/record/RecordIndexView'
import UserBotIndexView from '../views/user/bot/UserBotIndexView'
import NotFound from '../views/error/NotFound'
 
 
const routes = [
  {
    path:"/", /**根路径的重定向*/
    name:"home",
    redirect:"/pk/"
  },
  {
    path:"/pk/",/**相对路径(域名后开始) */
    name:"pk_index",
    component:PkIndexView,/**访问上面路径后映射的组件 */
  },
  {
    path: "/record/",
    name: "record_index",
    component: RecordIndexView,
  },
  {
    path: "/ranklist/",
    name: "ranklist_index",
    component: RanklistIndexView,
  },
  {
    path: "/user/bot",
    name: "user_bot_index",
    component: UserBotIndexView,
  },
  {
    path: "/404/",
    name: "404",
    component: NotFound,
  },
  {
    path: "/:catchAll(.*)",/**匹配所有其他路径，重定向到404 */
    redirect: "/404/",
  }
 
]
 
2. 路由模式（history）
Vue Router 提供了两种路由模式：
createWebHistory()：
基于 HTML5 History API，无需 # 号（例如 /about）。
createWebHashHistory()：
使用哈希模式，路径带有 #（例如 /#/about）。
const router = createRouter({
  history: createWebHistory(),
  routes
})
 
export default router

```

这是main.js，这句话import router from './router'是用来引入路由，创建 Vue 应用实例并使用路由 createApp(App)，挂载路由 app.use(router);  挂载应用到 DOM，app.mount('#app'); 为什么app.mount('#app')里面是app因为在public的index.html的html中定义的div的id是app，即<div id="app"></div>。

```javascript
import { createApp } from 'vue'
import App from './App.vue'
import router from './router'
import store from './store'

createApp(App).use(store).use(router).use(router).mount('#app')
```

# 2.实现关于点击标题会跳转链接

初始简陋版本是如下：(他的缺点在于：浏览器会刷新并显示新页面)

```html
<a class="nav-link" aria-current="page" href="/pk/">对战</a>
```

首先分析明白它工作原理，<a> 是超链接元素：`href` 属性定义链接目标。当用户点击链接时，浏览器会根据 `href` 的值跳转到指定的 URL。

至于为什么能跳转到对应界面，又归因于使用了 Vue Router 作为路由系统。在路由配置文件 `index.js` 中，定义了 `/pk/` 的路由规则，Vue Router 拦截了浏览器的默认跳转行为，将路径 `/pk/` 映射到 `PkIndexView` 组件并在 `<router-view>` 中渲染。

```javascript
const routes = [
  {
    path: "/pk/",
    name: "pk_index",
    component: PkIndexView, // 对应的组件
  },
];
```

新的版本是利用Vue Router 提供的专用组件<router-link>，这样就可以避免刷新。

```html
<router-link class="nav-link" :to="{name: 'pk_index'}">对战</router-link>
```

**`<router-link>`**：Vue Router 提供的组件，用于实现页面间的路由跳转。`:to` 是 Vue 的绑定语法，用来指定跳转的目标路由。`{name: 'pk_index'}` 是一个 路由对象，通过指定 `name` 来匹配路由规则。name必须在index.js的routes中定义。:to可以绑定多种形式，可以是字符串：`<router-link :to="/pk/">对战</router-link>`。可以是对象：`<router-link :to="{name: 'pk_index'}">对战</router-link>`。但是最好写成对象的形式，为什么呢？**避免硬编码路径**：如果路径修改，只需调整路由配置，而不需要改动每一个 `router-link`。**支持动态参数**：可以通过 `params` 和 `query` 轻松生成动态路径。那`params` 和 `query`又是什么呢？

`params` 是动态路由参数，可以生成路径的一部分。这样生成的路径是 `/user/123`。

```javascript
const routes = [
  {
    path: '/user/:id',  // 路径中包含动态参数 :id
    name: 'user_detail',
    component: UserDetailView,
  },
];
```

```html
<router-link :to="{name: 'user_detail', params: {id: 123}}">用户详情</router-link>
```

`query` 是查询参数，附加在 URL 的末尾。生成的路径是 `/pk/?page=2&size=10`。

```html
<router-link :to="{name: 'pk_index', query: {page: 2, size: 10}}">对战</router-link>
```

# 3.实现点击谁，谁就亮的操作，解析下面代码。结合来看。

```javascript
<script >
import { useRoute } from "vue-router"
import { computed } from "@vue/reactivity"
export default{
  setup(){
    const route = useRoute();
    //使用computed必须使用函数，这里用的是箭头函数。
    let route_name = computed(() => route.name) 
    return{
      route_name //return 是用来向模板 (template) 暴露变量和方法的关键步骤。
    }
  }
}     
</script>
```

两个import，第一个导入useRoute。**`useRoute` 的作用**：它返回当前激活的路由对象。返回的对象包括：

```javascript
{
  path: '/home',       // 当前路径
  name: 'home',        // 当前路由的名称
  params: {},          // 动态路径参数
  query: {},           // 查询参数 (?key=value)
  matched: [...],      // 匹配到的路由记录
  ... 其他属性
}
```

这样的话就可以使用返回的route.name。

第二个导入computed。**`computed` 的作用**：创建一个计算属性，依赖的值发生变化时，计算属性会自动重新计算。route.name 是响应式的，因此用 computed可以自动监听它的变化并更新。说白话就是如果状态发生变化，computed可以立刻知道。对于上述代码即为route.name发生变化，可以立刻捕捉到。

```html
<router-link :class="route_name == 'pk_index' ? 'nav-link active' : 'nav-link' ">
  对战
</router-link>
```

这里:class=其实是v-bind:class=的缩写。只有加上它们才能实现动态绑定 HTML 属性的值，值可以是变量、计算属性或 JavaScript 表达式。如果不加：那么它就是静态的无法改变。只有加了才是动态可以改变，这样就可以和上述说的route_name联系起来，当 `route_name` 的值变化时，`class` 会随之更新。

# 4.实现对于动态刷新

这是一个对于所有游戏或者说页面需要写的代码。在scripts的AcGameObject.js中，一秒60帧率，每帧刷新一次。这个是由requestAnimationFrame(step)实现的。当我们开始调用`requestAnimationFrame(step)`，会在下一帧执行`step`，在下一帧执行`step`时，执行完到最后再次触发`requestAnimationFrame(step)`，会在下下帧执行`step`。

![1](C:\Users\30816\OneDrive\桌面\Springoot项目笔记\image\1.png)

#### **整体流程总结**

1. **游戏对象的创建**：
   - 每当创建一个新的 `AcGameObject` 时，会自动加入到 `AC_GAME_OBJECTS` 数组中。
   - 通过构造函数初始化状态。
2. **帧循环启动**：
   - 调用 `requestAnimationFrame(step)` 开始动画帧循环。
   - 每一帧：
     - 检查对象是否需要执行 `start` 方法。
     - 如果 `start` 已执行，调用 `update` 更新对象逻辑。
3. **对象销毁**：
   - 调用destroy方法时：
     - 执行销毁前的逻辑（`on_destroy`）。
     - 从 `AC_GAME_OBJECTS` 数组中移除对象。

### 对于这个代码，有几个细节需要重申一下。第一个细节：

```javascript
class AcGameObject {
    constructor() {
        AcGame_Object.push(this); // 将当前实例加入数组
        this.timedelta = 0; // 初始化时间间隔
        this.has_called_start = false; // 初始化标记
    }
}
```

当你创建 `AcGameObject` 的实例时，例如：const obj = new AcGameObject();`this` 指向这个实例 `obj`。

如果有类继承了 `AcGameObject`：

```javascript
class Player extends AcGameObject {
    constructor() {
        super(); // 调用父类的构造函数
        this.name = "player"; // 子类的自定义属性
    }
}
const player = new Player();
```

在子类的构造函数中，`this` 指向 `Player` 的实例 `player`。`super()` 调用了父类的构造函数。在 `super()` 执行期间，`this` 仍指向当前的子类实例 `player`。

因此：`this.timedelta` 和 `this.has_called_start` 是 `Player` 实例的属性。`AcGame_Object.push(this)` 会将 `player` 添加到全局数组中。

### 第二个细节，总体流程的一个梳理

```javascript
// game_object.js

export const GAME_OBJECTS = [];

export class GameObject {
  constructor(name, speed) {
    this.name = name;
    this.x = 0;
    this.speed = speed; // px/s
    this.timedelta = 0;
    this.has_called_start = false;
    GAME_OBJECTS.push(this);
  }
  start() {
  }
  update() {
  }
  on_destroy() {
  }
  destroy() {
  }
}

let last_timestamp;

function step(timestamp) {
  for (let obj of GAME_OBJECTS) {
    if (!obj.has_called_start) {
      obj.has_called_start = true;
      obj.start();
    } else {
      // timedelta 反映的是每帧的真实时间差。因为每帧的时间间隔不一定是相同的，所以要计算一下
      obj.timedelta = timestamp - last_timestamp;
      obj.update();
    }
  }

  last_timestamp = timestamp;
  requestAnimationFrame(step);
}

export function start_game_loop() {
  requestAnimationFrame(step);
}



// main.js中进行调用

import { GameObject, start_game_loop } from './game_object.js';

// 创建两个物体
const obj1 = new GameObject("飞机", 100); // 100 px/s
const obj2 = new GameObject("子弹", 300); // 300 px/s

// 启动渲染循环
start_game_loop();


//如果你不封装成 start_game_loop() 函数，而是直接在模块中写：requestAnimationFrame(step);
//模块一被 import，requestAnimationFrame(step) 就立即运行一次，即import { GameObject } from './game_object.js'就会运行一次
//直接写 requestAnimationFrame(step) 会让循环在 模块被 import 时立刻启动，
//封装成 start_game_loop() 函数是为了让你控制启动时机，推荐用于正式项目。
```



### 第三个细节，函数部分的总体梳理。梳理：函数表达式（即箭头函数），函数声明，普通函数这三个东西。

先看下面这个箭头函数和普通函数。它们中的this的针对的对象不一样。通常使用箭头函数即为继承所属的类，使用的更常见一点。

```javascript
//箭头函数
const step = timestamp => {
};
//普通函数
const step = function(timestamp) {
};
```

这个里面step是函数，timestamp是参数。这里的箭头函数，基本格式应该为const 函数名 = (参数) => { 函数体 }，因为只有一个参数，所以( )可以省略。严谨的说这个箭头函数和普通函数都是匿名函数。因为function后面没有名字。我们举个例子来看一下。这里的**`sayHello` 是一个变量名**，它保存了匿名函数的引用，但它本身不是函数的名字。它只是一个引用。同样的道理，上面的两个代码中的step也都是引用。

```javascript
//匿名函数：没有函数名。
const sayHello = function() {
  console.log("Hello!");
};
//具名函数：可以给函数添加名字，便于调试。
//但此时，greet 只能在函数内部访问，外部仍需通过 sayHello 调用。
const sayHello = function greet() {
  console.log("Hello!");
};
```

我们再来看一下const的使用，这与平常在java,cpp中的使用方式不同。在 JavaScript 中，**函数也是一种值**，可以赋值给变量。`const step = ...` 是将函数的引用存储在常量 `step` 中。这个量是无法改变的，但是如果使用let定义，即可以改变。对比见下面代码。

```javascript
// 使用 const
const step1 = function(timestamp) {
  console.log("Step 1");
};
step1(); // 输出：Step 1

step1 = function() {
  console.log("Another Step 1");
}; // ❌ 报错：Assignment to constant variable

// 使用 let
let step2 = function(timestamp) {
  console.log("Step 2");
};
step2(); // 输出：Step 2

step2 = function() {
  console.log("Another Step 2");
}; // ✅ 合法，可以重新赋值
step2(); // 输出：Another Step 2
```

最后我们说一下，说一下函数表达式和函数声明。函数声明和我们在java和cpp学的函数很像。它比较来说有几个不同。**函数声明**：必须有名字。会被提升，因此可以在定义之前调用。**函数表达式**：可以是匿名函数或具名函数。不会被提升，必须先定义后使用。更灵活，适合动态场景。个人感觉可以定义前调用最好用。

```javascript
//函数表达式
const step = function(timestamp) {
  console.log(timestamp);
};
//函数声明
function step(timestamp) {
  console.log(timestamp);
}
```

# 5.现在讨论一下script里面的知识点。

export default里面写components:{}和steup(){}的用处不相同。components: {}用法即先用import把其他的vue文件导入进来，然后在components: {}进行注册导出，才能在这个vue模块的template中使用。官方表达即为：注册后，子组件才能在当前组件中使用。**`components: {}`** 是告诉 Vue：**“我这个组件需要用到哪些子组件”**。

```vue
// ParentComponent.vue
<script>
import ChildComponent from './ChildComponent.vue';

export default {
  components: {
    ChildComponent // 在这里注册子组件
  }
};
</script>

<template>
  <div>
    <ChildComponent /> <!-- 在模板中使用子组件 -->
  </div>
</template>
```

我们再讨论一下setup函数的使用，它的作用与 `components: {}` 的作用完全不同，并不是导入别的vue。`setup` 是用来：1.定义响应式数据（例如使用 `ref` 和 `reactive`）。2.定义方法、计算属性、生命周期钩子等。3.返回需要暴露给模板的变量和方法。`setup` 返回的内容（如 `count` 和 `increment`）会自动暴露给模板（`template`）。它负责当前组件的逻辑处理，而不是注册子组件。**`setup`** 是告诉 Vue：**“我这个组件需要什么逻辑和数据”**。

```vue
<script>
export default {
  setup() {
    const count = ref(0);
    const increment = () => count.value++;

    return {
      count,
      increment
    };
  }
};
</script>

<template>
  <div>
    <p>{{ count }}</p>
    <button @click="increment">增加</button>
  </div>
</template>

```

需要注意几个点`export default` 里面必须是对象，所以要写成setup: () => {}` 这是一个匿名函数！而不是 `setup = () => {}。前者可以理解为setup: function()是一个对象。而后面的理解为setup=function()，这是一个变量。这个setup对象里面才可以写变量，函数这些东西，例如const count = ref(0);这个变量，const increment = () => count.value++;这个函数。这里还有一个语法糖，可以省区return这个步骤。即写成<script setup> </script>这种样子。他会自动return所有变量和函数供给template使用。

```vue
<script setup>
import { ref } from 'vue';

const count = ref(0);
const increment = () => {
  count.value++;
};
</script>

<template>
  <div>
    <p>{{ count }}</p>
    <button @click="increment">增加</button>
  </div>
</template>
```

上面的代码涉及到很多响应式布局的知识点。<p>{{ count }}</p> 中的 {{}} 是 Vue 中的 插值语法，用于将 JavaScript 表达式的结果动态渲染到模板中。如果 `count` 发生变化即加一或者减一等操作，Vue 会自动更新页面显示。`ref(0)` 表示创建一个初始值为 `0` 的响应式变量。只要写了ref就表示基本响应式数据，可以在页面上改变的。

```vue
$.ajax({
  url: "请求的地址",
  type: "请求方法",   // GET 或 POST
  data: { 发送的数据 }, // 可选，POST 需要这个
  success: function(resp) { // 成功后执行的回调
    console.log("成功了", resp);
  },
  error: function(xhr, status, error) { // 失败时执行的回调
    console.log("出错了", status, error);
  }
});
```

`$.ajax({})` 是 **jQuery** 提供的 **Ajax（异步请求）方法**，用于在前端和后端之间交换数据。

`$.ajax()` 是 jQuery 的 AJAX 方法，它用于**向服务器发送 HTTP 请求**。

 你可以用它来向服务器发送请求（GET / POST等），并获取返回的数据。

# 6.讨论一下canvas的一些基础使用

在传统的 JavaScript 中，我们确实需要通过`document.getElementById("canvas")` 获取画布元素，再通过 `getContext("2d")` 获取 2D 绘图上下文。这两步结合起来可以理解为获得一个画布和画笔。

```javascript
//传统代码
var canvas = document.getElementById("canvas");//获取画布
var ctx = canvas.getContext("2d");//获取画笔
initializeGameMap(ctx);// 初始化画布
```

在vue方式中，`ref` 绑定了模板中的 `<canvas>` 元素。`canvas.value` 在组件挂载后会自动指向对应的 DOM 元素（即在template中的`<canvas>` 标签）。`canvas.value.getContext('2d')`：直接在绑定的 `canvas` 元素上调用 `getContext('2d')` 获取 2D 绘图上下文（画笔）。

```vue
//Vue 的方式
<template>
    <canvas ref="canvas">
    </canvas>
</template>

<script>
	import { ref, onMounted } from 'vue';
	let canvas = ref(null);

	onMounted(() => {
    	const ctx = canvas.value.getContext('2d');
    	initializeGameMap(ctx); // 初始化画布
	});
</script>
```

说下DOM元素，DOM 元素是 HTML 标签在浏览器中的表现形式。它是浏览器对 HTML 网页结构的解析结果，便于 JavaScript 操作。常用的方法有获取元素（`getElementById`）和操作元素（`style`、`textContent` 等）。就比如var canvas = document.getElementById("canvas");//获取画布，这也是直接对DOM元素进行操作。使用ref就避免了直接对DOM元素操作。

说下ref(null)` 和 `ref(0)分别什么时候使用。`ref()` 是 Vue 3 中创建响应式变量的方法。`ref(0)` 表示一个初始值为 `0` 的响应式变量，适用于数字类型`ref(null)` 表示一个初始值为 `null` 的响应式变量，适用于需要绑定 DOM 元素或对象的情况。

解析一个例子

```vue
GameMap.vue

<template>
<!--
这里面的两个ref分别绑定不同的东西 
<div> 就像是一个白板挂在墙上,
<canvas> 就是你往这个白板上贴的一张纸，
你用 JavaScript 拿着“画笔（canvas.getContext('2d')）”在纸上画图，
而画纸的大小，往往是根据白板（div）的大小来设置的。
-->

  <div ref="parent" class="gamemap">是容器，确定尺寸
    <canvas ref="canvas"></canvas>是真正的绘图区域，画图是在它上面画，不是在 div 上画的。
  </div>
</template>

<script>
import { GameMap } from "@/assets/scripts/GameMap.js";
import { ref, onMounted } from "vue";

export default {
  setup() {
    const parent = ref(null);
    const canvas = ref(null);

    onMounted(() => {
      const ctx = canvas.value.getContext("2d");
      // 这块就是进入GameMap.js文件了，这里传了两个参数，方便在里面统一控制尺寸和绘图。
      // 第一个参数，canvas.value.getContext('2d') 是画笔
      // 第二个参数，parent.value 是画布外的画框大小
      const gameMap = new GameMap(ctx, parent.value);
    });

    return {
      parent,
      canvas,
    };
  },
};
</script>

<style scoped>
.gamemap {
  width: 60vw;
  height: 70vh;
  border: 1px solid black;
}
canvas {
  width: 100%;
  height: 100%;
}
</style>
```

```javascript
GameMap.js

import { AcGameObject } from "./AcGameObject.js";

export class GameMap extends AcGameObject {
  //这个构造函数接收两个参数
  constructor(ctx, parent) {
    super();
    this.ctx = ctx;
    this.parent = parent;

    this.width = this.parent.clientWidth;
    this.height = this.parent.clientHeight;
	// 通过 ctx 拿到它所绑定的 canvas DOM 元素。
    //ctx.canvas 是一个属性，返回的是 canvas 标签。
    //你可以把这一步当成“反查回 canvas 元素”。
    this.canvas = this.ctx.canvas;

    // 设置 canvas 宽高以匹配 div 容器
    this.canvas.width = this.width;
    this.canvas.height = this.height;

    this.start();
  }

  start() {
    this.render();
  }

  render() {
    // 在 canvas 中绘制一个蓝色矩形，大小就是容器大小
    // this.ctx.fillStyle = "skyblue";设置当前画笔的填充颜色。
    this.ctx.fillStyle = "blue";
    // 用刚才设置的填充颜色，在 canvas 上画一个矩形。
    this.ctx.fillRect(0, 0, this.width, this.height);
  }
}
```

ctx有很多函数，下面就是比较常用的：

| 语句                             | 意义     | 类比                 |
| -------------------------------- | -------- | -------------------- |
| `this.ctx = ctx`                 | 拿到画笔 | 拿到一支笔           |
| `this.canvas = this.ctx.canvas`  | 拿到画布 | 找到这支笔对应的纸张 |
| `this.ctx.fillStyle = "skyblue"` | 设置颜色 | 把笔装上天蓝色墨水   |
| `this.ctx.fillRect(...)`         | 画图     | 用笔画一个矩形背景   |

# 7.讨论这里rows（行数）和cols（列数），与（canvas）的x和y的关系

![2](C:\Users\30816\OneDrive\桌面\Springoot项目笔记\image\2.png)

所以此时在Cell.js文件中，有 this.x = c + 0.5; // 中心点的 x 坐标（列号 + 0.5）和 this.y = r + 0.5;// 中心点的 y 坐标（行号 + 0.5），x对应的是c列数，y对应的是r行数。

上面的坐标的一个不同，主要是用于下面这个函数中，函数的每个参数如下展示：

## ✅ `fillRect(x, y, width, height)` 的参数含义如下：

| 参数     | 说明                                            |
| -------- | ----------------------------------------------- |
| `x`      | 矩形左上角的 **x 坐标**（相对于 canvas 左上角） |
| `y`      | 矩形左上角的 **y 坐标**（相对于 canvas 左上角） |
| `width`  | 矩形的 **宽度**（向右延伸的距离）               |
| `height` | 矩形的 **高度**（向下延伸的距离）               |

# 8.vuex的使用

`useStore()` 是 Vuex 提供的**组合式 API 入口**，用于获取全局的 `store` 实例。所以我们写import { useStore } from 'vuex'和const store = useStore()这两行组合代码，就可以读取state中的数据。

```javascript
import { useStore } from 'vuex'

const store = useStore() // 这个就是拿到 main.js 里挂载的全局 store
```



为什么能读取，是因为在初始的main.js中，类似于下面的main.js。其中import store from './store'导入，.use(store)挂载。然后就可以使用了。注意，`useStore()` 只是「获取入口」，他是个函数；状态本身是在你用 `createStore({...})` 创建的那一套东西里定义的。

```javascript
// main.js
import { createApp } from 'vue'
import App from './App.vue'
import store from './store'

createApp(App)
  .use(store)  // <<== 这里把 store 挂到全局了
  .mount('#app')
```

再解释一下，export default createStore({...})，它的作用是导出一个 Vuex store 实例，就是**“把一个用 `createStore()` 创建出来的 Vuex 状态仓库对象导出去”**，让别的文件可以用这个仓库来管理全局数据。export default {...}的作用是**把一个对象“丢出去”，让别的文件可以用它。**对应代码中，即

![3](C:\Users\30816\OneDrive\桌面\Springoot项目笔记\image\3.png)

index.js中用export default createStore({...})，然后把pk.js，record.js，user.js全都用export default {...}导入到index.js的module模块中。类似于下面这种。

```javascript
// store/index.js
import { createStore } from 'vuex'

// 引入模块
import user from './user'
import record from './record'
import pk from './pk'

export default createStore({
  modules: {
    user, // ← 注册模块
    record,
    pk
  }
})
```

再说一下，vuex的核心结构。

```javascript
import { createStore } from 'vuex'

export default createStore({
  state: {
  },
  mutations: {
  },
  actions: {
  },
  modules: {
  }
})
```

### ✅ 1. `state`

🔹 作用：**用来存储全局共享的数据（状态）**

```javascript
state: {
  count: 0,
  username: '小明'
}
```

你可以在组件中通过：

```javascript
const store = useStore()
store.state.count
```

来访问这些共享数据。

### ✅ 2. `mutations`

🔹 作用：**用来修改 state 中的数据，必须是同步的！**

```javascript
mutations: {
  increment(state) {
    state.count++;
  },
  setUsername(state, name) {
    state.username = name;
  }
}
```

你调用方式是：

```javascript
store.commit('increment')
store.commit('setUsername', '小红')
```

再讨论一下，类似于mutations:中的函数，这里面的state是把传进来的 `name` 这个值，存到全局状态树 `state` 的 `name` 字段里。

```
setUsername(state, name) {
    state.username = name;
  }
```

![7](C:\Users\30816\OneDrive\桌面\Springoot项目笔记\image\7.png)

### ✅ 3. `actions`

🔹 作用：**处理异步操作，比如发请求、定时器等，最终还是要通过 mutations 来改 state**

```javascript
actions: {
  asyncIncrement(context) {
    setTimeout(() => {
      context.commit('increment');
    }, 1000);
  }
}
```

调用方式：

```javascript
store.dispatch('asyncIncrement')
```

> ✅ `actions` 可以包含异步逻辑，`mutations` 必须同步，这是最重要的区别！

### ✅ 4. `modules`

🔹 作用：**把 store 拆成多个模块，方便管理和维护**

当你的 store 太大了，可以分模块，比如：

```javascript
modules: {
  user,
  game,
  record
}
```

然后你就可以访问：

```javascript
store.state.user.username
store.state.game.status
```

模块中的结构和主 store 是一样的，也是有 `state`、`mutations`、`actions`。

# 9.vue中的生命周期

Vue 组件从创建 ➡️ 渲染 ➡️ 更新 ➡️ 销毁，会经历一系列“生命周期阶段”。Vue 提供了生命周期函数（钩子），让你可以**在合适的时机写逻辑代码**。在 Vue 3 的组合式 API 中，`onMounted` 和 `onUnmounted` 等等这些生命周期函数都**写在 `setup()` 里或 `<script setup>` 语法糖中**。有的setup()函数中，没有`onMounted`、`onUnmounted` 的 `<script setup>` 其实是因为**不是每个组件都需要用到生命周期钩子**。

简单来说，**“打开当前页面”** → 就会触发当前页面组件的挂载 → 执行 `onMounted()` 里的代码。 **你离开页面/路由切换了**：`onUnmounted()` 被执行。

我们只需要关注onMounted()和onUnmounted()两个钩子函数即可

### 🔹 只要这个组件（GameMap.vue）被挂载到页面上，`onMounted()` 就会被自动调用！

------

## 📌 什么叫 “挂载”？

“挂载”就是：Vue 把这个组件渲染出来，放到页面里。比如：

```html
<!-- 页面结构 -->
<template>
  <GameMap />   <!-- ⬅️ 当这一行执行，组件就“挂载”了 -->
</template>
```

## ✅ `onMounted()` 的作用

Vue 的生命周期钩子之一，用于**在 DOM 元素都创建完成之后执行初始化逻辑**。

```javascript
import { onMounted } from 'vue'

onMounted(() => {
  // 这里写的代码会在模板中的 DOM 元素都准备好之后再执行
})
```

流程如下：

1. Vue 渲染页面，把 `<canvas>` 和 `<div>` 插入 DOM
2. Vue 调用 `onMounted()` 回调
3. `canvas.value` 和 `parent.value` 都能访问到真实 DOM
4. 用 canvas 拿到绘图上下文 ctx（画笔）
5. 创建 `GameMap` 类实例，开始绘图！

如果不写onMounted()就直接调用new GameMap(canvas.value.getContext('2d'), parent.value);，那么此时Vue **还没把 `<template>` 里的 DOM 节点渲染出来，`canvas.value` 和 `parent.value` 都是 `null`！**正确写法是用 `onMounted` 等待 DOM 渲染完成，再继续进行onMounted()里面的内容。

Vue 的组件渲染是**异步的**，它先执行 `setup()`，再去渲染 DOM。所以你在 `setup()` 里同步访问 DOM，基本都会是 `null`。

而 `onMounted()` 是 Vue 提供的生命周期钩子，它专门用来执行**“DOM 渲染完成后”的逻辑**，是画图、操作 DOM、注册事件的首选位置。

## ✅ `onUnmounted()` 的作用

`onUnmounted()` 是为了在组件销毁前**清理副作用**，防止内存泄漏、逻辑冲突和奇怪的 bug。例如，1.定时器 页面切换了还在执行 → 多个定时器重叠，浪费性能。 
`onUnmounted()` 是 Vue 提供的“清理工”，只要你在组件中创建了**定时器、监听器、图形、外部资源**等非响应式内容，**都应该用它来清除！**

**什么时候清理副作用？** —— 当你**知道这个组件会“离开视图”、“不再使用”时**，就必须清理掉它创建的副作用。

最稳妥的方式：**你只要在 `onMounted()` 里“绑定/创建了某些功能”，就在 `onUnmounted()` 里“解绑/销毁它”。**

# 10.匿名函数，js一些基础

首先看一下基础变换，包括有参和无参的。

![8](C:\Users\30816\OneDrive\桌面\Springoot项目笔记\image\8.png)

 **JavaScript 把“函数”当成一种值** —— 就像数字、字符串、对象一样，**函数也是“一等公民”**，所以你可以：

### 1.把它赋值给变量

```javascript
// 把函数当作值赋给变量
const sayHi = function() {
    console.log("Hi!");
};

// 调用这个变量，其实就是在调用函数
sayHi(); // 输出：Hi!
```

和我们平时写的这种是等价的：

```javascript
function sayHi() {
    console.log("Hi!");
}
```

### 2.把它作为参数传进其他函数

```javascript
function sayHello() {
    console.log("Hello!");
}

function callFn(fn) {
    console.log("准备调用函数：");
    fn(); // 调用传进来的函数
}

callFn(sayHello); // 👈 把函数当参数传进去
```

```
准备调用函数：
Hello!
```

### 3.甚至把它当作返回值返回出去

```javascript
function createGreeter(name) {
    return function() {
        console.log("Hello, " + name + "!");
    }
}

const greetTom = createGreeter("Tom");
const greetJerry = createGreeter("Jerry");

greetTom();   // Hello, Tom!
greetJerry(); // Hello, Jerry!
```

# 10.vue框架顺序的一个梳理

![9](C:\Users\30816\OneDrive\桌面\Springoot项目笔记\image\9.png)

1. **编译阶段（构建时）**
   - Vue Loader 会把 `<template>`、`<script>`、`<style>` 分开编译
   - 样式会被提取出来插入页面 `<head>` 中（变成 `<style>` 标签）
2. **运行时**
   - 加载组件 → 执行 `<script>` → 初始化数据、方法等
   - 然后使用这些数据渲染 `<template>`
   - 样式早已生效，不影响 JS 执行

对于``<style>``模块，**在用户看到界面之前，CSS 就已经被加进 HTML 的 `<head>` 了**。你哪怕还没点按钮，还没执行任何 JS，浏览器看到这个 `<style>` 就已经能用它来渲染页面了。

对于变化的数据来说，写在setup()中的：

`setup()` 并不是在每次点击按钮时重新执行一遍，而是**在组件创建时执行一次**，然后返回的响应式对象会**持续监听变化**，影响 `<template>` 的显示。

- 它返回的所有数据（比如 `route_name`, `logout`）如果是响应式的（比如 `computed`、`ref`、`reactive`）；
- 那么 Vue 就会建立起这些数据的“依赖追踪链”；
- 只要依赖的数据变了，对应的视图就更新，无需重新运行 `setup()`。

#11.前端和后端交互的url是怎么对应的？

### 前端发送 AJAX 时，后端是**怎么知道**对应哪个代码来处理请求的？

**后端框架会根据你配置的路由映射，自动把 URL 分发给对应的方法，而不是全局搜索文件。**

Spring Boot会在**项目启动时扫描所有带有注解（如 @PostMapping）的类和方法**，然后：

1. 把每个 URL 和它对应的 Java 方法保存到一个**映射表**（叫做“路由映射表”）。
2. 当前端发起 AJAX 请求时，框架就根据请求的 URL 和请求方法（GET/POST 等）**查找对应的处理函数**，然后调用它。

# 12.前后端交互的一个总结

整个系统的通信分两个阶段：

### 🟡 第一阶段：玩家尚未开始对战（全部使用 **HTTP 请求**）

- 所有按钮点击、注册、登录、查看排行榜、请求匹配 等操作，都是标准的 `HTTP 请求`。
- 后端通过 RESTful 接口处理请求，对应的就是我们前面说的 `Controller -> Service -> Mapper` 流程。

**ajax 是前端向后端链接的方式，后端用 @PostMapping 接应，return 返回值，前端用 resp 接收。**

如果代码里没有ajax，比如store.dispatch("login")，同理还有getinfo和logout这两个，都存在vuex中的store中的user.js中的actions中的函数中，写了ajax,如下。

![10](C:\Users\30816\OneDrive\桌面\Springoot项目笔记\image\10.png)

### 🟢 第二阶段：匹配成功，进入游戏后，前后端使用 **WebSocket 实时通信**

- 匹配成功后，服务器会通知前端连接 WebSocket。
- 此时，**每一帧的操作**（如移动方向）都会通过 WebSocket 发给后端。
- 后端用 `WebSocketServer.java` 来收发消息，维护两个玩家的连接状态。

**WebSocket 连接是由前端**主动发起建立的，类似代码：new WebSocket("wss://你的后端地址/...");

✅ **前端用 `socket.send()` 发消息 → 后端用 `@OnMessage` 接收**
✅ **后端用``sendMessage()``函数中包含的 `session.getBasicRemote().sendText()` 发消息 → 前端用 `socket.onmessage` 接收**

注意前端发消息的 `socket.send()` 是从代码中template中的PlayGround.vue模块中的GameMap.vue中的GameMap.js文件中有了send函数。

# 13.GmaeMap.vue和GameMap.js和PlayGround.vue的区别

其实不写GameMap.js也可以，直接把GameMap.js中的逻辑全写到GmaeMap.vue的<script></script>里面也行，但是这样子太乱了。所以我们把游戏逻辑和 Vue UI 分离开来，做到**“前端视图”**和**“游戏逻辑”**的职责分离。**这是一种前后端分层的思想：UI 层 和 数据逻辑层分离！**

| Vue 文件中做的事             | JS 文件中做的事                              |
| ---------------------------- | -------------------------------------------- |
| 负责 DOM、布局、UI、组件管理 | 负责游戏逻辑、坐标计算、物体移动、碰撞检测等 |

总的来说：

`GameMap.vue` 是 UI 组件，负责页面结构和渲染，
`GameMap.js` 是逻辑类，封装地图的运行逻辑、控制地图、蛇、游戏规则等。

![11](C:\Users\30816\OneDrive\桌面\Springoot项目笔记\image\11.png)

因为GameMap.js是继承那个基类AcGameObject.js的，GameMap.js` 可以给多个 `.vue 页面用

```javascript
GameMap.vue       // UI层：负责渲染 canvas 标签、布局
  ↳ 引用
    GameMap.js    // 逻辑层：负责处理地图坐标、渲染动画、控制物体行为
      ↳ 继承
        AcGameObject.js // 通用的生命周期框架：start(), update(), destroy()
```

再来看GmaeMap.vue和PlayGround.vue的关系，这是个很经典、合理的结构设计问题，涉及 **Vue 组件拆分** 与 **职责分离** 的思想。

**问题：为什么有了 `PlayGround.vue`，还要写一个 `GameMap.vue` 呢？是不是多此一举？**

**答案：不是多此一举，而是职责划分更清晰、扩展性更强。**

## ✅ 来看这两个组件到底各自负责什么：

### 🔷 `PlayGround.vue`

```vue
<template>
  <div class="playground">
    <GameMap />
  </div>
</template>
```

- 它是一个**容器型组件**
- 未来可能放：
  - `GameMap`（地图）
  - `ScoreBoard`（记分板）
  - `TimerBar`（倒计时）
  - `ControlPanel`（操作按钮）

✅ 换句话说，**它的职责是组织多个子组件的布局**，而不是具体干某一件事。

### 🔷 `GameMap.vue`

```vue
<template>
  <div class="gamemap">
    <canvas ref="canvas"></canvas>
  </div>
</template>
```

- 它是一个**功能型组件**
- 它的职责是渲染 canvas 地图，控制游戏画面
- 内部会使用 `GameObject.js` 等脚本逻辑

✅ 也就是说，**它只关心“画地图”这一件事**，不关心布局或别的 UI。



## ✅ 为什么要这么拆？

1. **职责分离**：每个组件只关心自己的部分，**便于维护**

2. **易于复用**：`GameMap.vue` 可以被别的地方复用

3. **组件更清爽**：每个 `.vue` 文件变得更短、更专注

4. **方便多人协作开发**：你写 PlayGround，别人写 GameMap，互不干扰

   

## ✅ 未来可能的结构

```bash
components/
├── PlayGround.vue      # ✅ 容器组件，放地图、计分板、控制器等
│   ├── GameMap.vue     # 地图画布（canvas）
│   ├── ScoreBoard.vue  # 分数显示
│   ├── Timer.vue       # 倒计时
│   └── ControlPanel.vue# 操作按钮、开始、暂停等
```

# 14.user.js和pk.js的区别

两个里面都有id属性，其中：

- `user.id` 是你当前登录用户的 ID
- `pk.a_id` 和 `pk.b_id` 是当前对局中两个玩家的 ID

这就是为什么能进行对比，你要判断一下：当前我是不是 A 玩家？（显示“左下角”）——所以需要比一下当前用户 ID 和对局玩家 ID。就可以显示是左下角用户还是右上角用户。

![12](C:\Users\30816\OneDrive\桌面\Springoot项目笔记\image\12.png)
