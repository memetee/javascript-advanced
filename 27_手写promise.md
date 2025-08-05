### promise结构的设计

如果自己设计promise需要参考一个规范

promise是在ES6才出现的，也就是之前是没有promise的

但是在之前社区里面是有实现promise的

但是每个人实现的promise是不一样的

这样就造成很混乱

然后，社区就流行了一个规范，叫promisea+

https://promisesaplus.com/

他就写了，如果你想实现promise，应该有什么方法



```js
const PROMISE_STATUS_PENDING = 'pending'
const PROMISE_STATUS_FULFILLED = 'fulfilled'
const PROMISE_STATUS_REJECTED = 'rejected'
class HTPromise {
  // 它会接收一个参数，这个参数就是一个函数
  constructor(executor) {
    // 默认状态是pending状态
    this.status = PROMISE_STATUS_PENDING;

    // 因为传进来的这个函数是有两个函数，一个是resolve, 一个是reject
    // 所以我们需要在这里实现这两个函数，并且给到executor中
      const resolve = () => {
        // 只有状态是pending的时候才能调用这个东西
        if(this.status === PROMISE_STATUS_PENDING){
          this.status = PROMISE_STATUS_FULFILLED
          console.log('resolve被调用')
        }
      }
      const reject = () => {
        if(this.status === PROMISE_STATUS_PENDING) {
          this.status = PROMISE_STATUS_REJECTED
          console.log('reject被调用')
        }
      }

      // 我们需要调用这个函数
      executor(resolve, reject);
  }
}

// 我们在使用promise的时候是这个样子
const promise = new HTPromise((resolve, reject) => {
  console.log('这里被调用了')
  // 但是有一个问题，如果resolve被调用了，那么实际上reject不能再被调用的，也就是无效的调用
  // 其实在调用executor这个函数的时候，整个promise的状态应该是pendding的状态的，当调用resolve的状态后，它的状态应该是fulfilled，当调用了reject后，它的状态应该是rejected状态，
  // 所以必须记录这个状态
  resolve();
  reject();
})
```

状态简单管理实现了，也就是状态一旦变化，就不能改变了



保存参数

```js
const PROMISE_STATUS_PENDING = 'pending'
const PROMISE_STATUS_FULFILLED = 'fulfilled'
const PROMISE_STATUS_REJECTED = 'rejected'
class HTPromise {
  // 它会接收一个参数，这个参数就是一个函数
  constructor(executor) {
    // 默认状态是pending状态
    this.status = PROMISE_STATUS_PENDING;
    // 保存传进来的参数
    this.value = undefined;
    this.reason = undefined;

    // 因为传进来的这个函数是有两个函数，一个是resolve, 一个是reject
    // 所以我们需要在这里实现这两个函数，并且给到executor中
    // 接收参数，这里叫value
      const resolve = (value) => {
        // 只有状态是pending的时候才能调用这个东西
        if(this.status === PROMISE_STATUS_PENDING){
          this.status = PROMISE_STATUS_FULFILLED
          this.value = value;
          console.log('resolve被调用')
        }
      }
      // 这里的叫reason
      const reject = (reason) => {
        if(this.status === PROMISE_STATUS_PENDING) {
          this.status = PROMISE_STATUS_REJECTED
          this.reason = reason;
          console.log('reject被调用')
        }
      }

      // 我们需要调用这个函数
      executor(resolve, reject);
  }
}

// 我们在使用promise的时候是这个样子
const promise = new HTPromise((resolve, reject) => {
  console.log('这里被调用了')
  // 但是有一个问题，如果resolve被调用了，那么实际上reject不能再被调用的，也就是无效的调用
  // 其实在调用executor这个函数的时候，整个promise的状态应该是pendding的状态的，当调用resolve的状态后，它的状态应该是fulfilled，当调用了reject后，它的状态应该是rejected状态，
  // 所以必须记录这个状态
  // 传进去的参数必须要保存下来
  resolve(111);
  reject(222);
})

promise.then(res => {
  console.log(res)
}, err => {
  console.log(err);
})


```



拿到then方法中的两个函数，并调用

```js
const PROMISE_STATUS_PENDING = 'pending'
const PROMISE_STATUS_FULFILLED = 'fulfilled'
const PROMISE_STATUS_REJECTED = 'rejected'
class HTPromise {
  // 它会接收一个参数，这个参数就是一个函数
  constructor(executor) {
    // 默认状态是pending状态
    this.status = PROMISE_STATUS_PENDING;
    // 保存传进来的参数
    this.value = undefined;
    this.reason = undefined;

    // 因为传进来的这个函数是有两个函数，一个是resolve, 一个是reject
    // 所以我们需要在这里实现这两个函数，并且给到executor中
    // 接收参数，这里叫value
      const resolve = (value) => {
        // 只有状态是pending的时候才能调用这个东西
        if(this.status === PROMISE_STATUS_PENDING){
          this.status = PROMISE_STATUS_FULFILLED
          this.value = value;
          this.onFulfilled(this.value);
          console.log('resolve被调用')
          // 这里要执行then传进来的回调
        }
      }
      // 这里的叫reason
      const reject = (reason) => {
        if(this.status === PROMISE_STATUS_PENDING) {
          this.status = PROMISE_STATUS_REJECTED
          this.reason = reason;
          console.log('reject被调用');
          this.onRejected(this.reason);
          // 这里要执行then传进来的第二个回调
        }
      }

      // 我们需要调用这个函数
      executor(resolve, reject);
  }
  // 调用.then的时候执行这个函数，并且接收两个函数
  then(onFulfilled, onRejected) {
    // 保存以后去调用
    this.onFulfilled = onFulfilled;
    this.onRejected = onRejected;
  }
}

// 我们在使用promise的时候是这个样子
const promise = new HTPromise((resolve, reject) => {
  console.log('这里被调用了')
  // 但是有一个问题，如果resolve被调用了，那么实际上reject不能再被调用的，也就是无效的调用
  // 其实在调用executor这个函数的时候，整个promise的状态应该是pendding的状态的，当调用resolve的状态后，它的状态应该是fulfilled，当调用了reject后，它的状态应该是rejected状态，
  // 所以必须记录这个状态
  // 传进去的参数必须要保存下来
  resolve(111);
  reject(222);
})

promise.then(res => {
  console.log(res)
}, err => {
  console.log(err);
})


```



![image-20220707064518212](.\18_promise\image-20220707064518212.png)

会报错，因为先调用的是new 也就是会先执行constructor中的代码，但是，这个时候this.onFulfilled是没有值的

那么怎么执行这一点呢？

加个定时器吧

加个定时器，其实也是加一个宏任务，加了宏任务，实际上就是等到下一个事件循环的时候再执行，不会阻塞主线程的执行的

```js
const PROMISE_STATUS_PENDING = 'pending'
const PROMISE_STATUS_FULFILLED = 'fulfilled'
const PROMISE_STATUS_REJECTED = 'rejected'
class HTPromise {
  // 它会接收一个参数，这个参数就是一个函数
  constructor(executor) {
    // 默认状态是pending状态
    this.status = PROMISE_STATUS_PENDING;
    // 保存传进来的参数
    this.value = undefined;
    this.reason = undefined;

    // 因为传进来的这个函数是有两个函数，一个是resolve, 一个是reject
    // 所以我们需要在这里实现这两个函数，并且给到executor中
    // 接收参数，这里叫value
      const resolve = (value) => {
        // 只有状态是pending的时候才能调用这个东西
        if(this.status === PROMISE_STATUS_PENDING){
          this.status = PROMISE_STATUS_FULFILLED
          this.value = value;
          setTimeout(() => {
            this.onFulfilled(this.value);
          }, 0)
          console.log('resolve被调用')
          // 这里要执行then传进来的回调
        }
      }
      // 这里的叫reason
      const reject = (reason) => {
        if(this.status === PROMISE_STATUS_PENDING) {
          this.status = PROMISE_STATUS_REJECTED
          this.reason = reason;
          console.log('reject被调用');
          setTimeout(() => {
            this.onRejected(this.reason);
          }, 0)
          // 这里要执行then传进来的第二个回调
        }
      }

      // 我们需要调用这个函数
      executor(resolve, reject);
  }
  // 调用.then的时候执行这个函数，并且接收两个函数
  then(onFulfilled, onRejected) {
    // 保存以后去调用
    this.onFulfilled = onFulfilled;
    this.onRejected = onRejected;
  }
}

// 我们在使用promise的时候是这个样子
const promise = new HTPromise((resolve, reject) => {
  console.log('这里被调用了')
  // 但是有一个问题，如果resolve被调用了，那么实际上reject不能再被调用的，也就是无效的调用
  // 其实在调用executor这个函数的时候，整个promise的状态应该是pendding的状态的，当调用resolve的状态后，它的状态应该是fulfilled，当调用了reject后，它的状态应该是rejected状态，
  // 所以必须记录这个状态
  // 传进去的参数必须要保存下来
  resolve(111);
  reject(222);
})

promise.then(res => {
  console.log(res)
}, err => {
  console.log(err);
})


```



但是在这里最好不要用setTimeout， 因为在原生的实现上，它用的就是微任务，而我们这里用的是宏任务，所以我们再改造一下

```js
const PROMISE_STATUS_PENDING = 'pending'
const PROMISE_STATUS_FULFILLED = 'fulfilled'
const PROMISE_STATUS_REJECTED = 'rejected'
class HTPromise {
  // 它会接收一个参数，这个参数就是一个函数
  constructor(executor) {
    // 默认状态是pending状态
    this.status = PROMISE_STATUS_PENDING;
    // 保存传进来的参数
    this.value = undefined;
    this.reason = undefined;

    // 因为传进来的这个函数是有两个函数，一个是resolve, 一个是reject
    // 所以我们需要在这里实现这两个函数，并且给到executor中
    // 接收参数，这里叫value
      const resolve = (value) => {
        // 只有状态是pending的时候才能调用这个东西
        if(this.status === PROMISE_STATUS_PENDING){
          this.status = PROMISE_STATUS_FULFILLED
          this.value = value;
          queueMicrotask(() => {
            this.onFulfilled(this.value);
          })
          console.log('resolve被调用')
          // 这里要执行then传进来的回调
        }
      }
      // 这里的叫reason
      const reject = (reason) => {
        if(this.status === PROMISE_STATUS_PENDING) {
          this.status = PROMISE_STATUS_REJECTED
          this.reason = reason;
          console.log('reject被调用');
          queueMicrotask(() => {
            this.onRejected(this.reason);
          }, 0)
          // 这里要执行then传进来的第二个回调
        }
      }

      // 我们需要调用这个函数
      executor(resolve, reject);
  }
  // 调用.then的时候执行这个函数，并且接收两个函数
  then(onFulfilled, onRejected) {
    // 保存以后去调用
    this.onFulfilled = onFulfilled;
    this.onRejected = onRejected;
  }
}

// 我们在使用promise的时候是这个样子
const promise = new HTPromise((resolve, reject) => {
  console.log('这里被调用了')
  // 但是有一个问题，如果resolve被调用了，那么实际上reject不能再被调用的，也就是无效的调用
  // 其实在调用executor这个函数的时候，整个promise的状态应该是pendding的状态的，当调用resolve的状态后，它的状态应该是fulfilled，当调用了reject后，它的状态应该是rejected状态，
  // 所以必须记录这个状态
  // 传进去的参数必须要保存下来
  resolve(111);
  reject(222);
})

promise.then(res => {
  console.log(res)
}, err => {
  console.log(err);
})


```

queueMicrotask是加在微任务当中，会在本轮的事件循环中进行执行

如果接着在下面调用then,应该是两个then都会执行的，但是我们调用的话，只会执行第二个

![image-20220707065950254](.\18_promise\image-20220707065950254.png)

因为第二个会把第一个覆盖掉

而且我们这里不能做链式调用

![image-20220707070138792](.\18_promise\image-20220707070138792.png)



当前的then方法不支持调用多次，后面把之前的方法调用，then里面的两个方法

![image-20220707070617695](.\18_promise\image-20220707070617695.png)

then方法不支持链式调用



那么怎么做呢？

塞入到数组里面去

```js
const PROMISE_STATUS_PENDING = 'pending'
const PROMISE_STATUS_FULFILLED = 'fulfilled'
const PROMISE_STATUS_REJECTED = 'rejected'
class HTPromise {
  constructor(executor) {
    this.status = PROMISE_STATUS_PENDING;
    this.value = undefined;
    this.reason = undefined;

    // 保存成功的回调函数
    this.onFulfilledFns = [];
    this.onRejectedFns = []
      const resolve = (value) => {
        if(this.status === PROMISE_STATUS_PENDING){
          this.status = PROMISE_STATUS_FULFILLED
          queueMicrotask(() => {
            this.value = value;
            this.onRejectedFns.forEach(fn => {
              fn(this.value);
            })
          })
          console.log('resolve被调用')
        }
      }
      const reject = (reason) => {
        if(this.status === PROMISE_STATUS_PENDING) {
          this.status = PROMISE_STATUS_REJECTED
          queueMicrotask(() => {
            this.reason = reason;
            console.log('reject被调用');
            this.onRejected(this.reason);
            this.onRejectedFns.forEach(fn => {
              fn(this.reason);
            })
          })
        }
      }
      executor(resolve, reject);
  }
  then(onFulfilled, onRejected) {

    // 将成功回调和失败回调加入到数组中
    this.onFulfilledFns.push(onFulfilled);
    this.onRejectedFns.push(onRejected);
  }
}

// 我们在使用promise的时候是这个样子
const promise = new HTPromise((resolve, reject) => {
  console.log('这里被调用了')
  // 但是有一个问题，如果resolve被调用了，那么实际上reject不能再被调用的，也就是无效的调用
  // 其实在调用executor这个函数的时候，整个promise的状态应该是pendding的状态的，当调用resolve的状态后，它的状态应该是fulfilled，当调用了reject后，它的状态应该是rejected状态，
  // 所以必须记录这个状态
  // 传进去的参数必须要保存下来
  resolve(111);
  reject(222);
})

promise.then(res => {
  console.log(res)
  console.log('then1')
}, err => {
  console.log(err);
})

promise.then(res => {
  console.log('then2')
  console.log(res)
}, err => {
  console.log(err);
})



```

![image-20220707071229835](.\18_promise\image-20220707071229835.png)



但是如果promise变成了resolve

![image-20220707071435510](.\18_promise\image-20220707071435510.png)

不会，因为这里已经回调完了

![image-20220707071608119](.\18_promise\image-20220707071608119.png)

我在执行onFulfilledFns的时候，你延迟了一秒钟，但是我已经执行完了，等我执行完了你才加进来，这个时候，我肯定不会再来执行一遍你的



但是原生的promise是可以的

![image-20220707071748205](.\18_promise\image-20220707071748205.png)

这个代码就会直接回调的



那要怎么办呢？

![image-20220707072055130](.\18_promise\image-20220707072055130.png)

优化多次调用，延迟一会再调用

```js
const PROMISE_STATUS_PENDING = 'pending'
const PROMISE_STATUS_FULFILLED = 'fulfilled'
const PROMISE_STATUS_REJECTED = 'rejected'
class HTPromise {
  constructor(executor) {
    this.status = PROMISE_STATUS_PENDING;
    this.value = undefined;
    this.reason = undefined;

    // 保存成功的回调函数
    this.onFulfilledFns = [];
    this.onRejectedFns = []
      const resolve = (value) => {
        if(this.status === PROMISE_STATUS_PENDING){
          queueMicrotask(() => {
            this.status = PROMISE_STATUS_FULFILLED
            this.value = value;
            this.onFulfilledFns.forEach(fn => {
              fn(this.value);
            })
          })
          console.log('resolve被调用')
        }
      }
      const reject = (reason) => {
        if(this.status === PROMISE_STATUS_PENDING) {
          queueMicrotask(() => {
            this.status = PROMISE_STATUS_REJECTED
            this.reason = reason;
            this.onRejectedFns.forEach(fn => {
              fn(this.reason);
            })
          })
        }
      }
      executor(resolve, reject);
  }
  then(onFulfilled, onRejected) {
    if(this.status === PROMISE_STATUS_FULFILLED && onFulfilled) {
      onFulfilled(this.value);
    }
    if(this.status === PROMISE_STATUS_REJECTED && onRejected) {
      onRejected(this.reason);
    }

    if(this.status === PROMISE_STATUS_PENDING) {
        this.onFulfilledFns.push(onFulfilled);
        this.onRejectedFns.push(onRejected);
      }
    }
}

// 我们在使用promise的时候是这个样子
const promise = new HTPromise((resolve, reject) => {
  console.log('这里被调用了')
  // 但是有一个问题，如果resolve被调用了，那么实际上reject不能再被调用的，也就是无效的调用
  // 其实在调用executor这个函数的时候，整个promise的状态应该是pendding的状态的，当调用resolve的状态后，它的状态应该是fulfilled，当调用了reject后，它的状态应该是rejected状态，
  // 所以必须记录这个状态
  // 传进去的参数必须要保存下来
  resolve(111);
  // reject(222);
})

promise.then(res => {
  console.log('then1')
  console.log(res)
}, err => {
  console.log(err);
})

promise.then(res => {
  console.log('then2')
  console.log(res)
}, err => {
  console.log(err);
})

setTimeout(() => {
  promise.then(res => {
    console.log('promise3')
    console.log(res);
  })
}, 1000)


```



![image-20220707072649015](.\18_promise\image-20220707072649015.png)

如果这里都执行了，那么他们都会执行

![image-20220707072915256](.\18_promise\image-20220707072915256.png)

这样解决这个问题

```js
const PROMISE_STATUS_PENDING = 'pending'
const PROMISE_STATUS_FULFILLED = 'fulfilled'
const PROMISE_STATUS_REJECTED = 'rejected'
class HTPromise {
  constructor(executor) {
    this.status = PROMISE_STATUS_PENDING;
    this.value = undefined;
    this.reason = undefined;

    // 保存成功的回调函数
    this.onFulfilledFns = [];
    this.onRejectedFns = []
      const resolve = (value) => {
        if(this.status === PROMISE_STATUS_PENDING){
          queueMicrotask(() => {
            if(this.status !== PROMISE_STATUS_PENDING) return;
            this.status = PROMISE_STATUS_FULFILLED
            this.value = value;
            this.onFulfilledFns.forEach(fn => {
              fn(this.value);
            })
          })
          console.log('resolve被调用')
        }
      }
      const reject = (reason) => {
        if(this.status === PROMISE_STATUS_PENDING) {
          queueMicrotask(() => {
            if(this.status !== PROMISE_STATUS_PENDING) return;
            this.status = PROMISE_STATUS_REJECTED
            this.reason = reason;
            this.onRejectedFns.forEach(fn => {
              fn(this.reason);
            })
          })
        }
      }
      executor(resolve, reject);
  }
  then(onFulfilled, onRejected) {
    if(this.status === PROMISE_STATUS_FULFILLED && onFulfilled) {
      onFulfilled(this.value);
    }
    if(this.status === PROMISE_STATUS_REJECTED && onRejected) {
      onRejected(this.reason);
    }

    if(this.status === PROMISE_STATUS_PENDING) {
        this.onFulfilledFns.push(onFulfilled);
        this.onRejectedFns.push(onRejected);
      }
    }
}

// 我们在使用promise的时候是这个样子
const promise = new HTPromise((resolve, reject) => {
  console.log('这里被调用了')
  // 但是有一个问题，如果resolve被调用了，那么实际上reject不能再被调用的，也就是无效的调用
  // 其实在调用executor这个函数的时候，整个promise的状态应该是pendding的状态的，当调用resolve的状态后，它的状态应该是fulfilled，当调用了reject后，它的状态应该是rejected状态，
  // 所以必须记录这个状态
  // 传进去的参数必须要保存下来
  reject(222);
  resolve(111);
})

promise.then(res => {
  console.log('then1')
  console.log(res)
}, err => {
  console.log(err);
})

promise.then(res => {
  console.log('then2')
  console.log(res)
}, err => {
  console.log(err);
})

setTimeout(() => {
  promise.then(res => {
    console.log('promise3')
    console.log(res);
  })
}, 1000)



```



现在我们要做链式调用

![image-20220707073523436](.\18_promise\image-20220707073523436.png)

这样才能链式调用

但是我们是没有返回值的，所以默认返回的是undefined

![image-20220707073606856](.\18_promise\image-20220707073606856.png)

![image-20220707073749138](.\18_promise\image-20220707073749138.png)

但是这里不应该写死的， 而且，应该是第一次有结果的时候，才能拿到这个结果，然后在第二次才能调用resolve



![image-20220707074225567](.\18_promise\image-20220707074225567.png)



```js
// ES6 ES2015
// https://promisesaplus.com/
const PROMISE_STATUS_PENDING = 'pending'
const PROMISE_STATUS_FULFILLED = 'fulfilled'
const PROMISE_STATUS_REJECTED = 'rejected'

// 工具函数
function execFunctionWithCatchError(execFn, value, resolve, reject) {
  try {
    const result = execFn(value)
    resolve(result)
  } catch(err) {
    reject(err)
  }
}

class HYPromise {
  constructor(executor) {
    this.status = PROMISE_STATUS_PENDING
    this.value = undefined
    this.reason = undefined
    this.onFulfilledFns = []
    this.onRejectedFns = []

    const resolve = (value) => {
      if (this.status === PROMISE_STATUS_PENDING) {
        // 添加微任务
        queueMicrotask(() => {
          if (this.status !== PROMISE_STATUS_PENDING) return
          this.status = PROMISE_STATUS_FULFILLED
          this.value = value
          this.onFulfilledFns.forEach(fn => {
            fn(this.value)
          })
        });
      }
    }

    const reject = (reason) => {
      if (this.status === PROMISE_STATUS_PENDING) {
        // 添加微任务
        queueMicrotask(() => {
          if (this.status !== PROMISE_STATUS_PENDING) return
          this.status = PROMISE_STATUS_REJECTED
          this.reason = reason
          this.onRejectedFns.forEach(fn => {
            fn(this.reason)
          })
        })
      }
    }

    try {
      executor(resolve, reject)
    } catch (err) {
      reject(err)
    }
  }

  then(onFulfilled, onRejected) {
    return new HYPromise((resolve, reject) => {
      // 1.如果在then调用的时候, 状态已经确定下来
      if (this.status === PROMISE_STATUS_FULFILLED && onFulfilled) {
        // try {
        //   const value = onFulfilled(this.value)
        //   resolve(value)
        // } catch(err) {
        //   reject(err)
        // }
        execFunctionWithCatchError(onFulfilled, this.value, resolve, reject)
      }
      if (this.status === PROMISE_STATUS_REJECTED && onRejected) {
        // try {
        //   const reason = onRejected(this.reason)
        //   resolve(reason)
        // } catch(err) {
        //   reject(err)
        // }
        execFunctionWithCatchError(onRejected, this.reason, resolve, reject)
      }

      // 2.将成功回调和失败的回调放到数组中
      if (this.status === PROMISE_STATUS_PENDING) {
        this.onFulfilledFns.push(() => {
          // try {
          //   const value = onFulfilled(this.value)
          //   resolve(value)
          // } catch(err) {
          //   reject(err)
          // }
          execFunctionWithCatchError(onFulfilled, this.value, resolve, reject)
        })
        this.onRejectedFns.push(() => {
          // try {
          //   const reason = onRejected(this.reason)
          //   resolve(reason)
          // } catch(err) {
          //   reject(err)
          // }
          execFunctionWithCatchError(onRejected, this.reason, resolve, reject)
        })
      }
    })
  }
}

const promise = new HYPromise((resolve, reject) => {
  console.log("状态pending")
  // resolve(1111) // resolved/fulfilled
  reject(2222)
  // throw new Error("executor error message")
})

// 调用then方法多次调用
promise.then(res => {
  console.log("res1:", res)
  return "aaaa"
  // throw new Error("err message")
}, err => {
  console.log("err1:", err)
  return "bbbbb"
  // throw new Error("err message")
}).then(res => {
  console.log("res2:", res)
}, err => {
  console.log("err2:", err)
})

```



Promise A+要实现的功能，现在已经实现完了

实现catch方法

我们之前的catch是放到then的第二个参数，但是如果我们想这样写，就必须实现一个catch函数

![image-20220709081109342](.\18_promise\image-20220709081109342.png)

我们可以这样写

![image-20220709081347387](.\18_promise\image-20220709081347387.png)

有一个问题

![image-20220709081330166](.\18_promise\image-20220709081330166.png)

因为这里是空的

![image-20220709081425994](.\18_promise\image-20220709081425994.png)

因为undefined会被加入到里面去，执行undefined就报错了

解决

![image-20220709081744868](.\18_promise\image-20220709081744868.png)



继续执行

![image-20220709081727117](.\18_promise\image-20220709081727117.png)

![image-20220709082028799](.\18_promise\image-20220709082028799.png)

所以调不到的

要这样做

![image-20220709082543272](.\18_promise\image-20220709082543272.png)

```js
// ES6 ES2015
// https://promisesaplus.com/
const PROMISE_STATUS_PENDING = 'pending'
const PROMISE_STATUS_FULFILLED = 'fulfilled'
const PROMISE_STATUS_REJECTED = 'rejected'

// 工具函数
function execFunctionWithCatchError(execFn, value, resolve, reject) {
  try {
    const result = execFn(value)
    resolve(result)
  } catch(err) {
    reject(err)
  }
}

class HYPromise {
  constructor(executor) {
    this.status = PROMISE_STATUS_PENDING
    this.value = undefined
    this.reason = undefined
    this.onFulfilledFns = []
    this.onRejectedFns = []

    const resolve = (value) => {
      if (this.status === PROMISE_STATUS_PENDING) {
        // 添加微任务
        queueMicrotask(() => {
          if (this.status !== PROMISE_STATUS_PENDING) return
          this.status = PROMISE_STATUS_FULFILLED
          this.value = value
          this.onFulfilledFns.forEach(fn => {
            fn(this.value)
          })
        });
      }
    }

    const reject = (reason) => {
      if (this.status === PROMISE_STATUS_PENDING) {
        // 添加微任务
        queueMicrotask(() => {
          if (this.status !== PROMISE_STATUS_PENDING) return
          this.status = PROMISE_STATUS_REJECTED
          this.reason = reason
          this.onRejectedFns.forEach(fn => {
            fn(this.reason)
          })
        })
      }
    }

    try {
      executor(resolve, reject)
    } catch (err) {
      reject(err)
    }
  }

  then(onFulfilled, onRejected) {
    onRejected = onRejected === undefined ? err => {throw err} : onRejected;
    return new HYPromise((resolve, reject) => {
      // 1.如果在then调用的时候, 状态已经确定下来
      if (this.status === PROMISE_STATUS_FULFILLED && onFulfilled) {
        // try {
        //   const value = onFulfilled(this.value)
        //   resolve(value)
        // } catch(err) {
        //   reject(err)
        // }
        execFunctionWithCatchError(onFulfilled, this.value, resolve, reject)
      }
      if (this.status === PROMISE_STATUS_REJECTED && onRejected) {
        // try {
        //   const reason = onRejected(this.reason)
        //   resolve(reason)
        // } catch(err) {
        //   reject(err)
        // }
        execFunctionWithCatchError(onRejected, this.reason, resolve, reject)
      }

      // 2.将成功回调和失败的回调放到数组中
      if (this.status === PROMISE_STATUS_PENDING) {
        onFulfilled && this.onFulfilledFns.push(() => {
          // try {
          //   const value = onFulfilled(this.value)
          //   resolve(value)
          // } catch(err) {
          //   reject(err)
          // }
          execFunctionWithCatchError(onFulfilled, this.value, resolve, reject)
        })
        onRejected && this.onRejectedFns.push(() => {
          // try {
          //   const reason = onRejected(this.reason)
          //   resolve(reason)
          // } catch(err) {
          //   reject(err)
          // }
          execFunctionWithCatchError(onRejected, this.reason, resolve, reject)
        })
      }
    })
  }
  catch(onRejected) {
    // 第一个参数不需要传
    this.then(undefined, onRejected)
  }
}

const promise = new HYPromise((resolve, reject) => {
  console.log("状态pending")
  // resolve(1111) // resolved/fulfilled
  reject(2222)
  // throw new Error("executor error message")
})

promise.then(res => {
  console.log('res', res);
}, err => {
  console.log(err)
}).catch(err => {
  console.log(err)
})
```



这样做，就可以让catch实现链式了



实现finally方法

当前不能使用finally方法，因为catch没有return任何东西

![image-20220709083452916](.\18_promise\image-20220709083452916.png)



![image-20220709083531149](.\18_promise\image-20220709083531149.png)

因为这里

![image-20220709083604507](.\18_promise\image-20220709083604507.png)



因为undefined，断层了

![image-20220709083803538](.\18_promise\image-20220709083803538.png)

```js
// ES6 ES2015
// https://promisesaplus.com/
const PROMISE_STATUS_PENDING = 'pending'
const PROMISE_STATUS_FULFILLED = 'fulfilled'
const PROMISE_STATUS_REJECTED = 'rejected'

// 工具函数
function execFunctionWithCatchError(execFn, value, resolve, reject) {
  try {
    const result = execFn(value)
    resolve(result)
  } catch(err) {
    reject(err)
  }
}

class HYPromise {
  constructor(executor) {
    this.status = PROMISE_STATUS_PENDING
    this.value = undefined
    this.reason = undefined
    this.onFulfilledFns = []
    this.onRejectedFns = []

    const resolve = (value) => {
      if (this.status === PROMISE_STATUS_PENDING) {
        // 添加微任务
        queueMicrotask(() => {
          if (this.status !== PROMISE_STATUS_PENDING) return
          this.status = PROMISE_STATUS_FULFILLED
          this.value = value
          this.onFulfilledFns.forEach(fn => {
            fn(this.value)
          })
        });
      }
    }

    const reject = (reason) => {
      if (this.status === PROMISE_STATUS_PENDING) {
        // 添加微任务
        queueMicrotask(() => {
          if (this.status !== PROMISE_STATUS_PENDING) return
          this.status = PROMISE_STATUS_REJECTED
          this.reason = reason
          this.onRejectedFns.forEach(fn => {
            fn(this.reason)
          })
        })
      }
    }

    try {
      executor(resolve, reject)
    } catch (err) {
      reject(err)
    }
  }

  then(onFulfilled, onRejected) {
    onRejected = onRejected === undefined ? err => {throw err} : onRejected;
      // 这里要加个判断
    onFulfilled = onFulfilled === undefined ? value => {return value} : onFulfilled;
    return new HYPromise((resolve, reject) => {
      // 1.如果在then调用的时候, 状态已经确定下来
      if (this.status === PROMISE_STATUS_FULFILLED && onFulfilled) {
        // try {
        //   const value = onFulfilled(this.value)
        //   resolve(value)
        // } catch(err) {
        //   reject(err)
        // }
        execFunctionWithCatchError(onFulfilled, this.value, resolve, reject)
      }
      if (this.status === PROMISE_STATUS_REJECTED && onRejected) {
        // try {
        //   const reason = onRejected(this.reason)
        //   resolve(reason)
        // } catch(err) {
        //   reject(err)
        // }
        execFunctionWithCatchError(onRejected, this.reason, resolve, reject)
      }

      // 2.将成功回调和失败的回调放到数组中
      if (this.status === PROMISE_STATUS_PENDING) {
        onFulfilled && this.onFulfilledFns.push(() => {
          // try {
          //   const value = onFulfilled(this.value)
          //   resolve(value)
          // } catch(err) {
          //   reject(err)
          // }
          execFunctionWithCatchError(onFulfilled, this.value, resolve, reject)
        })
        onRejected && this.onRejectedFns.push(() => {
          // try {
          //   const reason = onRejected(this.reason)
          //   resolve(reason)
          // } catch(err) {
          //   reject(err)
          // }
          execFunctionWithCatchError(onRejected, this.reason, resolve, reject)
        })
      }
    })
  }
  catch(onRejected) {
    // 第一个参数不需要传
    return this.then(undefined, onRejected)
  }
  finally(onFinally) {
    this.then(() => {
      onFinally()
    }, () => {
      onFinally();
    });
  }
}

const promise = new HYPromise((resolve, reject) => {
  console.log("状态pending")
  // resolve(1111) // resolved/fulfilled
  reject(2222)
  // throw new Error("executor error message")
})

promise.then(res => {
  console.log('res', res);
}, err => {
  console.log(err)
}).catch(res => {
  console.log('bbbb')
  return 'bbbbb'
}).finally(() => {
  console.log('finally')
})
```

对象方法已经实现完了

然后实现类方法，resolve和reject

```js
// ES6 ES2015
// https://promisesaplus.com/
const PROMISE_STATUS_PENDING = 'pending'
const PROMISE_STATUS_FULFILLED = 'fulfilled'
const PROMISE_STATUS_REJECTED = 'rejected'

// 工具函数
function execFunctionWithCatchError(execFn, value, resolve, reject) {
  try {
    const result = execFn(value)
    resolve(result)
  } catch(err) {
    reject(err)
  }
}

class HYPromise {
  constructor(executor) {
    this.status = PROMISE_STATUS_PENDING
    this.value = undefined
    this.reason = undefined
    this.onFulfilledFns = []
    this.onRejectedFns = []

    const resolve = (value) => {
      if (this.status === PROMISE_STATUS_PENDING) {
        // 添加微任务
        queueMicrotask(() => {
          if (this.status !== PROMISE_STATUS_PENDING) return
          this.status = PROMISE_STATUS_FULFILLED
          this.value = value
          this.onFulfilledFns.forEach(fn => {
            fn(this.value)
          })
        });
      }
    }

    const reject = (reason) => {
      if (this.status === PROMISE_STATUS_PENDING) {
        // 添加微任务
        queueMicrotask(() => {
          if (this.status !== PROMISE_STATUS_PENDING) return
          this.status = PROMISE_STATUS_REJECTED
          this.reason = reason
          this.onRejectedFns.forEach(fn => {
            fn(this.reason)
          })
        })
      }
    }

    try {
      executor(resolve, reject)
    } catch (err) {
      reject(err)
    }
  }

  then(onFulfilled, onRejected) {
    onRejected = onRejected === undefined ? err => {throw err} : onRejected;
    onFulfilled = onFulfilled === undefined ? value => {return value} : onFulfilled;
    return new HYPromise((resolve, reject) => {
      // 1.如果在then调用的时候, 状态已经确定下来
      if (this.status === PROMISE_STATUS_FULFILLED && onFulfilled) {
        // try {
        //   const value = onFulfilled(this.value)
        //   resolve(value)
        // } catch(err) {
        //   reject(err)
        // }
        execFunctionWithCatchError(onFulfilled, this.value, resolve, reject)
      }
      if (this.status === PROMISE_STATUS_REJECTED && onRejected) {
        // try {
        //   const reason = onRejected(this.reason)
        //   resolve(reason)
        // } catch(err) {
        //   reject(err)
        // }
        execFunctionWithCatchError(onRejected, this.reason, resolve, reject)
      }

      // 2.将成功回调和失败的回调放到数组中
      if (this.status === PROMISE_STATUS_PENDING) {
        onFulfilled && this.onFulfilledFns.push(() => {
          // try {
          //   const value = onFulfilled(this.value)
          //   resolve(value)
          // } catch(err) {
          //   reject(err)
          // }
          execFunctionWithCatchError(onFulfilled, this.value, resolve, reject)
        })
        onRejected && this.onRejectedFns.push(() => {
          // try {
          //   const reason = onRejected(this.reason)
          //   resolve(reason)
          // } catch(err) {
          //   reject(err)
          // }
          execFunctionWithCatchError(onRejected, this.reason, resolve, reject)
        })
      }
    })
  }
  catch(onRejected) {
    // 第一个参数不需要传
    return this.then(undefined, onRejected)
  }
  finally(onFinally) {
    this.then(() => {
      onFinally()
    }, () => {
      onFinally();
    });
  }
  static resolve(value) {
    return new HYPromise(resolve => resolve(value))
  }
  static reject(reason) {
    return new HYPromise((resolve, reject) => reject(reason))
  }
}

const promise = new HYPromise((resolve, reject) => {
  console.log("状态pending")
  // resolve(1111) // resolved/fulfilled
  reject(2222)
  // throw new Error("executor error message")
})
HYPromise.resolve('hello world').then(res => {
  console.log(res)
})
HYPromise.reject('hello world').catch(err => {
  console.log(err)
})
```



all, allSettled方法

```js
// ES6 ES2015
// https://promisesaplus.com/
const PROMISE_STATUS_PENDING = 'pending'
const PROMISE_STATUS_FULFILLED = 'fulfilled'
const PROMISE_STATUS_REJECTED = 'rejected'

// 工具函数
function execFunctionWithCatchError(execFn, value, resolve, reject) {
  try {
    const result = execFn(value)
    resolve(result)
  } catch(err) {
    reject(err)
  }
}

class HYPromise {
  constructor(executor) {
    this.status = PROMISE_STATUS_PENDING
    this.value = undefined
    this.reason = undefined
    this.onFulfilledFns = []
    this.onRejectedFns = []

    const resolve = (value) => {
      if (this.status === PROMISE_STATUS_PENDING) {
        // 添加微任务
        queueMicrotask(() => {
          if (this.status !== PROMISE_STATUS_PENDING) return
          this.status = PROMISE_STATUS_FULFILLED
          this.value = value
          this.onFulfilledFns.forEach(fn => {
            fn(this.value)
          })
        });
      }
    }

    const reject = (reason) => {
      if (this.status === PROMISE_STATUS_PENDING) {
        // 添加微任务
        queueMicrotask(() => {
          if (this.status !== PROMISE_STATUS_PENDING) return
          this.status = PROMISE_STATUS_REJECTED
          this.reason = reason
          this.onRejectedFns.forEach(fn => {
            fn(this.reason)
          })
        })
      }
    }

    try {
      executor(resolve, reject)
    } catch (err) {
      reject(err)
    }
  }

  then(onFulfilled, onRejected) {
    onRejected = onRejected === undefined ? err => {throw err} : onRejected;
    onFulfilled = onFulfilled === undefined ? value => {return value} : onFulfilled;
    return new HYPromise((resolve, reject) => {
      // 1.如果在then调用的时候, 状态已经确定下来
      if (this.status === PROMISE_STATUS_FULFILLED && onFulfilled) {
        // try {
        //   const value = onFulfilled(this.value)
        //   resolve(value)
        // } catch(err) {
        //   reject(err)
        // }
        execFunctionWithCatchError(onFulfilled, this.value, resolve, reject)
      }
      if (this.status === PROMISE_STATUS_REJECTED && onRejected) {
        // try {
        //   const reason = onRejected(this.reason)
        //   resolve(reason)
        // } catch(err) {
        //   reject(err)
        // }
        execFunctionWithCatchError(onRejected, this.reason, resolve, reject)
      }

      // 2.将成功回调和失败的回调放到数组中
      if (this.status === PROMISE_STATUS_PENDING) {
        onFulfilled && this.onFulfilledFns.push(() => {
          // try {
          //   const value = onFulfilled(this.value)
          //   resolve(value)
          // } catch(err) {
          //   reject(err)
          // }
          execFunctionWithCatchError(onFulfilled, this.value, resolve, reject)
        })
        onRejected && this.onRejectedFns.push(() => {
          // try {
          //   const reason = onRejected(this.reason)
          //   resolve(reason)
          // } catch(err) {
          //   reject(err)
          // }
          execFunctionWithCatchError(onRejected, this.reason, resolve, reject)
        })
      }
    })
  }
  catch(onRejected) {
    // 第一个参数不需要传
    return this.then(undefined, onRejected)
  }
  finally(onFinally) {
    this.then(() => {
      onFinally()
    }, () => {
      onFinally();
    });
  }
  static resolve(value) {
    return new HYPromise(resolve => resolve(value))
  }
  static reject(reason) {
    return new HYPromise((resolve, reject) => reject(reason))
  }
  static all(promises) {

    return new HYPromise((resolve, reject) => {
      const values = [];
      promises.forEach(promise => {
        promise.then(res => {
          values.push(res);
          if(values.length === promises.length) {
            resolve(values);
          }
        }, err => {
          reject(err);
        })
      })
    })
  }
}
const p1 = new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve(111)
  }, 1000)
})
const p2 = new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve(222)
  }, 2000)
})
const p3 = new Promise((resolve, reject) => {
  setTimeout(() => {
    reject(333)
  }, 3000)
})

// 等所有的都有结果了之后，才执行then
// 如果有一个是err，那么就执行catch
HYPromise.all([p1, p2, p3]).then(res => {
  console.log(res);
}).catch(err => {
  console.log(err);
})

```

实现了all

```js
// ES6 ES2015
// https://promisesaplus.com/
const PROMISE_STATUS_PENDING = 'pending'
const PROMISE_STATUS_FULFILLED = 'fulfilled'
const PROMISE_STATUS_REJECTED = 'rejected'

// 工具函数
function execFunctionWithCatchError(execFn, value, resolve, reject) {
  try {
    const result = execFn(value)
    resolve(result)
  } catch(err) {
    reject(err)
  }
}

class HYPromise {
  constructor(executor) {
    this.status = PROMISE_STATUS_PENDING
    this.value = undefined
    this.reason = undefined
    this.onFulfilledFns = []
    this.onRejectedFns = []

    const resolve = (value) => {
      if (this.status === PROMISE_STATUS_PENDING) {
        // 添加微任务
        queueMicrotask(() => {
          if (this.status !== PROMISE_STATUS_PENDING) return
          this.status = PROMISE_STATUS_FULFILLED
          this.value = value
          this.onFulfilledFns.forEach(fn => {
            fn(this.value)
          })
        });
      }
    }

    const reject = (reason) => {
      if (this.status === PROMISE_STATUS_PENDING) {
        // 添加微任务
        queueMicrotask(() => {
          if (this.status !== PROMISE_STATUS_PENDING) return
          this.status = PROMISE_STATUS_REJECTED
          this.reason = reason
          this.onRejectedFns.forEach(fn => {
            fn(this.reason)
          })
        })
      }
    }

    try {
      executor(resolve, reject)
    } catch (err) {
      reject(err)
    }
  }

  then(onFulfilled, onRejected) {
    onRejected = onRejected === undefined ? err => {throw err} : onRejected;
    onFulfilled = onFulfilled === undefined ? value => {return value} : onFulfilled;
    return new HYPromise((resolve, reject) => {
      // 1.如果在then调用的时候, 状态已经确定下来
      if (this.status === PROMISE_STATUS_FULFILLED && onFulfilled) {
        // try {
        //   const value = onFulfilled(this.value)
        //   resolve(value)
        // } catch(err) {
        //   reject(err)
        // }
        execFunctionWithCatchError(onFulfilled, this.value, resolve, reject)
      }
      if (this.status === PROMISE_STATUS_REJECTED && onRejected) {
        // try {
        //   const reason = onRejected(this.reason)
        //   resolve(reason)
        // } catch(err) {
        //   reject(err)
        // }
        execFunctionWithCatchError(onRejected, this.reason, resolve, reject)
      }

      // 2.将成功回调和失败的回调放到数组中
      if (this.status === PROMISE_STATUS_PENDING) {
        onFulfilled && this.onFulfilledFns.push(() => {
          // try {
          //   const value = onFulfilled(this.value)
          //   resolve(value)
          // } catch(err) {
          //   reject(err)
          // }
          execFunctionWithCatchError(onFulfilled, this.value, resolve, reject)
        })
        onRejected && this.onRejectedFns.push(() => {
          // try {
          //   const reason = onRejected(this.reason)
          //   resolve(reason)
          // } catch(err) {
          //   reject(err)
          // }
          execFunctionWithCatchError(onRejected, this.reason, resolve, reject)
        })
      }
    })
  }
  catch(onRejected) {
    // 第一个参数不需要传
    return this.then(undefined, onRejected)
  }
  finally(onFinally) {
    this.then(() => {
      onFinally()
    }, () => {
      onFinally();
    });
  }
  static resolve(value) {
    return new HYPromise(resolve => resolve(value))
  }
  static reject(reason) {
    return new HYPromise((resolve, reject) => reject(reason))
  }
  static all(promises) {

    return new HYPromise((resolve, reject) => {
      const values = [];
      promises.forEach(promise => {
        promise.then(res => {
          values.push(res);
          if(values.length === promises.length) {
            resolve(values);
          }
        }, err => {
          reject(err);
        })
      })
    })
  }
  static allSettled(promises) {
    return new HYPromise((resolve, reject) => {
      const results = [];
      promises.forEach(promise => {
        promise.then(res => {
          results.push({status: PROMISE_STATUS_FULFILLED, value: res})
          if(results.length === promises.length) {
            resolve(results)
          }
        }, err => {
          results.push({status: PROMISE_STATUS_REJECTED, value: err})
          if(results.length === promises.length) {
            resolve(results)
          }
        })
      })
    })
  }
}
const p1 = new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve(111)
  }, 1000)
})
const p2 = new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve(222)
  }, 2000)
})
const p3 = new Promise((resolve, reject) => {
  setTimeout(() => {
    reject(333)
  }, 3000)
})

// 等所有的都有结果了之后，才执行then
// 如果有一个是err，那么就执行catch
HYPromise.allSettled([p1, p2, p3]).then(res => {
  console.log(res);
}).catch(err => {
  console.log(err);
})

```

实现了allSettled



race方法

```js
// ES6 ES2015
// https://promisesaplus.com/
const PROMISE_STATUS_PENDING = 'pending'
const PROMISE_STATUS_FULFILLED = 'fulfilled'
const PROMISE_STATUS_REJECTED = 'rejected'

// 工具函数
function execFunctionWithCatchError(execFn, value, resolve, reject) {
  try {
    const result = execFn(value)
    resolve(result)
  } catch(err) {
    reject(err)
  }
}

class HYPromise {
  constructor(executor) {
    this.status = PROMISE_STATUS_PENDING
    this.value = undefined
    this.reason = undefined
    this.onFulfilledFns = []
    this.onRejectedFns = []

    const resolve = (value) => {
      if (this.status === PROMISE_STATUS_PENDING) {
        // 添加微任务
        queueMicrotask(() => {
          if (this.status !== PROMISE_STATUS_PENDING) return
          this.status = PROMISE_STATUS_FULFILLED
          this.value = value
          this.onFulfilledFns.forEach(fn => {
            fn(this.value)
          })
        });
      }
    }

    const reject = (reason) => {
      if (this.status === PROMISE_STATUS_PENDING) {
        // 添加微任务
        queueMicrotask(() => {
          if (this.status !== PROMISE_STATUS_PENDING) return
          this.status = PROMISE_STATUS_REJECTED
          this.reason = reason
          this.onRejectedFns.forEach(fn => {
            fn(this.reason)
          })
        })
      }
    }

    try {
      executor(resolve, reject)
    } catch (err) {
      reject(err)
    }
  }

  then(onFulfilled, onRejected) {
    onRejected = onRejected === undefined ? err => {throw err} : onRejected;
    onFulfilled = onFulfilled === undefined ? value => {return value} : onFulfilled;
    return new HYPromise((resolve, reject) => {
      // 1.如果在then调用的时候, 状态已经确定下来
      if (this.status === PROMISE_STATUS_FULFILLED && onFulfilled) {
        // try {
        //   const value = onFulfilled(this.value)
        //   resolve(value)
        // } catch(err) {
        //   reject(err)
        // }
        execFunctionWithCatchError(onFulfilled, this.value, resolve, reject)
      }
      if (this.status === PROMISE_STATUS_REJECTED && onRejected) {
        // try {
        //   const reason = onRejected(this.reason)
        //   resolve(reason)
        // } catch(err) {
        //   reject(err)
        // }
        execFunctionWithCatchError(onRejected, this.reason, resolve, reject)
      }

      // 2.将成功回调和失败的回调放到数组中
      if (this.status === PROMISE_STATUS_PENDING) {
        onFulfilled && this.onFulfilledFns.push(() => {
          // try {
          //   const value = onFulfilled(this.value)
          //   resolve(value)
          // } catch(err) {
          //   reject(err)
          // }
          execFunctionWithCatchError(onFulfilled, this.value, resolve, reject)
        })
        onRejected && this.onRejectedFns.push(() => {
          // try {
          //   const reason = onRejected(this.reason)
          //   resolve(reason)
          // } catch(err) {
          //   reject(err)
          // }
          execFunctionWithCatchError(onRejected, this.reason, resolve, reject)
        })
      }
    })
  }
  catch(onRejected) {
    // 第一个参数不需要传
    return this.then(undefined, onRejected)
  }
  finally(onFinally) {
    this.then(() => {
      onFinally()
    }, () => {
      onFinally();
    });
  }
  static resolve(value) {
    return new HYPromise(resolve => resolve(value))
  }
  static reject(reason) {
    return new HYPromise((resolve, reject) => reject(reason))
  }
  static all(promises) {

    return new HYPromise((resolve, reject) => {
      const values = [];
      promises.forEach(promise => {
        promise.then(res => {
          values.push(res);
          if(values.length === promises.length) {
            resolve(values);
          }
        }, err => {
          reject(err);
        })
      })
    })
  }
  static allSettled(promises) {
    return new HYPromise((resolve, reject) => {
      const results = [];
      promises.forEach(promise => {
        promise.then(res => {
          results.push({status: PROMISE_STATUS_FULFILLED, value: res})
          if(results.length === promises.length) {
            resolve(results)
          }
        }, err => {
          results.push({status: PROMISE_STATUS_REJECTED, value: err})
          if(results.length === promises.length) {
            resolve(results)
          }
        })
      })
    })
  }
  static race(promises) {
    return new HYPromise((resolve, reject) => {
      promises.forEach(promise => {
        promise.then(res => {
          resolve(res);
        }, err => {
          reject(err);
        })
      })
    })
  }
  static any(promises) {
    
  }
}
const p1 = new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve(111)
  }, 4000)
})
const p2 = new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve(222)
  }, 5000)
})
const p3 = new Promise((resolve, reject) => {
  setTimeout(() => {
    reject(333)
  }, 3000)
})

HYPromise.race([p1, p2, p3]).then(res => {
  console.log(res)
}).catch(err => {
  console.log('err', err)
})

```

 

实现了any方法

```js
// ES6 ES2015
// https://promisesaplus.com/
const PROMISE_STATUS_PENDING = 'pending'
const PROMISE_STATUS_FULFILLED = 'fulfilled'
const PROMISE_STATUS_REJECTED = 'rejected'

// 工具函数
function execFunctionWithCatchError(execFn, value, resolve, reject) {
  try {
    const result = execFn(value)
    resolve(result)
  } catch(err) {
    reject(err)
  }
}

class HYPromise {
  constructor(executor) {
    this.status = PROMISE_STATUS_PENDING
    this.value = undefined
    this.reason = undefined
    this.onFulfilledFns = []
    this.onRejectedFns = []

    const resolve = (value) => {
      if (this.status === PROMISE_STATUS_PENDING) {
        // 添加微任务
        queueMicrotask(() => {
          if (this.status !== PROMISE_STATUS_PENDING) return
          this.status = PROMISE_STATUS_FULFILLED
          this.value = value
          this.onFulfilledFns.forEach(fn => {
            fn(this.value)
          })
        });
      }
    }

    const reject = (reason) => {
      if (this.status === PROMISE_STATUS_PENDING) {
        // 添加微任务
        queueMicrotask(() => {
          if (this.status !== PROMISE_STATUS_PENDING) return
          this.status = PROMISE_STATUS_REJECTED
          this.reason = reason
          this.onRejectedFns.forEach(fn => {
            fn(this.reason)
          })
        })
      }
    }

    try {
      executor(resolve, reject)
    } catch (err) {
      reject(err)
    }
  }

  then(onFulfilled, onRejected) {
    onRejected = onRejected === undefined ? err => {throw err} : onRejected;
    onFulfilled = onFulfilled === undefined ? value => {return value} : onFulfilled;
    return new HYPromise((resolve, reject) => {
      // 1.如果在then调用的时候, 状态已经确定下来
      if (this.status === PROMISE_STATUS_FULFILLED && onFulfilled) {
        // try {
        //   const value = onFulfilled(this.value)
        //   resolve(value)
        // } catch(err) {
        //   reject(err)
        // }
        execFunctionWithCatchError(onFulfilled, this.value, resolve, reject)
      }
      if (this.status === PROMISE_STATUS_REJECTED && onRejected) {
        // try {
        //   const reason = onRejected(this.reason)
        //   resolve(reason)
        // } catch(err) {
        //   reject(err)
        // }
        execFunctionWithCatchError(onRejected, this.reason, resolve, reject)
      }

      // 2.将成功回调和失败的回调放到数组中
      if (this.status === PROMISE_STATUS_PENDING) {
        onFulfilled && this.onFulfilledFns.push(() => {
          // try {
          //   const value = onFulfilled(this.value)
          //   resolve(value)
          // } catch(err) {
          //   reject(err)
          // }
          execFunctionWithCatchError(onFulfilled, this.value, resolve, reject)
        })
        onRejected && this.onRejectedFns.push(() => {
          // try {
          //   const reason = onRejected(this.reason)
          //   resolve(reason)
          // } catch(err) {
          //   reject(err)
          // }
          execFunctionWithCatchError(onRejected, this.reason, resolve, reject)
        })
      }
    })
  }
  catch(onRejected) {
    // 第一个参数不需要传
    return this.then(undefined, onRejected)
  }
  finally(onFinally) {
    this.then(() => {
      onFinally()
    }, () => {
      onFinally();
    });
  }
  static resolve(value) {
    return new HYPromise(resolve => resolve(value))
  }
  static reject(reason) {
    return new HYPromise((resolve, reject) => reject(reason))
  }
  static all(promises) {

    return new HYPromise((resolve, reject) => {
      const values = [];
      promises.forEach(promise => {
        promise.then(res => {
          values.push(res);
          if(values.length === promises.length) {
            resolve(values);
          }
        }, err => {
          reject(err);
        })
      })
    })
  }
  static allSettled(promises) {
    return new HYPromise((resolve, reject) => {
      const results = [];
      promises.forEach(promise => {
        promise.then(res => {
          results.push({status: PROMISE_STATUS_FULFILLED, value: res})
          if(results.length === promises.length) {
            resolve(results)
          }
        }, err => {
          results.push({status: PROMISE_STATUS_REJECTED, value: err})
          if(results.length === promises.length) {
            resolve(results)
          }
        })
      })
    })
  }
  static race(promises) {
    return new HYPromise((resolve, reject) => {
      promises.forEach(promise => {
        promise.then(res => {
          resolve(res);
        }, err => {
          reject(err);
        })
      })
    })
  }
  static any(promises) {
    return new HYPromise((resolve, reject) => {
      let reasons = []
      promises.forEach(promise => {
        promise.then(res => {
          resolve(res);
        }, err => {
          reasons.push(err);
          if(reasons.length === promises.length) {
            // 只能在浏览器测试
            reject(new AggregateError(reasons))
          }
        })
      })
    })
  }
}
const p1 = new Promise((resolve, reject) => {
  setTimeout(() => {
    reject(111)
  }, 4000)
})
const p2 = new Promise((resolve, reject) => {
  setTimeout(() => {
    reject(222)
  }, 5000)
})
const p3 = new Promise((resolve, reject) => {
  setTimeout(() => {
    reject(333)
  }, 3000)
})

HYPromise.any([p1, p2, p3]).then(res => {
  console.log(res)
}).catch(err => {
  console.log('err', err.errors)
})

```

# 简单总结手写Promise

## 一. Promise规范

https://promisesaplus.com/



## 二. Promise类设计

```js
class HYPromise {}
```

```js
function HYPromise() {}
```



## 三. 构造函数的规划

```js
class HYPromise {
  constructor(executor) {
   	// 定义状态
    // 定义resolve、reject回调
    // resolve执行微任务队列：改变状态、获取value、then传入执行成功回调
    // reject执行微任务队列：改变状态、获取reason、then传入执行失败回调
    
    // try catch
    executor(resolve, reject)
  }
}
```



## 四. then方法的实现

```js
class HYPromise {
  then(onFulfilled, onRejected) {
    // this.onFulfilled = onFulfilled
    // this.onRejected = onRejected
    
    // 1.判断onFulfilled、onRejected，会给默认值
    
    // 2.返回Promise resolve/reject
    
    // 3.判断之前的promise状态是否确定
    // onFulfilled/onRejected直接执行（捕获异常）
    
    // 4.添加到数组中push(() => { 执行 onFulfilled/onRejected 直接执行代码})
  }
}
```



## 五. catch方法

```js
class HYPromise {
  catch(onRejected) {
    return this.then(undefined, onRejected)
  }
}
```



## 六. finally

```js
class HYPromise {
  finally(onFinally) {
    return this.then(() => {onFinally()}, () => {onFinally()})
  }
}
```



## 七. resolve/reject



## 八. all/allSettled

核心：要知道new Promise的resolve、reject在什么情况下执行

all：

* 情况一：所有的都有结果
* 情况二：有一个reject

allSettled：

* 情况：所有都有结果，并且一定执行resolve



## 九.race/any

race:

* 情况：只要有结果

any:

* 情况一：必须等到一个resolve结果
* 情况二：都没有resolve，所有的都是reject