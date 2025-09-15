---
title: 关于eventloop
date: 2023-10-08
tags: [eventloop, js基础, async/await案例题]
categories: eventloop
---

一些参考：

测试eventloop地址：[https://www.jsv9000.app](https://www.jsv9000.app)

参考视频：[【事件循环】【前端】事件原理讲解](https://www.bilibili.com/video/BV1K4411D7Jb/?spm_id_from=333.337.search-card.all.click&vd_source=a1a75df55f32a09493aa84363d0e2aa0)

## 浏览器的事件循环机制（EventLoop）
概述：

要真正理解事件循环，我们需要先了解浏览器的多进程架构：

+ **浏览器主进程**：负责界面显示、用户交互
+ **GPU进程**：处理图形渲染
+ **网络进程**：处理网络请求
+ **渲染进程**（核心）：每个标签页一个渲染进程，包含：
    - **主线程**：<u>执行JS、解析HTML/CSS、布局、绘制（就是我们常说的JS线程）</u>
    - **合成线程**：负责图层分割
    - **光栅线程**：将图层转换为像素

> <font style="background-color:rgba(77, 208, 225, 0.15);">关键点：JS引擎（如V8）只是渲染进程的一部分，JS的"单线程"指的是主线程的单线程，浏览器整体是多线程的。</font>
>



关于浏览器渲染机制的笔记，可以查看：[浏览器渲染机制](https://www.yuque.com/u54400072/kgraay/gyclhvv7y5wogkf0)



### JavaScript的单线程特性与异步机制
#### JS为什么是单线程的？
JavaScript被设计成单线程的，主要是为了避免DOM操作的复杂性。如果JavaScript是多线程的，那么当多个线程同时操作同一个DOM元素时，就会出现竞态条件（Race Condition），导致不可预测的结果。例如，一个线程要删除某个DOM元素，另一个线程要修改它，那么到底应该以哪个线程的操作为准呢？为了避免这种复杂性，JavaScript从诞生之初就被设计为单线程。

#### 单线程带来的问题与解决方案
（1）阻塞问题

+ 单线程意味着所有任务需按顺序执行，若某个任务耗时过长（如复杂计算或网络请求），会阻塞后续代码执行，导致页面卡顿
+ 解决方案：
    - **异步编程模型**：通过回调函数、Promise、async/await 处理异步操作，避免阻塞主线程

```javascript
// 示例：使用 Promise 处理异步请求
fetch('https://api.example.com/data')
  .then(response => response.json())
  .then(data => console.log(data))
  .catch(error => console.error(error));
```

    - **Web Worker**：将耗时任务交给后台线程处理，不阻塞主线程

```javascript
// 主线程
const worker = new Worker('worker.js');
worker.postMessage('开始计算');
worker.onmessage = (e) => {
  console.log('计算结果:', e.data);
};

// worker.js（独立线程）
onmessage = (e) => {
  const result = performHeavyCalculation();
  postMessage(result);
};
```

（2）I/O密集型场景的优化

+ 浏览器中的 JavaScript 主要处理 I/O 密集型任务（如网络请求、文件操作），单线程模型配合异步 I/O 机制（如事件循环）可高效处理这类场景。
+ **事件循环（Event Loop）**：
    - JavaScript 通过事件循环机制处理异步任务，将耗时操作放入任务队列，主线程空闲时再处理这些任务。



### 事件循环的完整运行机制：
#### 核心组件详解
（1）调用栈（Call Stack）

+ 本质：记录函数调用的数据结构（LIFO栈），先进后出，如下图所示：

![调用栈callstack](images/callstack.webp)

+ 特点：
    - **合成线程**：每次函数调用都会创建新的栈帧（包含参数、局部变量等）
    - 栈溢出：当递归深度超过最大调用栈大小（Chrome约1万层）

```javascript
// 栈溢出示例
function stackOverflow() {
  stackOverflow()
}
stackOverflow() // Uncaught RangeError: Maximum call stack size exceeded
```



（2）堆内存（Heap）

+ 存储引用类型（对象、数组等）的内存区域
+ 与栈的区别：
    - 栈：自动分配固定大小内存（基础类型、指针）
    - 堆：动态分配内存，需要垃圾回收

（3）任务队列系统

| **队列类型** | **触发方式** | **优先级** |
| --- | --- | --- |
| 微任务队列 | JS引擎直接管理 | 高 |
| 宏任务队列 | 由浏览器宿主环境管理 | 低 |
| 动画回调队列 | requestAnimationFrame | 特殊 |
| 空闲回调队列 | requestIdleCallback | 最低 |


#### 完整事件循环流程
下图为主线程中事件循环的运行示例图：

![主线程中的eventloop](images/主线程中的eventloop.png)

<img src="/images/eventloop流程.png" alt="eventloop流程" height="auto" style="width:40%; display:block;">

##### 宏任务【Macro-tasks】与微任务【micro-tasks】
宏任务大概包括：

+ script(整体代码)
+ setTimeout
+ setInterval
+ setImmediate
+ I/O（例如网络请求 **<font style="background-color:rgba(77, 208, 225, 0.08);">ajax</font>**、文件读写）
+ UI render
+ **<font style="background-color:rgba(77, 208, 225, 0.08);">setImmediate</font>** (Node.js环境特有)

微任务大概包括：

+ **<font style="background-color:rgba(77, 208, 225, 0.08);">Promise.then()</font>**、**<font style="background-color:rgba(77, 208, 225, 0.08);">Promise.catch()</font>**、**<font style="background-color:rgba(77, 208, 225, 0.08);">Promise.finally()</font>**
+ **<font style="background-color:rgba(77, 208, 225, 0.08);">process.nextTick</font>** (Node.js环境特有)
+ **<font style="background-color:rgba(77, 208, 225, 0.08);">MutationObserver</font>** (h5新特性 用于监听DOM变化)

##### EventLoop的执行顺序详解：
1. **执行同步代码：** 当JavaScript代码开始执行时，会首先执行所有的同步代码。这些同步代码可以被看作是当前宏任务的一部分。在执行过程中，如果遇到异步任务（无论是宏任务还是微任务），就会将其对应的回调函数放入相应的任务队列中。
2. **清空微任务队列：** 当所有同步代码执行完毕后，Event Loop并不会立即去执行宏任务队列中的任务。它会优先检查并清空**微任务队列**。这意味着，所有在当前宏任务执行期间产生的微任务，都会在下一个宏任务开始之前被执行完毕。
3. **页面渲染（可选）：** 在微任务队列清空之后，如果浏览器判断有必要进行页面渲染（比如DOM结构发生了变化，或者需要更新UI），它就会进行一次页面渲染。这一步是可选的，浏览器会根据实际情况决定是否进行渲染。
4. **执行下一个宏任务：** 页面渲染完成后，Event Loop会从**宏任务队列**中取出一个任务来执行。这个任务执行完毕后，又会重复步骤2，检查并清空微任务队列，然后再次进行页面渲染（如果需要），接着再从宏任务队列中取出下一个任务……如此循环往复，直到所有任务执行完毕。

这个过程可以概括为：**一个宏任务执行完毕 -> 清空所有微任务 -> 页面渲染（如果需要） -> 执行下一个宏任务**。这个循环会一直持续下去，直到所有任务都执行完毕。



### 代码实战：
#### async/await在EventLoop中的表现：
##### 概述：async/await是js中用于处理异步操作的语法糖，基于promise+generator构建
+ async函数
    - async函数隐式返回一个Promise对象。如果函数返回一个值，该值会被包装为Promise（通过`Promise.resolve`）；如果函数抛出异常，则返回的Promise状态为`rejected`。

```javascript
async function foo(){
  return 'hello world!'
}

// 等同于
function foo() {
  return Promise.resolve('hello world!')
}
```

+ await表达式
    - `await`只能在async函数内部使用。
    - `await`后面可以跟一个Promise对象，它会暂停async函数的执行，等待Promise的状态变为`resolved`，然后返回结果值。
    - 如果`await`后面是一个非Promise的值，它会被立即转换为一个已解决的Promise。
    - 如果Promise被拒绝（rejected），`await`会抛出拒绝的原因（可以使用try/catch捕获）

##### async函数在事件循环中造成的细微差别：
<img src="/images/async和微任务.png" alt="async和微任务" height="auto" style="width:60%; display:block;">

**重点：**

**根据规范，async函数返回一个Promise，并且await会暂停函数的执行，等待Promise解决，然后继续执行函数，并将后续代码放入微任务队列。**

**特别要注意的是：当async函数返回一个Promise时，会有额外的微任务产生（因为需要等待返回的Promise被解决，然后才能解决async函数自己的Promise）。**


<details class="lake-collapse"><summary id="uee28ecd3"><span class="ne-text">async async2()函数分析：</span></summary><p id="u6adc2781" class="ne-p"><span class="ne-text">执行同步代码：</span></p><ul class="ne-list-wrap"><ul ne-level="1" class="ne-ul"><li id="u93ecd645" data-lake-index-type="0"><span class="ne-text">输出 'script start'</span></li><li id="ub0b236ee" data-lake-index-type="0"><span class="ne-text">定义async1和async2（不执行）</span></li><li id="u83f602f8" data-lake-index-type="0"><span class="ne-text">调用async1()</span></li><li id="u3b88aae8" data-lake-index-type="0"><span class="ne-text">在async1中，调用async2()（因为await后面是async2()，所以先执行async2）</span></li></ul></ul><ol class="ne-ol"><li id="u901c7dd1" data-lake-index-type="0"><span class="ne-text">执行async2：</span></li></ol><ul class="ne-list-wrap"><ul ne-level="1" class="ne-ul"><li id="u7be6e3e7" data-lake-index-type="0"><span class="ne-text">输出 'async2 end'</span></li><li id="uf1064ee2" data-lake-index-type="0"><span class="ne-text">然后执行 return Promise.resolve().then(...)<br /></span><span class="ne-text">  这里，Promise.resolve()返回一个已解决的Promise，然后调用then方法，将then的回调（输出'async2 end1'）放入微任务队列。</span></li><li id="u13468c5f" data-lake-index-type="0"><span class="ne-text" style="background-color: #CEF5F7">注意：async2是一个async函数，它返回的Promise不是直接这个then返回的Promise，而是会额外包装一层。<br /></span><span class="ne-text" style="background-color: #CEF5F7">根据ECMAScript规范，async函数内部return一个值x，相当于执行</span><code class="ne-code"><span class="ne-text" style="background-color: #CEF5F7">Promise.resolve(x)</span></code><span class="ne-text" style="background-color: #CEF5F7">，然后会等待x（如果x是Promise）解决，再解决async函数返回的Promise。但是，如果x是一个Promise，那么就会产生两个微任务：一个用于等待x解决，另一个用于解决async函数的Promise。<br /></span><span class="ne-text" style="background-color: #CEF5F7">具体到async2：<br /></span><span class="ne-text" style="background-color: #CEF5F7">async2中：return Promise.resolve().then(...)<br /></span><span class="ne-text" style="background-color: #CEF5F7">这个then方法返回一个新的Promise（我们称为P1）。而async2函数会返回一个新的Promise（称为P2），P2的解决会等待P1的解决。所以，在P1解决后，才会解决P2，然后async1中的await才会继续。</span></li></ul></ul><ol start="2" class="ne-ol"><li id="u485bd34a" data-lake-index-type="0"><span class="ne-text">继续同步代码：</span></li></ol><ul class="ne-list-wrap"><ul ne-level="1" class="ne-ul"><li id="u29d7af0e" data-lake-index-type="0"><span class="ne-text">调用setTimeout，将回调放入宏任务队列（0毫秒后，但会在当前宏任务执行完后执行）</span></li><li id="ud7fba569" data-lake-index-type="0"><span class="ne-text">执行new Promise，输出 'Promise'，并立即resolve，将第一个then的回调（输出'promise1'）放入微任务队列</span></li><li id="u2c84e1e6" data-lake-index-type="0"><span class="ne-text">输出 'script end'<br /></span><span class="ne-text">此时，同步代码执行完毕。当前微任务队列中有两个任务（注意顺序）：</span></li><li id="u9f2acfbe" data-lake-index-type="0"><span class="ne-text">第一个是async2中放入的：输出'async2 end1'（来自P1的then回调）</span></li><li id="u5048b8b9" data-lake-index-type="0"><span class="ne-text">第二个是new Promise的then回调：输出'promise1'</span></li></ul></ul><ol start="3" class="ne-ol"><li id="u1fcaa51b" data-lake-index-type="0"><span class="ne-text">开始执行微任务队列：<br /></span><span class="ne-text">第一个微任务：执行async2中then的回调<br /></span><span class="ne-text">  输出 'async2 end1'<br /></span><span class="ne-text">  这个回调执行完毕，P1被解决（值为undefined，因为没有return）<br /></span><span class="ne-text"> </span><strong><span class="ne-text"> </span></strong><span class="ne-text" style="background-color: #CEF5F7">这时，因为async2返回的P2在等待P1解决，所以会安排一个微任务来解决P2（这是规范要求的，当async函数返回一个Promise时，需要等待这个Promise解决，然后才能解决async函数自己的Promise。这个等待过程会产生一个微任务）。</span><span class="ne-text"><br /></span><span class="ne-text">第二个微任务：执行输出'promise1'<br /></span><span class="ne-text">  输出 'promise1'<br /></span><span class="ne-text">  由于这个then回调返回undefined，所以它返回的Promise立即解决，于是下一个then的回调（输出'promise2'）被放入微任务队列。<br /></span><span class="ne-text">此时，微任务队列中新增了两个微任务（按顺序）：<br /></span><span class="ne-text">微任务3：解决async2返回的P2（这个微任务会触发async1中await后面的代码放入微任务队列）<br /></span><span class="ne-text">微任务4：输出'promise2'<br /></span><span class="ne-text">注意：微任务3是在执行第一个微任务（输出'async2 end1'）后产生的，所以它排在微任务4（由第二个微任务产生）之前。<br /></span><span class="ne-text">当前微任务队列：[微任务3, 微任务4]</span></li><li id="u213c16ff" data-lake-index-type="0"><span class="ne-text">继续执行微任务队列：<br /></span><span class="ne-text">执行微任务3：解决async2返回的P2<br /></span><span class="ne-text">此时，await async2()的Promise（即P2）被解决，然后await后面的代码（输出'async1 end'）被放入微任务队列。<br /></span><span class="ne-text">执行微任务4：输出 'promise2'<br /></span><span class="ne-text">此时，微任务队列中新增了一个微任务（输出'async1 end'）</span></li><li id="uae2fa8c4" data-lake-index-type="0"><span class="ne-text">继续执行微任务队列（新的一轮？不，微任务队列会一直执行直到清空，所以会继续）：<br /></span><span class="ne-text">微任务5：输出 'async1 end'</span></li><li id="ub048ce21" data-lake-index-type="0"><span class="ne-text">微任务队列清空，执行宏任务：<br /></span><span class="ne-text">  输出 'setTimeout'</span></li></ol></details>
###### 事件循环详细过程：

{% tabs 页面内不重复的ID %}
 
<!-- tab 左边流程 -->
 
阶段1：同步代码执行（宏任务）
同右
微任务队列：[ 回调A (async2 end1), 回调B (promise1) ]
阶段2：微任务执行（第一轮）
1. 执行回调A：
  ○ console.log('async2 end1') → 输出 "async2 end1"
  ○ 解析 P2（async2 内部 Promise）
  ○ await async2() 完成，将 async1 end 加入微任务队列（回调C）
微任务队列更新：
[ 回调B (promise1), 回调C (async1 end) ]
1. 执行回调B：
  ○ console.log('promise1') → 输出 "promise1"
  ○ 返回 undefined，自动创建新解析的 Promise P4
  ○ 将 promise2 回调加入微任务队列（回调C）
微任务队列更新：
[ 回调C (async1 end) , 回调D (promise2)]
阶段3：微任务执行（第二轮）
1. 执行回调C：
  ○ console.log('async1 end') → 输出 "async1 end"
2. 执行回调D：
  ○ console.log('promise2') → 输出 "promise2"
阶段4：宏任务执行
1. 执行 setTimeout 回调：
  ○ console.log('setTimeout') → 输出 "setTimeout"
 
<!-- endtab -->
<!-- tab 右边流程 -->
 
阶段1：同步代码执行（宏任务）
1. console.log('script start')
→ 输出 "script start"
2. 定义函数：
  ○ 定义 async1 和 async2（不执行函数体）
3. 调用 async1()：
  ○ 进入 async1，遇到 await async2()
  ○ 调用 async2()
4. 执行 async2()：
  ○ console.log('async2 end') → 输出 "async2 end"
  ○ return Promise.resolve().then(...)
创建 Promise P2 并将 .then 回调加入微任务队列（回调A）
5. setTimeout：
  ○ 将回调加入宏任务队列
6. new Promise：
  ○ console.log('Promise') → 输出 "Promise"
  ○ resolve() 立即解析 Promise P3
  ○ 将 .then 回调加入微任务队列（回调B）
7. console.log('script end')
→ 输出 "script end"
微任务队列：[ 回调A (async2 end1), 回调B (promise1) ]
阶段2：微任务执行（第一轮）
1. 执行回调A：
  ○ console.log('async2 end1') → 输出 "async2 end1"
  ○ 解析 P2（async2 内部 Promise）
  ○ 由于 async2 是 async 函数，需要额外微任务解析其返回的 Promise P1
微任务队列更新：
[ 回调B (promise1), 新任务 (解析P1) ]
1. 执行回调B：
  ○ console.log('promise1') → 输出 "promise1"
  ○ 返回 undefined，自动创建新解析的 Promise P4
  ○ 将 promise2 回调加入微任务队列（回调C）
微任务队列更新：
[ 解析P1, 回调C (promise2) ]
阶段3：微任务执行（第二轮）
1. 执行解析P1任务：
  ○ 解析 async2 返回的 Promise P1
  ○ 使 await async2() 完成，将 async1 end 加入微任务队列（回调D）
微任务队列更新：
[ 回调C (promise2), 回调D (async1 end) ]
1. 执行回调C：
  ○ console.log('promise2') → 输出 "promise2"
2. 执行回调D：
  ○ console.log('async1 end') → 输出 "async1 end"
阶段4：宏任务执行
1. 执行 setTimeout 回调：
  ○ console.log('setTimeout') → 输出 "setTimeout"




 
<!-- endtab -->
 
{% endtabs %}


  


##### await表达式在事件循环中，参考[async和await的面试题](https://www.bilibili.com/video/BV1x29iYdEQy/?spm_id_from=333.337.search-card.all.click&vd_source=a63e68ff2d1fc881d5dfb3ccab9ccf89)， 举例：
```javascript
  async function asy1(params) {
      console.log(1);
      await asy2();
      console.log(2);
    }

  const asy2 = async () => {
    await setTimeout(() => {
      Promise.resolve().then(() => {
        console.log(3);
      });
      console.log(4);
    }, 0);
  };

  const asy3 = async () => {
    Promise.resolve().then(() => {
      console.log(6);
    });
  };

  asy1();
  console.log(7);
  asy3();

// 输出顺序如下：
// 1
// 7
// 6
// 2
// 4
// 3
```

###### 分析如下：
<details class="lake-collapse"><summary id="ua9d4e59b"><span class="ne-text">执行流程：</span></summary><ol class="ne-ol"><li id="ua7bdc105" data-lake-index-type="0"><span class="ne-text">调用 </span><code class="ne-code"><span class="ne-text">asy1()</span></code><span class="ne-text">，进入</span><code class="ne-code"><span class="ne-text">asy1</span></code><span class="ne-text">函数：</span></li></ol><ul class="ne-list-wrap"><ul ne-level="1" class="ne-ul"><li id="ubff5558e" data-lake-index-type="0"><span class="ne-text">输出 </span><code class="ne-code"><span class="ne-text">1</span></code></li><li id="u11d6bd6e" data-lake-index-type="0"><span class="ne-text">调用 </span><code class="ne-code"><span class="ne-text">asy2()</span></code></li></ul></ul><ol start="2" class="ne-ol"><li id="ub1b11355" data-lake-index-type="0"><span class="ne-text">在 </span><code class="ne-code"><span class="ne-text">asy2</span></code><span class="ne-text"> 中：</span></li></ol><ul class="ne-list-wrap"><ul ne-level="1" class="ne-ul"><li id="u4398c32c" data-lake-index-type="0"><span class="ne-text">调用 </span><code class="ne-code"><span class="ne-text">setTimeout</span></code><span class="ne-text">，将回调函数放入宏任务队列（将在下一个宏任务执行）。</span></li><li id="u5f055c33" data-lake-index-type="0"><code class="ne-code"><span class="ne-text">await</span></code><span class="ne-text"> 后面是 </span><code class="ne-code"><span class="ne-text">setTimeout</span></code><span class="ne-text"> 返回的数字，所以会生成一个立即解决的Promise，并将 </span><code class="ne-code"><span class="ne-text">asy2</span></code><span class="ne-text"> 函数中 </span><code class="ne-code"><span class="ne-text">await</span></code><span class="ne-text"> 后面的代码（这里没有代码，所以实际上是等待这个Promise解决，然后 </span><code class="ne-code"><span class="ne-text">asy2</span></code><span class="ne-text"> 函数返回）包装成微任务，放入微任务队列。</span></li></ul></ul><ol start="3" class="ne-ol"><li id="u3a31576e" data-lake-index-type="0"><span class="ne-text">继续执行同步代码：</span></li></ol><ul class="ne-list-wrap"><ul ne-level="1" class="ne-ul"><li id="u6e2f88c4" data-lake-index-type="0"><span class="ne-text">输出 </span><code class="ne-code"><span class="ne-text">7</span></code></li><li id="u2b84c779" data-lake-index-type="0"><span class="ne-text">调用 </span><code class="ne-code"><span class="ne-text">asy3()</span></code><span class="ne-text">，在 </span><code class="ne-code"><span class="ne-text">asy3</span></code><span class="ne-text"> 中，</span><code class="ne-code"><span class="ne-text">Promise.resolve().then</span></code><span class="ne-text"> 将回调（输出6）放入微任务队列。</span></li></ul></ul><ol start="4" class="ne-ol"><li id="u10e27d36" data-lake-index-type="0"><span class="ne-text">同步代码执行完毕，开始执行微任务队列：</span></li></ol><ul class="ne-list-wrap"><ul ne-level="1" class="ne-ul"><li id="ucaa3f54f" data-lake-index-type="0"><span class="ne-text">微任务队列中有两个微任务：</span></li></ul></ul><ol class="ne-list-wrap"><ol class="ne-list-wrap"><ol ne-level="2" class="ne-ol"><li id="u8a3a55ba" data-lake-index-type="0"><span class="ne-text">由 </span><code class="ne-code"><span class="ne-text">asy2</span></code><span class="ne-text"> 中的 </span><code class="ne-code"><span class="ne-text">await</span></code><span class="ne-text"> 产生的微任务（表示 </span><code class="ne-code"><span class="ne-text">asy2</span></code><span class="ne-text"> 函数可以继续执行，实际上就是让 </span><code class="ne-code"><span class="ne-text">asy2</span></code><span class="ne-text"> 函数返回，并解决 </span><code class="ne-code"><span class="ne-text">asy1</span></code><span class="ne-text"> 中 </span><code class="ne-code"><span class="ne-text">await</span></code><span class="ne-text"> 所等待的Promise）。</span></li><li id="u19404ecd" data-lake-index-type="0"><span class="ne-text">由 </span><code class="ne-code"><span class="ne-text">asy3</span></code><span class="ne-text"> 放入的微任务（输出6）。</span></li></ol></ol></ol><ul class="ne-list-wrap"><ul ne-level="1" class="ne-ul"><li id="u8a4fffbd" data-lake-index-type="0"><span class="ne-text">微任务队列按顺序执行：</span></li></ul></ul><ul class="ne-list-wrap"><ul class="ne-list-wrap"><ul ne-level="2" class="ne-ul"><li id="u6d3cc76f" data-lake-index-type="0"><span class="ne-text">首先执行第一个微任务（来自</span><code class="ne-code"><span class="ne-text">asy2</span></code><span class="ne-text">的</span><code class="ne-code"><span class="ne-text">await</span></code><span class="ne-text">）：这个微任务的执行会解决</span><code class="ne-code"><span class="ne-text">asy2</span></code><span class="ne-text">函数返回的Promise，从而让</span><code class="ne-code"><span class="ne-text">asy1</span></code><span class="ne-text">函数中</span><code class="ne-code"><span class="ne-text">await</span></code><span class="ne-text">后面的代码（输出2）可以继续，但是注意，这个继续并不是立即执行输出2，而是将输出2的代码作为一个新的微任务加入微任务队列（因为async函数中，</span><code class="ne-code"><span class="ne-text">await</span></code><span class="ne-text">后面的代码总是被包装成微任务）。</span></li><li id="uab041688" data-lake-index-type="0"><span class="ne-text">然后执行第二个微任务：输出 </span><code class="ne-code"><span class="ne-text">6</span></code><span class="ne-text">。</span></li></ul></ul></ul><ol start="5" class="ne-ol"><li id="ucf226f8c" data-lake-index-type="0"><span class="ne-text">此时微任务队列中又有了一个新的微任务（输出2），所以接下来执行这个微任务：输出 </span><code class="ne-code"><span class="ne-text">2</span></code><span class="ne-text">。</span></li><li id="u6c503281" data-lake-index-type="0"><span class="ne-text">微任务队列清空，然后执行宏任务队列中的定时器回调：</span></li></ol><ul class="ne-list-wrap"><ul ne-level="1" class="ne-ul"><li id="uccbe3d59" data-lake-index-type="0"><span class="ne-text">在定时器回调中：</span></li></ul></ul><ul class="ne-list-wrap"><ul class="ne-list-wrap"><ul ne-level="2" class="ne-ul"><li id="u31c6a5c4" data-lake-index-type="0"><span class="ne-text">执行 </span><code class="ne-code"><span class="ne-text">Promise.resolve().then</span></code><span class="ne-text">，将输出3的回调放入微任务队列。</span></li><li id="u4f93746a" data-lake-index-type="0"><span class="ne-text">输出 </span><code class="ne-code"><span class="ne-text">4</span></code><span class="ne-text">。</span></li></ul></ul></ul><ul class="ne-list-wrap"><ul ne-level="1" class="ne-ul"><li id="u30f2e9ee" data-lake-index-type="0"><span class="ne-text">这个宏任务执行完毕，然后执行微任务队列：输出 </span><code class="ne-code"><span class="ne-text">3</span></code><span class="ne-text">。</span></li></ul></ul><h3 id="GAz0l"><span class="ne-text">重点：</span><code class="ne-code"><span class="ne-text">await</span></code><span class="ne-text"> 产生的微任务次数</span></h3><p id="ua93b7a29" class="ne-p"><span class="ne-text">在 </span><code class="ne-code"><span class="ne-text">asy2</span></code><span class="ne-text"> 函数中：</span></p><ul class="ne-ul"><li id="ua4ecaa40" data-lake-index-type="0"><code class="ne-code"><span class="ne-text">await</span></code><span class="ne-text"> 一个非Promise值（数字）会产生一个微任务（用于继续执行async函数后面的代码，即使后面没有代码，也需要解决async函数返回的Promise）。<br /></span><span class="ne-text">在 </span><code class="ne-code"><span class="ne-text">asy1</span></code><span class="ne-text"> 函数中：</span></li><li id="u15aad2bc" data-lake-index-type="0"><code class="ne-code"><span class="ne-text">await asy2()</span></code><span class="ne-text">：</span><code class="ne-code"><span class="ne-text">asy2</span></code><span class="ne-text"> 返回一个Promise，这个Promise的解决会在 </span><code class="ne-code"><span class="ne-text">asy2</span></code><span class="ne-text"> 函数内部的 </span><code class="ne-code"><span class="ne-text">await</span></code><span class="ne-text"> 完成时（即上面产生的微任务执行时）解决。然后，</span><code class="ne-code"><span class="ne-text">asy1</span></code><span class="ne-text"> 函数中 </span><code class="ne-code"><span class="ne-text">await</span></code><span class="ne-text"> 后面的代码（输出2）会被放入微任务队列。<br /></span><span class="ne-text">因此，整个流程中微任务队列的变化：</span></li></ul><ol class="ne-ol"><li id="u67f1aaae" data-lake-index-type="0"><span class="ne-text">同步代码执行完毕后，微任务队列有两个微任务：</span></li></ol><ul class="ne-list-wrap"><ul ne-level="1" class="ne-ul"><li id="uea94d940" data-lake-index-type="0"><span class="ne-text">第一个是 </span><code class="ne-code"><span class="ne-text">asy2</span></code><span class="ne-text"> 函数中 </span><code class="ne-code"><span class="ne-text">await</span></code><span class="ne-text"> 产生的（标记为微任务A）：用于解决 </span><code class="ne-code"><span class="ne-text">asy2</span></code><span class="ne-text"> 函数返回的Promise。</span></li><li id="u0ac362e1" data-lake-index-type="0"><span class="ne-text">第二个是 </span><code class="ne-code"><span class="ne-text">asy3</span></code><span class="ne-text"> 放入的（微任务B）：输出6。</span></li></ul></ul><ol start="2" class="ne-ol"><li id="u161f7bd5" data-lake-index-type="0"><span class="ne-text">执行微任务A：这会解决 </span><code class="ne-code"><span class="ne-text">asy2</span></code><span class="ne-text"> 函数返回的Promise，然后导致 </span><code class="ne-code"><span class="ne-text">asy1</span></code><span class="ne-text"> 函数中的 </span><code class="ne-code"><span class="ne-text">await</span></code><span class="ne-text"> 完成，从而将 </span><code class="ne-code"><span class="ne-text">asy1</span></code><span class="ne-text"> 函数中 </span><code class="ne-code"><span class="ne-text">await</span></code><span class="ne-text"> 后面的代码（输出2）作为新的微任务（微任务C）加入队列。</span></li><li id="u35263207" data-lake-index-type="0"><span class="ne-text">执行微任务B：输出6。</span></li><li id="u7a57f36d" data-lake-index-type="0"><span class="ne-text">然后执行微任务C：输出2。<br /></span><span class="ne-text">所以，</span><code class="ne-code"><span class="ne-text">asy2</span></code><span class="ne-text"> 函数中的 </span><code class="ne-code"><span class="ne-text">await</span></code><span class="ne-text"> 确实产生了一个微任务（微任务A），并且这个微任务的执行又导致了一个新的微任务（微任务C）产生。</span></li></ol><h3 id="OZ2IP"><span class="ne-text">结论</span></h3><p id="u030784a1" class="ne-p"><code class="ne-code"><span class="ne-text">asy2</span></code><span class="ne-text"> 函数中的 </span><code class="ne-code"><span class="ne-text">await</span></code><span class="ne-text"> 产生了微任务。整个代码中，微任务队列的执行顺序是：</span></p><ol class="ne-ol"><li id="u270ebb63" data-lake-index-type="0"><span class="ne-text">微任务A（来自</span><code class="ne-code"><span class="ne-text">asy2</span></code><span class="ne-text">的</span><code class="ne-code"><span class="ne-text">await</span></code><span class="ne-text">）：解决</span><code class="ne-code"><span class="ne-text">asy2</span></code><span class="ne-text">的Promise，触发</span><code class="ne-code"><span class="ne-text">asy1</span></code><span class="ne-text">中的</span><code class="ne-code"><span class="ne-text">await</span></code><span class="ne-text">完成，将输出2加入微任务队列（微任务C）。</span></li><li id="ufa55c812" data-lake-index-type="0"><span class="ne-text">微任务B（来自</span><code class="ne-code"><span class="ne-text">asy3</span></code><span class="ne-text">）：输出6。</span></li><li id="u671bd9d5" data-lake-index-type="0"><span class="ne-text">微任务C（来自</span><code class="ne-code"><span class="ne-text">asy1</span></code><span class="ne-text">的</span><code class="ne-code"><span class="ne-text">await</span></code><span class="ne-text">完成）：输出2。<br /></span><span class="ne-text">然后执行宏任务（定时器回调），在定时器回调中又产生了一个微任务（输出3），最后执行。</span></li></ol></details>
###### 上面的例子稍微变形后：
```javascript
async function asy1(params) {
    console.log(1);
    await asy2();
    console.log(2);
  }

const asy2 = async () => {
  await (async () => {
    await (() => {
      console.log(3);
    })();
    console.log(4);
  })();
};

const asy3 = async () => {
  Promise.resolve().then(() => {
    console.log(6);
  });
};

asy1();
console.log(7);
asy3();

// 输出顺序如下：
// 1
// 3
// 7
// 4
// 6
// 2
```

<details class="lake-collapse"><summary id="u058a4c63"><span class="ne-text">分析如下：</span></summary><ol class="ne-ol"><li id="uc339dd7e" data-lake-index-type="0"><span class="ne-text">声明asy1、asy2、asy3函数，不进入执行栈。</span></li><li id="u4182cade" data-lake-index-type="0"><span class="ne-text">调用 </span><code class="ne-code"><span class="ne-text">asy1()</span></code><span class="ne-text">，进入</span><code class="ne-code"><span class="ne-text">asy1</span></code><span class="ne-text">函数：</span></li></ol><ul class="ne-list-wrap"><ul ne-level="1" class="ne-ul"><li id="uc8b547f5" data-lake-index-type="0"><span class="ne-text">输出 </span><code class="ne-code"><span class="ne-text">1</span></code></li><li id="uc54df11b" data-lake-index-type="0"><span class="ne-text">调用 </span><code class="ne-code"><span class="ne-text">asy2()</span></code></li></ul></ul><ol start="3" class="ne-ol"><li id="u96ec74de" data-lake-index-type="0"><span class="ne-text">在 </span><code class="ne-code"><span class="ne-text">asy2</span></code><span class="ne-text"> 中：</span></li></ol><ul class="ne-list-wrap"><ul ne-level="1" class="ne-ul"><li id="ub8620ccd" data-lake-index-type="0"><code class="ne-code"><span class="ne-text">await (() =&gt; {console.log(3) })()</span></code><span class="ne-text"> 即</span><code class="ne-code"><span class="ne-text">await console.log(3)</span></code><span class="ne-text">最终会生成一个立即解决的Promise，即</span><code class="ne-code"><span class="ne-text">await Promise.resolve()</span></code><span class="ne-text">记为P1，该Promise解决后，将</span><code class="ne-code"><span class="ne-text">console.log(4);</span></code><span class="ne-text">（回调A）推入微任务队列。</span></li><li id="u47e33dc8" data-lake-index-type="0"><span class="ne-text">此时</span><code class="ne-code"><span class="ne-text">await (async () =&gt; {...)()</span></code><span class="ne-text"> 返回的Promise，记为P2，需要等待内部的</span><code class="ne-code"><span class="ne-text">console.log(4)</span></code><span class="ne-text">执行后才完成，所以该Promise挂起，</span><code class="ne-code"><span class="ne-text">asy1()</span></code><span class="ne-text">中暂停执行。</span></li></ul></ul><ol start="4" class="ne-ol"><li id="u5e47407d" data-lake-index-type="0"><code class="ne-code"><span class="ne-text">console.log(7)</span></code><span class="ne-text">执行，输出7。</span></li><li id="uaf22f070" data-lake-index-type="0"><span class="ne-text">调用 </span><code class="ne-code"><span class="ne-text">asy3()</span></code><span class="ne-text">，将</span><code class="ne-code"><span class="ne-text">console.log(6)</span></code><span class="ne-text">（回调B）加入微任务队列中，此时微任务队列：[回调A（console.log(4)），回调B（console.log(6)）]</span></li><li id="ueb91d798" data-lake-index-type="0"><span class="ne-text">执行微任务队列（阶段1）：</span></li></ol><ul class="ne-list-wrap"><ul ne-level="1" class="ne-ul"><li id="uef015d46" data-lake-index-type="0"><span class="ne-text">回调A调用，输出4。P2完成，</span><code class="ne-code"><span class="ne-text">await</span></code><span class="ne-text">将后续代码（没有）推入微任务队列，所以会产生一个微任务解析</span><code class="ne-code"><span class="ne-text">asy2()</span></code><span class="ne-text">的完成（微任务C）。</span></li><li id="u2643c2d9" data-lake-index-type="0"><span class="ne-text">回调B调用，输出6。此时微任务队列：[微任务C（asy2的完成）]</span></li></ul></ul><ol start="7" class="ne-ol"><li id="ue3844e51" data-lake-index-type="0"><span class="ne-text">执行微任务队列（阶段2）：</span></li></ol><ul class="ne-list-wrap"><ul ne-level="1" class="ne-ul"><li id="uee079303" data-lake-index-type="0"><span class="ne-text">回调C调用，</span><code class="ne-code"><span class="ne-text">asy1()</span></code><span class="ne-text">中</span><code class="ne-code"><span class="ne-text">await asy2()</span></code><span class="ne-text">完成，将后续代码（</span><code class="ne-code"><span class="ne-text">console.log(2)</span></code><span class="ne-text">回调D）推入微任务队列。</span></li><li id="u08fd80c5" data-lake-index-type="0"><span class="ne-text">执行回调D，输出2。</span></li></ul></ul><p id="u8e7806a1" class="ne-p"><br></p></details>
