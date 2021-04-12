### 一轮

#### computed原理  怎么实现的
```js
<body>
  <div id="app">
    <p>原来的数据: {{ msg }}</p>
    <p>反转后的数据为: {{ reversedMsg }}</p>
  </div>
  <script type="text/javascript">
    var vm = new Vue({
      el: '#app',
      data: {
        msg: 'hello'
      },
      computed: {
        reversedMsg() {
          // this 指向 vm 实例
          return this.msg.split('').reverse().join('')
        }
      }
    });
  </script>
</body>
```
- computed怎么计算的？ 
    - Object.defineProperty来监听Vue实例上的　reversedMsg　属性. 然后会执行sharedPropertyDefinition中的get或set函数的。因此只要我们的data对象中的某个属性发生改变的话, 我们的reversedMsg方法中依赖了该属性的话, 也会调用sharedPropertyDefinition方法中的get/set方法的。
- 怎么缓存？
    - dirty标志是否需要重新计算；数据D的依赖收集器里面包含了watchC和watchP；当数据D 变化时会先将dirty置为true，然后通知这两个watch，于是更新computed C 和页面P
- 什么时候初始化？ 
    - 在创建Vue实例时，init() initState方法里initComputed
- 如何做到实例访问的？
    - Object.defineProperty()方法 将computed里面的computed挂载到实例上
- 页面第一次初始化的时候, 我们要如何初始化执行　computed中的对应方法呢？
    - this.lazy = false;不缓存结果标志，因此会执行 this.get()函数，该函数返回对应的value
- cdn为什么能使加载速度变快
    - 用户访问使用CDN服务的网站时，向本地DNS服务发起域名解析请求
    - DNS服务器通过CNAME方式将最终域名重定向到CDN服务器
    - CDN对域名进行智能解析，将响应速度最快的CDN节点的IP地址返回给本地DNS
    - 浏览器访问响应速度最快节点的IP地址，获取资源
- style-loader css-loader输入输出是什么   vue-loader输出是什么
    - sass-loader 输入是sass 输出是css
    - css-loader 输入是css  输出是css   css-loader只是找出css中依赖的资源、压缩css代码
    - style-loader 输入是css  输出是通过脚本加载的 JavaScript 代码
    - vue-loader 输入是Vue单文件组件  输出是js对象，主要包括render、components以及一些其他内容自动生成的函数。其中template(转成HTML)、script、style(注入到HTML里)是通过vue-compiler实现的。
- webpack如何优化打包速度和包体积
    - 优化打包速度
        - 合理利用缓存减少打包时间，如文件没有更改就使用缓存loader: 'babel-loader?chaheDirectory'或者公共不改动的库cache-loader
        - 合理的使用plugin（可以使用IgonrePlugin忽略插件的某个无用的文件夹）
        - 多进程打包（使用happy和使用thread-loader）
        - 热更新技术
    - 优化打包体积
        - treeshake剔除无用代码
        - 合理的使用plugin
- `https认证过程，握手过程`
    - 客户端发起https请求，连接到服务器的443端口
    - 服务器把自己的信息以数字证书的形式返回给客户端（证书内容有密钥公钥，网站地址，证书颁发机构，失效日期等）公钥客户端用来自己加密，私钥服务器持有
    - 验证证书的合法性：客户端收到响应后会验证证书中包含的地址和正在访问的地址是否一致
    - 生成随机密码：如果验证通过，或者用户接受了不受信任的证书，浏览器会生成一个随机对称的密钥，并用公钥加密，让服务端用私钥解密，解密后就用改密钥传输，服务端确实是私钥的持有者
    - 生成对称加密算法：验证完服务端身份后，客户端会生成一个对称加密的算法和密钥，密钥加密后发送给服务端。
- 合并两个有序数组
- 青蛙跳台阶问题
- 栈内存和堆内存
    - 栈内存线性有序存储，容量小，系统分配效率高。而堆内存首先要在堆内存新分配存储区域，之后又要把指针存储到栈内存中，效率相对就要低一些了
    - 基本数据类型在内存中分别占有固定大小的空间，他们的值保存在栈空间，我们通过按值来访问的。
    - 引用类型，值大小不固定，栈内存中存放地址指向堆内存中的对象。是按引用访问的。

- 讲讲xss和csrf， 即如何防御   为什么对脚本过滤可以防止攻击
- html2canvas为什么会出现白屏，代码角度分析
- 拥塞控制
- 策略模式 工厂模式

- computed原理  怎么实现的
- style-loader css-loader输入输出是什么   vue-loader输出是什么？输出的js文件里面具体内容结构
- webpack如何优化打包速度和包体积
- cdn为什么能使加载速度变快
- https认证过程
- 合并两个有序数组
- 青蛙跳台阶问题
- 栈内存和堆内存

- 讲讲xss和csrf， 即如何防御   为什么对脚本过滤可以防止攻击
- html2canvas为什么会出现白屏，代码角度分析
- 拥塞控制
- 策略模式 工厂模式


2021.3.23
### 代码执行顺序
```js
new Vue({
    data() {
        return {
            list: []
        }
    },
    methods: {
        a() {
            this.list[0] = 1
        },
        b() {
            this.list.push(2)
            this.list.push(3)
            this.list.push(4)
        },
        c() {
            this.list.fliter(value => value > 2)
        },
        d() {
            this.list.length = 2
        },
        e() {
            Array.prototype.slice.call(this.list, 0, 1)
        },
        h() {
            this.list.pop()
        }
    }
})
```

- 平时使用的是Vue哪个版本
```js
vue.js 

vue.common.js 

vue.esm.js 

vue.esm.browser.js

vue.runtime.js 

 vue.runtime.common.js

 vue.runtime.esm.js

```
- 模拟实现 Promise.allSettled
```js
模拟实现一个 Promise.allSettled



Promise.allSettled([
  Promise.resolve(33),
  new Promise(resolve => setTimeout(() => resolve(66), 0)),
  99,
  Promise.reject(new Error('an error'))
])
.then(values => console.log(values));

// [
//   {status: "fulfilled", value: 33},
//   {status: "fulfilled", value: 66},
//   {status: "fulfilled", value: 99},
//   {status: "rejected",  reason: Error: an error}
// ]
```
- 实现一个迭代器，可以迭代出指定年的每个周末时间

```js
const getDateIterator = // 
const dateIterator = getDateIterator(2021)

for(let date of dateIterator) {
    console.log(date) // 2021.1.2
    console.log(date) // 2021.1.3
}
```
- 实现一个 Proxy 代理对象，可以做到对于任意深度的读取对象，都不会报错，对于任意深度的赋值都可以成功

```js
var a = {}

const proxy = // TODO

var b = proxy(a)

console.log(b.a.c.s) // undefined

b.a.c.s = 1

console.log(a.a.c.s) // 1

```
- 给定一个整数数组 nums ，找到一个具有最大和的连续子数组（子数组最少包含一个元素），返回其最大和。

```js
输入：nums = [-2,1,-3,4,-1,2,1,-5,4]
输出：6
解释：连续子数组 [4,-1,2,1] 的和最大，为 6 。
```