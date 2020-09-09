```js
const PENGDING = 'pengding'
const RESOLVE = 'resolve'
const REJECT = 'reject'

function MyPromise(excutor) {
  const that = this
  that.state = PENGDING
  that.value = null
  that.resolvedCallbacks = []
  that.rejectedCallbacks = []

  function resolve(value){
    if(that.state === PENGDING) {
      that.state = RESOLVE
      that.value = value
      that.resolvedCallbacks.map(cb => cb(that.value))
    }
  }

  function reject(value) {
    if(that.state === PENGDING) {
      that.state = REJECT
      that.value = value
      that.rejectedCallbacks.map(cb => cb(that.value))
    }
  }

  try{
    excutor(resolve, reject)
  } catch(e){
    reject(e)
  }

}

MyPromise.prototype.then = function(onFulfilled, onRejected) {
  const that = this
  onFulfilled = typeof onFulfilled === 'function' ? onFulfilled : v => v
  onRejected = typeof onRejected === 'function' ? onRejected : r => {throw r}

  switch (that.state) {
    case PENGDING: 
      that.resolvedCallbacks.push(onFulfilled)
      that.rejectedCallbacks.push(onRejected)
      break
    case RESOLVE:
      onFulfilled(that.value)
      break
    case REJECT:
      onRejected(that.value)
      break
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

new MyPromise((resolve, reject) => {
  setTimeout(() => {
    resolve('成功的回调数据')
  }, 1000)
}).then(value => {
  console.log('Promise.then:  ', value)
})
```