明源云笔试
2020.6.29
1. 请按顺序写出控制台打印结果
image.png
User.action.getCount():this指向的是User.action对象，所以为undefined 
getCount()：函数作为对象的属性被赋值到另一个变量中调用时，this指向window
2. 用正则实现numSplit

3. 实现点击li弹出index


4. 实现自适应布局

使用order进行排序

5. 实现一个LazyMan
```js
function LazyMan(name){
    //由于直接通过_LazyMan("Hank")只是一个普通函数，无法通过this调用方法，所以这里返回一个new _LazyMan()
    return new _LazyMan(name);
}


function _LazyMan(name){
    this.tasks = [];    //任务队列
    this.hello(name);
    let that = this;
    setTimeout(() => {
        that.next()
    }, 0);
}
_LazyMan.prototype.next = function(){
    //console.log(this.tasks)
    let fn = this.tasks.shift();
    fn && fn();
}
//将要执行的内容封装到fn函数中，然后放入任务列表内
_LazyMan.prototype.hello = function(name){
    console.log("hello")
    let that = this;
    var fn = function(){
                console.log("Hi " + name);
                that.next();
            }


    this.tasks.push(fn);
    return this;
}
_LazyMan.prototype.eat = function(what){
    console.log("eat")
    let that = this;
    var fn = function(){
            console.log("Eat " + what);
            that.next();
        }
    this.tasks.push(fn);
    return this;
}
_LazyMan.prototype.sleep = function(s){
    console.log("sleep")
    let that = this;
    var fn = function(){
            setTimeout(() => {
                console.log("Wake up after " + s);
                that.next();
            }, s*1000);
        }
    this.tasks.push(fn);
    return this;
}
_LazyMan.prototype.sleepFirst = function(s){
    console.log("sleepFirst")
    let that = this;
    var fn = function(){
            setTimeout(() => {
                console.log("Wake up after " + s);
                that.next();
            }, s*1000);
        }
    this.tasks.unshift(fn);
    return this;
}
```
输出
image.png
2020.8.24
1. 和上面的第一题一样
2. 如何解决跨域问题
image.png
3. 将数字格式化为千分位（如将12345.11 => 12,345.11）
小数点以前的数处理方法：
image.png
完整代码
image.png
replace的第二个参数
image.png
4. 实现一个clone函数，可以对JavaScript中的5种主要数据类型进行复制
浅拷贝
image.png
深拷贝
image.png
5. 如何设置浏览器缓存，缓存与不缓存两种
服务器设置
expries: 0,
cache-control: no-cache
浏览器设置
```js
<!--设置过期时间设置0为直接过期并清除缓存-->
<meta http-equiv="Expires" content="0">

<!--设置不修改消息存储-->
<meta http-equiv="Cache-control" content="no-cache">

<!--http/1.0, pragma效果和cache-control一致-->
<meta http-equiv="Pragma" content="no-cache">
```
6.代码实现
image.png
实现
image.png
7. LazyMan，和上次的第五题一样
2021.4
1. 同"笔试题 2020.6.29"第四题，考察flex
2. css画扇形
image.png
  解答： 利用clip进行裁剪 + transform:rotate()来旋转
```css
.round {
  width: 200px;
  height: 200px;
  margin: 20px;
  border-radius: 50%;
  background: #00b4c8;
}
.round-one {
  clip-path: polygon(0% 0%, 25% 0%,50% 50%, 0% 25%);
}
.round-two {
  clip-path: polygon(0% 0%, 100% 0%, 100% 100%, 50% 100%, 50% 50%, 0% 25%);
}
.round-three {
  background: transparent;
  box-sizing: border-box;
  border: 30px solid #00b4c8;
  clip-path: polygon(0% 0%, 25% 0%,50% 50%, 0% 25%);
  animation: rotate 1s linear infinite;
}

@keyframes rotate {
  from{
    transform: rotateZ(0deg);
  } to{
    transform: rotateZ(360deg);
  }
}
       
//html
<div id="app">
  <div class="round round-one"></div>
  <div class="round round-two"></div>
  <div class="round round-three"></div>
</div>
```

3. 实现功能函数
```js
var add = function(num){
    return num
}
var mutiply = function(num){
    return -1 * num
}
var computed = function(...args){
    return function(num){
        console.log(args)
        return num + args.reduce((sum,val) => {
            return sum + val
        })
    }
}
var num1 = computed(add(1))(2);
var num2 = computed(add(1), mutiply(7))(2);
console.log(num1)
console.log(num2)
```

实现图片预加载列表
image.png
```js
var num = 0;
var loadPic = function(src){
  return new Promise((resolve, reject) =>{
    let img = new Image();
    img.onload = function(){
      num ++;
      resolve(img)
    }
    img.onerror = function(){
      resolve()
    }
    img.src = src;
  })
}
var preload = function(list, maxNum = 1){
  //获取promise数组
  var arr = list.map(val => {
    return loadPic(val)
  })
  Promise.all(arr).then(res => {
    var imgs = res;
    //将获取的图片显示出来
    imgs.forEach(item =>{
      document.getElementsByTagName("body")[0].appendChild(item)
    })
    return {
      success: num,
        fail: maxNum - num
    }
  })
}
var list = [
"https://www.baidu.com/img/flexible/logo/pc/result.png",
"./1.jfif",
];
preload(list, list.length)
```
5. 正则
image.png
```js
var url = "https://regex101.com/1dd/2/3";
var reg = /(?<=\/)[s\S]+?((?=\/)|$)/g
console.log(url.match(reg));
```
6. 实现Storage
```js
class Storage {
    constructor() {
        this.storageName = 'expiredStorage'
    }
    // name缓存名称
    // value缓存的值
    // expires缓存过期时间 单位秒
    /**
     *  设置缓存
     *  @param {string} name 缓存名称
     *  @param {any} value 缓存的值
     *  @param {any} expires 缓存过期时间(秒) 
    **/
    set(name, value, expires) {
      const storages ={}
      storages[name] = {
        value,
        expires: storages[name]
                ? storages[name].expires
                : expires === undefined
                ? +new Date() + 31536000000
                : expires * 1000 + +new Date(),
      };
      localStorage.setItem(this.storageName, JSON.stringify(storages))
    }
    get(name) {
      const storages = JSON.parse(localStorage.getItem(this.storageName))
      try {
        if (!storages[name]) {
            return null;
        }
        console.log('log=====',  storages[name].expires - new Date());
        if (+new Date() > storages[name].expires) {
            // 存在但过期
            this.remove(name);
            return null;
        }
        return storages[name].value;
      } catch (error) {
        console.log('[ControlStorage] the error message: get field failed\n', error);
      }
    }
    remove(name) {
      const storages = JSON.parse(localStorage.getItem(this.storageName))
      try {
        delete storages[name];
        if (JSON.stringify(storages) === '{}') {
          // 缓存字段为空对象时，删除该字段
          this.clear();
          return;
        }
        this.baseStorage.setItem(storages);
      } catch (error) {
        console.log('[ControlStorage] the error message: remove field failed\n', error);
      }
    }
    clear() {
      localStorage.removeItem(this.storageName)
    }
}

export default new Storage();
```