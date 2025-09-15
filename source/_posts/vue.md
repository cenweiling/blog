---
title: Vue
date: 2024-02-07
tags: [Vue]
categories: Vue
---

# 一个重要的内置关系：
**VueComponent.prototype.__proto__=Vue.prototype**

**让组建实例对象（vc）可以访问到 Vue 原型上的属性、方法**

![vm、vc的原型关系](images/vue_prototype.png)



# 关于不同版本的Vue：
**<font style="background-color:#E7E9E8;"> vue.js </font>** **与 <font style="background-color:#E7E9E8;"> </font>**<font style="background-color:#E7E9E8;">vue.runtime.xxx.js </font> 的区别：

1. **<font style="background-color:#E7E9E8;">vue.js</font>** 是完整版的vue。包含：核心功能+模板解析器。
2. **<font style="background-color:#E7E9E8;">vue.runtime.xxx.js</font>** 是运行版的Vue。只包含：核心功能，没有模板解析器。

因为 **<font style="background-color:#E7E9E8;"> vue.runtime.xxx.js </font>** 没有模板解析器，所以 **<font style="background-color:#E7E9E8;"> main.js（入口文件）</font>** 创建 **<font style="background-color:#E7E9E8;"> vue实例（new Vue(...)）</font>** 时不能使用 **<font style="background-color:#E7E9E8;"> template</font>** 配置项，需要使用 **<font style="background-color:#E7E9E8;"> render</font>** 函数接收到的 **<font style="background-color:#E7E9E8;">  createElement </font>** 函数去指定具体内容。而 **<font style="background-color:#E7E9E8;"> .vue文件 </font>** 中的模板字符串语法 **<font style="background-color:#E7E9E8;"> <template></template> </font>** 是由 **<font style="background-color:#E7E9E8;"> vue-template-compiler </font>** 去解析的。

# Vue Cli脚手架
###### 查看配置
Vue脚手架隐藏了所有webpack相关的配置，若想查看具体的webpack配置，请执行：<font style="background-color:#E7E9E8;"> </font>**<font style="background-color:#E7E9E8;">vue inspect > output.js </font>**

会在项目根路径下输出一个 **output.js**文件，**注意：该文件仅供查看，修改无效。**

![vue-cli默认配置](images/vue-cli默认配置.png)

###### 修改配置
`vue.config.js` 是一个可选的配置文件，如果项目的 (和 `package.json` 同级的) 根目录中存在这个文件，那么它会被 `@vue/cli-service` 自动加载。可以对脚手架进行个性化定制，详情见vue-cli文档：[https://cli.vuejs.org/zh/config/](https://cli.vuejs.org/zh/config/)



# vue2与vue3
## 响应式实现：
[面试官的步步紧逼：Vue2 和 Vue3 的响应式原理比对](https://juejin.cn/post/7124351370521477128?searchId=20250818012335EC20BB0273AF51843CB1)**（超绝文章！醍醐灌顶！！**

vue2：**Object.defineProperty **遍历递归对象，改造属性getter、setter

vue3：**Proxy( + Reflect ) **原始类型：类的访问器属性（get value(...){}, set value(...){}）

| **特性** | **Vue 2** | **Vue 3** |
| --- | --- | --- |
| **实现方式** | Object.defineProperty | Proxy |
| **检测范围** | 仅能劫持已有属性 | 可拦截对象所有操作 |
| **数组处理** | 重写数组方法 | 原生支持数组操作 |
| **新增属性** | 需用Vue.set/$set/$delete | 自动检测 |
| **性能** | 递归遍历所有属性初始化 | 惰性代理，按需响应 |
| **嵌套对象** | 初始化时递归劫持 | 访问时递归代理 |


1. **数组监测**：
    - Vue 2中需要特殊方法触发更新
    - Vue 3中所有数组操作自动触发
2. **动态属性**：
    - Vue 2中需使用Vue.set
    - Vue 3中可直接赋值
3. **性能敏感场景**：
    - Vue 3的readonly/shallowRef/shallowReactive提供更细粒度控制

### 关于一些关键概念（vue2：Watcher / vue3：ReactiveEffect）
#### vue2：Watcher 
##### 一、Watcher 的三种类型与职责
Watcher 并不是单一功能的，根据其执行的任务不同，主要分为三种：

| 类型 | 触发场景 | 职责 | 代码中的体现 |
| --- | --- | --- | --- |
| **渲染 Watcher** | 每个组件实例只有一个 | **负责组件的视图更新**。当它依赖的数据变化时，会触发组件的重新渲染（`_update`<br/>）。 | 你写的模板中的每一个数据绑定 `{{ message }}`<br/>、`v-bind`<br/>，最终都依赖于它。 |
| **计算属性Watcher** | 每个 `computed`属性一个 | **监控计算属性所依赖的数据**，依赖变则重新计算计算属性的值，并缓存结果。 | 你在 `computed`<br/> 中定义的每一个函数，Vue 都会为其创建一个对应的 Watcher。 |
| **用户 Watcher** | 每个 `watch`选项一个 | **监听一个特定的数据变化，并执行用户定义的回调函数**。 |  |


##### 二、Watcher 的工作流程：与 Dep 和 Observer 联动
**Watcher** 不能单独工作，它必须和 **Observer**（数据劫持者）和 **Dep**（依赖管理器）协同工作。这三者的关系是 Vue 响应式的铁三角。

让我们用一个经典的流程图来揭示它们是如何协作的，**并以一个简单的模板 **`{{ user.name }}`** 为例**：

<img src="/images/Watcher工作流程.svg" alt="Watcher工作流程" height="auto" style="width:50%; display:block;">

:::tips
**关键步骤解读**：

1. **`Observer`**：通过 `Object.defineProperty` 将 `data` 中的每个属性（如 `user`）转换为 `getter` 和 `setter`。
2. **`Dep`**：每个被监听的属性都会拥有一个自己的 `Dep` 实例（依赖管理器），用来存储所有“依赖”这个数据的 `Watcher`。
3. **`Watcher` 的**创建： 当 Vue 初始化组件时，会创建一个**渲染 Watcher**。
    - 这个 Watcher 在执行它的第一个任务（即渲染页面）时，会去读取模板中用到的数据，如 `user.name`。
4. **依赖收集（Depend）**：
    - 读取 `user.name` 会触发之前定义的 `getter`。
    - 在 `getter` 中，会检查当前是否有正在执行的 Watcher（通过 `Dep.target` 静态属性指向它）。**有！** 就是刚才创建的渲染 Watcher（W1）。
    - `getter` 会调用 `dep.depend()`，将当前这个 Watcher（W1）**收集**到 `user.name` 的 Dep 的订阅者列表中。从此，Dep 就知道：“哦，W1 依赖我。”
5. **派发更新（Notify）**：
    - 当我们修改数据：`user.name = 'New'` 时，会触发 `setter`。
    - `setter` 会调用 `dep.notify()`。
    - `dep.notify()` 会遍历它收集的所有 Watcher（这里只有 W1），通知它们：“我变了，你们该干活了！”
6. **执行更新**：
    - 每个被通知的 Watcher（W1）会执行自己的 `update()` 方法。
    - 对于**渲染 Watcher**，这个“干活”就是重新执行组件的渲染函数（`render`），生成新的虚拟 DOM，然后进行 patch（Diff）更新真实 DOM。页面就这样更新了。

:::

#### vue3：ReactiveEffect
##### 一、什么是 ReactiveEffect？
它的核心思想来源于函数式编程中的 **“副作用”（Effect）** 概念。任何会“对外部世界”造成影响的操作都是副作用，例如：修改 DOM、发送网络请求、操作本地存储等。

在 Vue 的上下文中，**最常见的“副作用”就是“渲染视图”**。当响应式数据变化时，我们需要重新执行这个副作用来更新视图。

`ReactiveEffect` 就是一个**封装了这些副作用函数的对象**。它的职责和 Vue 2 的 `Watcher` 类似，但设计和实现更加优雅和强大。

**一个极简的 ReactiveEffect 示例：**

```javascript
import { reactive, effect } from 'vue'; // 注意：effect 在 Vue 3 中是一个底层 API

const state = reactive({ count: 0 });

// 创建一个 Effect：副作用是打印 count 的值
const myEffect = new ReactiveEffect(() => {
  console.log(`Count is: ${state.count}`);
});

// 首次手动执行副作用
myEffect.run(); // 输出: Count is: 0

// 当 state.count 变化时，这个 effect 会自动重新执行
state.count++; // 自动输出: Count is: 1
```

**关键点**：

1. 你创建一个 `ReactiveEffect` 实例，并传入一个**副作用函数**。
2. 当你执行 `effect.run()` 时，Vue 3 会：
    - 设置一个全局的“活动效应”（`activeEffect`）为当前这个 `ReactiveEffect`。
    - 然后执行你传入的副作用函数。
3. 在执行过程中，如果副作用函数**读取**了某个响应式数据（如 `state.count`），就会触发该数据的 `getter`。
4. 在 `getter` 中，Vue 3 会发现当前有一个 `activeEffect` 正在运行，于是就会**建立依赖关系**：将这个 `ReactiveEffect` 收集为这个响应式数据的依赖。
5. 将来该数据**变化**时，就会通知所有依赖它的 `ReactiveEffect` 实例，调用它们的 `run` 方法（或调度函数），从而重新执行副作用。

##### 二、ReactiveEffect 与 Vue 2 Watcher 的核心差异
虽然目标一致（追踪依赖、执行副作用），但两者在设计和能力上有天壤之别。

| 特性 | Vue 2 **Watcher** | Vue 3 **ReactiveEffect** |
| --- | --- | --- |
| **设计理念** | **“全能型员工”**   与组件实例、选项式 API 强耦合，种类繁多（渲染、计算、用户）。 | **“纯粹的工具”**   一个**与上下文无关**的、**单一职责**的副作用封装器。它不知道自己是用于渲染、计算还是监听。 |
| **与组件的关系** | **紧密耦合**   每个组件实例必然对应一个渲染 Watcher。Watcher 知道自己是哪个组件的。 | **松散耦合**   `ReactiveEffect`<br/> 本身不知道组件。组件的 `setup`<br/> 函数本身就是一个大的 `ReactiveEffect`<br/>。 |
| **依赖收集** | **显式、侵入式**   通过 `Dep`<br/> 类和 `pushTarget`<br/>/`popTarget`<br/> 等全局状态管理。 | **隐式、基于栈**   通过全局变量 `activeEffect`<br/> 和 `effectStack`<br/> 来追踪当前正在运行的 effect，更加清晰可靠。 |
| **功能与灵活性** | **功能固定**   种类和行为在创建时就确定了（如 `lazy`<br/>, `sync`<br/> 等配置）。 | **极其灵活**   通过 **`scheduler`**<br/> **调度器** 实现各种高级功能（如 `computed`<br/> 的懒计算、`watch`<br/> 的异步回调）。 |
| **性能优化** | **较差**   在依赖收集阶段需要频繁创建和遍历 Dep 实例。 | **更优**   使用 `Set`<br/> 和 `Map`<br/> 等原生数据结构管理依赖，效率更高。依赖关系更精细。 |


---

##### 三、最重要的差异：调度器 (scheduler)
这是 `ReactiveEffect` 相比 `Watcher` 最强大的设计优势。

在创建 `ReactiveEffect` 时，你可以传入一个 **`scheduler`** **函数**。当依赖变化时，**默认行为是直接调用**** `effect.run()`**，但如果你提供了 `scheduler`，则会**改为调用** **`scheduler(effect)`**，把**何时、如何执行副作用**的控制权完全交给了你。

**正是这个** **`scheduler`**，实现了 Vue 3 的所有高级特性：

**1. 实现** **`computed`**

```javascript
function computed(getter) {
  let dirty = true; // 标记是否需要重新计算
  let value;

  // 1. 创建一个“懒执行”的 effect
  const effect = new ReactiveEffect(getter, () => {
    // 2. 这个 scheduler 只标记 dirty，但不立刻计算值
    if (!dirty) {
      dirty = true;
      // 3. 通知依赖此 computed 的 effect 更新（例如模板）
      trigger(computedRef, TriggerOpTypes.SET, 'value');
    }
  });

  const computedRef = {
    get value() {
      if (dirty) {
        value = effect.run(); // 只有在需要时才执行计算
        dirty = false;
      }
      track(computedRef, TrackOpTypes.GET, 'value'); // 收集依赖
      return value;
    }
  };
  return computedRef;
}
```

**2. 实现** **`watch`** / **`watchEffect`**

```javascript
function watch(source, cb, options) {
  let getter = () => source();
  let oldValue;

  // scheduler 决定回调的执行时机（同步、异步、前置等）
  const scheduler = () => {
    if (options.flush === 'sync') {
      // 同步执行
      run();
    } else {
      // 异步执行（默认是 'pre'，在组件更新前）
      queueJob(run);
    }
  };

  const effect = new ReactiveEffect(getter, scheduler); // 传入 scheduler!

  const run = () => {
    const newValue = effect.run();
    cb(newValue, oldValue); // 执行用户回调
    oldValue = newValue;
  };

  effect.run(); // 首次运行
}
```

**3. 实现组件的异步更新队列**

组件的渲染本身也是一个 `ReactiveEffect`。它的 `scheduler` 是一个将渲染任务推入**异步队列**的函数，这才实现了 Vue “批量更新”和“异步更新”的特性。

javascript

```plain
// 组件更新 effect
const updateEffect = new ReactiveEffect(
  componentUpdateFn, // 副作用：组件渲染函数
  () => queueJob(updateEffect) // scheduler: 不是立即执行，而是放入队列
);
```

---

##### 总结：演进与优势
|  | Vue 2 **Watcher** | Vue 3 **ReactiveEffect** |
| --- | --- | --- |
| **定位** | 一个与组件生命周期绑定的**具体概念** | 一个抽象的、可复用的**副作用容器** |
| **设计** | **“大而全”**，多种类型，逻辑复杂 | **“小而美”**，职责单一，通过**调度器**注入不同行为 |
| **能力** | 功能固定，扩展性差 | **极其灵活**，通过 `scheduler`<br/> 可实现各种高级异步和控制流程 |
| **关系** | **继承与分类**（RenderWatcher, ComputedWatcher...） | **组合与赋能**（一个 Effect + 不同的 scheduler = 不同功能） |


**结论：**  
`ReactiveEffect` 是 Vue 3 对 `Watcher` 概念的**一次彻底的重构和升级**。它通过**分离“副作用本身”和“副作用的执行时机（调度）”**，将响应式系统从一个相对僵化的模型解放为一个极其灵活和强大的架构。这种设计不仅使代码更清晰、更易于维护，更重要的是为 `computed`、`watch` 以及未来的所有响应式高级功能提供了一个统一而强大的底层基础。

## Composition api (组合式api) 和 Options api (选项式api)
1. 代码组织方式改变
2. composition api优势
    - 更好的逻辑复用（自定义hooks）
    - 更灵活的代码组织
    - 更好的TypeScript支持
    - 更小的代码打包体积
1. 模板和组件变化
2. 多根节点支持
3. v-model升级：v-model:title="pageTitle"
4. 组件生命周期变化



## Fragment/Teleport/Suspense
+ Fragment：多根节点组件
+ Teleport：将内容渲染到DOM其他位置
+ Suspense：异步组件加载状态处理

# diff算法


