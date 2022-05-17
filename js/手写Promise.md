- Promise：Promise 对象用于表示一个异步操作的最终完成 (或失败)及其结果值。
  - 回调嵌套：最开始是用回调函数来解决这个问题，`避免回调地狱，推出了Promise来解决`
  - `信任问题(控制权反转再反转)：经常使用第三方API(如Ajax), 控制权交给第三方，出现回调函数调用过早过晚，次数过多，未调用`
- 实现Promise细节讲解
  - .then方法里 若状态值是pending的话，是考虑promise压根就没有决议或由于某些原因延迟决议，这种情况下方法内获取到的state状态值是pending；当状态值为pending时，可以先将onFulfilled、onRejected方法先保存起来，然后在构造函数里面决议(即调用resolve或reject方法)后再去调用；
  - resolvePromise：Promise解决过程是一个抽象操作，其需要输入一个promise和一个值，再加上我们拿到x值后还要处理，resolvePromise(promise2, x, resolve, reject);
  - `.then回调函数应该是异步执行，考虑用setTimeout(宏任务)去实现，浏览器中应当采用MutationObserver来产生微任务`
  - Promise解决过程函数resolvePromise实现细节：
    - x 与 promise 相等
    - x 为 promise （这条略过，因为任何具有.then方法的对象和函数都会被认为是Promise对象）
    - x 为对象或者函数，x为对象时， 以x为参数执行promise；x为函数时，
    - x 不为对象或函数，以x为参数执行promise
  - `then 方法中，创建并返回了新的 Promise 实例，这是串行Promise的基础，是实现真正链式调用的根本。`
  - `then 回调中没有resolve或者reject时 还是会继续执行下个.then回调，因为.then方法里根据各种情况调用了resolve或reject方法`
  - `Promise.then 为什么是微任务`：源码内部是用asap方法process.nextTick(/mutationObserver/messageChannel/setTimeout)封装的
  - `Promise中为什么使用微任务：由于Promise使用了延时绑定技术，所以在执行resolve函数的时候，回调函数还没有绑定，只能推迟回调函数的执行`，`如果使用定时器的话，定时器是个宏任务，效率不高，所以Promise为了提升代码执行效率，改造成了微任务`。
  - `Promise.then为什么是异步：因为回调是有副作用的，同步和异步共存时，没办法保证程序逻辑的一致性，导致回调调用时太早或太晚；同步的话多个Promise在多线程环境下的话构成了竞态，因此是异步`
- 总结：
  - promise是立即执行的，它在创建的时候会执行，不存在将promise推入微任务中的说法
  - resolve()是用来表示promise的状态为fullfilled，相当于只是定义了一个有状态的promise，但是并没有调用它
  - promise调用then的前提是promise的状态为fullfilled
  - 只有promise调用then的时候，then里面的函数才会被推入微任务中
    

```js
const STATUS = { PENDING: 'PENDING', FUFILLED: 'FUFILLED', REJECTED: 'REJECTED' }

// 我们的promise 按照规范来写 就可以和别人的promise公用
function resolvePromise(x, promise2, resolve, reject) {
  // 规范 2.3.1
  if (promise2 == x) { // 防止自己等待自己完成
    return reject(new TypeError('出错了'))
  }
  // 规范 2.3.3
  if ((typeof x === 'object' && x !== null) || typeof x === 'function') {
    // x可以是一个对象 或者是函数
    let called;
    try {
      // 规范 2.3.3.1
      let then = x.then;
      if (typeof then == 'function') {
        // 2.3.3.3
        then.call(x, function(y) {
          // 规范 2.3.3.3.3
          if (called) return
          called = true;
          // 规范 2.3.3.3.1
          resolvePromise(y, promise2, resolve, reject);
        }, function(r) {
          // 规范 2.3.3.3.3
          if (called) return
          called = true;
          // 规范 2.3.3.3.2
          reject(r);
          })
        } else {
          resolve(x); // 此时x 就是一个普通对象
        }
    } catch (e) {
      // 规范 2.3.3.3.4.1
      if (called) return
      called = true;
      // 规范 2.3.3.3.4 
      reject(e); // 取then时抛出错误了
    }
  } else {
    resolve(x); // x是一个原始数据类型 不能是promise
  }
  // 不是proimise 直接就调用resolve
}

class Promise {
  constructor(executor) {
    this.status = STATUS.PENDING;
    this.value = undefined;
    this.reason = undefined;
    this.onResolvedCallbacks = []; // 存放成功的回调的 
    this.onRejectedCallbacks = []; // 存放失败的回调的
    const resolve = (val) => {
      if(val instanceof Promise){ // 是promise 就继续递归解析
        return val.then(resolve,reject)
      }

      if (this.status == STATUS.PENDING) {
        this.status = STATUS.FUFILLED;
        this.value = val;
        // 发布
        this.onResolvedCallbacks.forEach(fn => fn());
      }
    }
    const reject = (reason) => {
      if (this.status == STATUS.PENDING) {
        this.status = STATUS.REJECTED;
        this.reason = reason;
       
        this.onRejectedCallbacks.forEach(fn => fn());
      }
    }

    try {
      executor(resolve, reject);
    } catch (e) {
      // 出错走失败逻辑
      reject(e)
    }
  }
   then(onFulfilled, onRejected) { // swtich  作用域
      // 可选参数
      onFulfilled = typeof onFulfilled === 'function' ? onFulfilled : x => x
      onRejected = typeof onRejected === 'function'? onRejected: err=> {throw err}
      let promise2 = new Promise((resolve, reject) => {
        if (this.status === STATUS.FUFILLED) {
          // to....
          setTimeout(() => {
            try {
              let x = onFulfilled(this.value);
              resolvePromise(x, promise2, resolve, reject)
            } catch (e) {
              reject(e);
            }
          }, 0);
        }
        if (this.status === STATUS.REJECTED) {
          setTimeout(() => {
            try {
              let x = onRejected(this.reason);
              resolvePromise(x, promise2, resolve, reject)
            } catch (e) {
              reject(e);
            }
          }, 0);
        }
        if (this.status === STATUS.PENDING) {
          // 装饰模式 切片编程
          this.onResolvedCallbacks.push(() => { // todo..
            setTimeout(() => {
              try {
                let x = onFulfilled(this.value);
                resolvePromise(x, promise2, resolve, reject)
              } catch (e) {
                reject(e);
              }
            }, 0);
          })
          this.onRejectedCallbacks.push(() => { // todo..
            setTimeout(() => {
              try {
                let x = onRejected(this.reason);
                resolvePromise(x, promise2, resolve, reject)
              } catch (e) {
                reject(e);
              }
            }, 0);
          })
        }
      });
    return promise2;
   }
}
              


Promise.all = function(arr) {
  return new Promise((resolve, reject) => {
    if(!Array.isArray(arr)) {
      throw new Error('arguments must be an array')
    }

    let results = [], count = 0

    arr.forEach((item, index) => {
      item.then(data => {
        results.push(data)
        count++

        if(count === arr.length) {
          return resolve(data)
        }

      })
      .catch(e => {
        return reject(e)
      })
    })

  })
}

if (!Promise.allSettled) {
  Promise.allSettled = function (promises) {
    return new Promise(resolve => {
      const data = [], len = promises.length;
      let count = len;
      for (let i = 0; i < len; i += 1) {
        const promise = promises[i];
        promise.then(res => {
          data[i] = { status: 'fulfilled', value: res };
        }, error => {
          data[i] = { status: 'rejected', reason: error };
        }).finally(() => { // promise has been settled
          if (!--count) {
            resolve(data);
          }
        });
      }
    });
  }
}

new MyPromise((resolve, reject) => {
  setTimeout(() => {
    resolve('成功的回调数据')
  }, 1000)
}).then(value => {
  console.log('Promise.then:  ', value)
})
```