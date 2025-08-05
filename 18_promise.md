## 异步任务的处理

在ES6出来之后，有很多关于Promise的讲解、文章，也有很多经典的书籍讲解Promise

- 虽然等你学会Promise之后，会觉得Promise不过如此，但是在初次接触的时候都会觉得这个东西不好理解；

那么这里我从一个实际的例子来作为切入点：

- 我们调用一个函数，这个函数中发送网络请求（我们可以用定时器来模拟）；
- 如果发送网络请求成功了，那么告知调用者发送成功，并且将相关数据返回过去；
- 如果发送网络请求失败了，那么告知调用者发送失败，并且告知错误信息；

```js

// request.js
function requestData(url, successCallback, failtureCallback) {
  // 模拟网络请求
  setTimeout(() => {
    // 拿到请求的结果
    // url传入的是coderwhy, 请求成功
    if (url === "coderwhy") {
      // 成功
      let names = ["abc", "cba", "nba"]
      successCallback(names)
    } else { // 否则请求失败
      // 失败
      let errMessage = "请求失败, url错误"
      failtureCallback(errMessage)
    }
  }, 3000);
}

// main.js
requestData("kobe", (res) => {
  console.log(res)
}, (err) => {
  console.log(err)
})
```

以上是处理回调的一种方案但是，这种回调的方式有很多的弊端:

- 如果是我们自己封装的requestData,那么我们在封装的时候必须要自己设计好callback名称, 并且使用好
- 如果我们使用的是别人封装的requestData或者一些第三方库, 那么我们必须去看别人的源码或者文档, 才知道它这个函数需要怎么去获取到结果





## 什么是Promise呢？

在上面的解决方案中，我们确确实实可以解决请求函数得到结果之后，获取到对应的回调，但是它存在两个主要的问题：

- 第一，我们需要自己来设计回调函数、回调函数的名称、回调函数的使用等；
- 第二，对于不同的人、不同的框架设计出来的方案是不同的，那么我们必须耐心去看别人的源码或者文档，以 便可以理解它这个函数到底怎么用；

我们来看一下Promise的API是怎么样的：

- Promise是ES6新增的

- Promise是一个类（构造函数），可以翻译成承诺、许诺 、期约；
- 当我们需要给予调用者一个承诺：待会儿我会给你回调数据时，就可以创建一个Promise的对象；
- 在通过new创建Promise对象时，我们需要传入一个回调函数，我们称之为executor（执行）
  - 这个回调函数会被立即执行，并且给传入另外两个回调函数resolve、reject；
  - 当我们调用resolve回调函数时，会执行Promise对象的then方法传入的回调函数；
  - 当我们调用reject回调函数时，会执行Promise对象的catch方法传入的回调函数；



## Promise的代码结构

我们来看一下Promise代码结构：

```js
const promise = new Promise((resolve, reject) => {
  // 调用resolve, 那么then传入的回调会被执行
  resolve('哈哈哈')
  
  // 调用reject, 那么catch传入的回调会被执行
  reject('错误信息')
})

promise.then(res => {
  console.log(res)
}).catch(err => {
  console.log(err)
})
```

new Promise接收的是一个回调函数，这个回调函数被称为executor（执行者），并且这个回调函数会立即执行，为什么立即执行，因为它内部类似：

```js
class Promise {
  constructor(callback) {
    callback()	// 这里就是传入的函数(executor),会立即执行
  }
}
```

代码执行是这样的：

```js
const promise = new Promise(() => {
  console.log('promise传入的函数被执行')
})

// promise传入的函数被执行
```

executor有两个参数

```js
// 传入的这个函数被称之为executor
//	resolve: 回调函数
//	reject: 回调函数
const promise = new Promise((resolve, reject) => {
  console.log('promise传入的函数被执行')
})
```

拿到Promise， 在成功的时候调resolve，在失败的时候调reject

resolve基本使用：

```js
function foo() {
  return new Promise((resolve, reject) => {
    resolve()	// 当执行resolve的时候就会调用then里面传入的回调函数
  })
}

const fooPromise = foo()

// then方法传入的回调函数，会在Promise执行resolve函数时，被回调
fooPromise.then(() => {
  console.log('调用resolve之后会执行此函数')
})
// 调用resolve之后会执行此函数
```



reject基本使用：

```js
function foo() {
  return new Promise((resolve, reject) => {
    reject()	// 当执行reject的时候就会调用catch里面传入的回调函数
  })
}

const fooPromise = foo()

// catch方法传入的回调函数，会在Promise执行reject函数时，被回调
fooPromise.catch(() => {
  console.log('调用reject之后会执行此函数')
})
// 调用reject之后会执行此函数
```



## Promise重构请求

那么有了Promise，我们就可以将之前的代码进行重构了：

```js
// request.js
function requestData(url,) {
  // 异步请求的代码会被放入到executor中
   //给别人返回一个承诺 
  return new Promise((resolve, reject) => {
    // 模拟网络请求
    setTimeout(() => {
      // 拿到请求的结果
      // url传入的是coderwhy, 请求成功
      if (url === "coderwhy") {
        // 成功
        let names = ["abc", "cba", "nba"]
        resolve(names)
      } else { // 否则请求失败
        // 失败
        let errMessage = "请求失败, url错误"
        reject(errMessage)
      }
    }, 3000);
  })
}

// main.js， node需要这样写
// then有两个回调函数，第一个是成功的回调，第二个是失败的回调
const promise = requestData("coderwhy")
promise.then((res) => {
  console.log("请求成功:", res)
}, (err) => {
   console.log("请求失败:", err)
})
// 或者这样写
Promise.then(res => {
    console.log('res', res)
}).catch((err) => {
    console.log(err)
})

// 在node不能这样写
Promise.then(res => {
    console.log('请求成功,=', res)
})

Promise.cath(() => {
    console.log('请求失败')
})

```

成功的时候调resolve，失败的时候调reject

拥有promise之后，不需要知道里面的回调函数怎么写的，只需要知道如果返回的是promise，调用这个promise就行，成功的话就调用.then并传入回调函数，失败的话调用.catch传入失败的回调函数就行了。

promise还有链式调用写法

```js
new Promise((resolve, reject) => {
  resolve()
}).then(res => {
  
}).catch(err => {
  
})
```



上面Promise使用过程，可以将它划分成三个状态：

待定（pending）: 初始状态，既没有被兑现，也没有被拒绝；

- 当执行executor中的代码时，处于该状态；

已兑现（fulfilled）: 意味着操作成功完成；

- 执行了resolve时，处于该状态；

已拒绝（rejected）: 意味着操作失败;

- 执行了reject时，处于该状态；

Promise是一种机制，所有的调用方法，调用模式都已经给你规范好了，这样大家就都统一了，不会出现每个人在设计回调或者其他方式的时候没有办法达到统一，这个时候就要花费时间来沟通或者看文档，所以promise能解决这种问题



## Executor

Executor是在创建Promise时需要传入的一个回调函数，这个回调函数会被立即执行，并且传入两个参数：

```js
new Promise((resolve, reject) => {
  console.log('executor代码')
})
```

通常我们会在Executor中确定我们的Promise状态：

- 通过resolve，可以兑现（fulfilled）Promise的状态，我们也可以称之为已决议（resolved）；
- 通过reject，可以拒绝（reject）Promise的状态；

这里需要注意：一旦状态被确定下来，Promise的状态会被 锁死，该Promise的状态是不可更改的

- 在我们调用resolve的时候，如果resolve传入的值本身不是一个Promise，那么会将该Promise的状态变成兑现（fulfilled）；
- 在之后我们去调用reject时，已经不会有任何的响应了（并不是这行代码不会执行，而是无法改变Promise状 态）；

```js
new Promise((resolve, reject) => {
	//  阶段1：pending
}).then(res => {
	// 阶段2：fulfilled、resolve(固定、已敲定)
}).catch(err => {
	// 阶段2：rejected(已拒绝)
})
```

状态一旦确定下来，就是不可更改的

```js
new Promise((resolve, reject) => {
  // 阶段1：pending
  resolve()
  console.log('----')	// 这行代码会执行
  reject() // 这里不可以更改状态了，因为上面调用了resolve，表示状态已经确定了，那么不能修改状态了，这行代码没有意义了
}).then(res => {
	// 阶段2：调用resolve执行这个函数
}).catch(err => {
	// 阶段2：调用reject执行这个函数
})
```



## resolve不同值的区别

情况一：如果resolve传入一个普通的值或者对象，那么这个值会作为then回调的参数；

情况二：如果resolve中传入的是另外一个Promise，那么这个新Promise会决定原Promise的状态：

情况三：如果resolve中传入的是一个对象，并且这个对象有实现then方法，那么会执行该then方法，并且根据 then方法的结果来决定Promise的状态：

```js
// 情况1：传入普通的值，pending -> fulfilled
new Promise((resolve, reject) => {
  resolve("aaaaaa")
}).then(res => {
  console.log('res1', res)
}).catch(err => {
  console.log('err1', err)
})
// res1 aaaaaa

// 情况2：传入一个Promise，那么当前的Promise的状态会由传入的Promise来决定，相当于状态进行了移交
new Promise((resolve, reject) => {
  resolve(new Promise((resolve1, reject) => {
    resolve1('resolve')
  }))
}).then(res => {
  console.log('res1', res)
}).catch(err => {
  console.log('err1', err)
})
// res1 resolve

// 情况3：传入一个对象, 并且这个对象有实现then方法(并且这个对象是实现了thenable接口)，那么也会执行该then方法, 并且由该then方法决定后续状态
new Promise((resolve, reject) => {
  resolve({
    then: function(resolve, reject) {
      resolve("resolve message")
    }
  })
}).then(res => {
  console.log("res:", res)
}, err => {
  console.log("err:", err)
})
// res: resolve message

```

> // eatable/runable, 也就是在这个对象里面实现了这么一个接口，当前这个对象可以称为实现了eatable接口的对象或者实现了runable接口的对象
> const obj = {
>   eat: function() {},
>   run: function() {}
> }



## then方法–接受两个参数

then方法是Promise对象上的一个方法：它其实是放在Promise的原型上的 Promise.prototype.then

```js
console.log(Object.getOwnPropertyDescriptors(Promise.prototype))

// {
//   constructor: {
//     value: [Function: Promise],
//     writable: true,
//     enumerable: false,
//     configurable: true
//   },
//   then: {
//     value: [Function: then],
//     writable: true,
//     enumerable: false,
//     configurable: true
//   },
//   catch: {
//     value: [Function: catch],
//     writable: true,
//     enumerable: false,
//     configurable: true
//   },
//   finally: {
//     value: [Function: finally],
//     writable: true,
//     enumerable: false,
//     configurable: true
//   },
//   [Symbol(Symbol.toStringTag)]: {
//     value: 'Promise',
//     writable: false,
//     enumerable: false,
//     configurable: true
//   }
// }
```

这些都是对象方法。

then方法接受两个参数：

- fulfilled的回调函数：当状态变成fulfilled时会回调的函数；
- reject的回调函数：当状态变成reject时会回调的函数；

```js
const promise = new Promise((resolve, reject) => {
  console.log('executor代码')
})

promise.then(res => {
  // fulfilled的回调函数
  console.log('res:', err)
}, err => {
  // reject的回调函数
  console.log('err:', err)
})
// 等价于
promise.then(res => {
  console.log('res:', res)
}).catch(err => {
  console.log('err:', err)
})
```



## then方法 – 多次调用

一个Promise的then方法是可以被多次调用的：

- 每次调用我们都可以传入对应的fulfilled回调；
- 当Promise的状态变成fulfilled的时候，这些回调函数都会被执行；

```js
const promise = new Promise((resolve, reject) => {
  console.log('executor代码')
  resolve('执行resolve')
})

promise.then(res => {
  console.log('res1:', res)
})
promise.then(res => {
  console.log('res2:', res)
})
promise.then(res => {
  console.log('res3:', res)
})

// executor代码
// res1: 执行resolve
// res2: 执行resolve
// res3: 执行resolve
```



## then方法 – 返回值

then方法本身是有返回值的，它的返回值是一个Promise，所以我们可以进行如下的链式调用：

- 但是then方法返回的Promise到底处于什么样的状态呢？

- 当then方法中的回调函数本身在执行的时候，那么它处于pending状态；

- 当then方法中的回调函数返回一个结果时，那么它处于fulfilled状态，并且会将结果作为resolve的参数；

  - 情况一：返回一个普通的值；

  - 情况二：返回一个Promise；

  - 情况三：返回一个thenable值；
  - 情况四：未返回任何值，默认返回的是undefined

- 当then方法抛出一个异常时，那么它处于reject状态；

```js
// 情况一：返回一个普通的值；
const promise = new Promise((resolve, reject) => {
  resolve('message')
})
promise.then(res => {
  return 'aaa'
}).then(res => {
  console.log(res)
  return 'bbb'
}).then(res => {
  console.log(res)
})
// aaa
// bbb

// 情况二：返回一个Promise；
const promise = new Promise((resolve, reject) => {
  resolve('message')
})
promise.then(res => {
  return new Promise((resolve1, reject1) => {
    resolve1('bbb')
  })
}).then(res => {
  console.log(res)
})
// bbb

// 情况三：返回一个thenable值；
const promise = new Promise((resolve, reject) => {
  resolve('message')
})
promise.then(res => {
  return {
    then: function (resolve, reject) {
      resolve('ccc')
    }
  }
}).then(res => {
  console.log(res)
})
// ccc
```

调用reject或者抛出一个错误，将回调then的第二个参数

```js
// 调用reject
const promise = new Promise((resolve, reject) => {
  reject('err message')
})

promise.then(undefined, (err) => {
  console.log('err:', err)
})
// err: err message

// 抛出错误
const promise = new Promise((resolve, reject) => {
  throw new Error('err message')
})

promise.then(undefined, (err) => {
  console.log('err:', err)
})
// err: Error: err message
```

当executor抛出异常时或者调用reject的时候，会调用错误捕获的回调函数的，也就是then的第二个函数

但是上面这种阅读性比较差，所以有其他的写法



## catch方法 – 多次调用

catch方法也是Promise对象上的一个方法：它也是放在Promise的原型上的 Promise.prototype.catch

一个Promise的catch方法是可以被多次调用的：

- 每次调用我们都可以传入对应的reject回调；
- 当Promise的状态变成reject的时候，这些回调函数都会被执行；

```js
const promise = new Promise((resolve, reject) => {
  console.log('executor代码')
  reject('执行reject')
})

promise.catch(err => {
  console.log('err1:', err)
})
promise.catch(err => {
  console.log('err2:', err)
})
promise.catch(err => {
  console.log('err3:', err)
})

// executor代码
// err1: 执行reject
// err2: 执行reject
// err3: 执行reject
```

但是这种写法，不符合promise/a+的规范(promise/a+没有catch)。

catch既可以捕获`executor/then`中抛出错误，也可以捕获`executor/then`中的`reject`

```js
// 1.捕获reject异常
new Promise((resolve, reject) => {
  reject('err message')
}).then(res => {
  console.log('res', res)
}).catch(err => {
  console.log('err:', err)
})
// err: err message

// 2.捕获抛出的错误
new Promise((resolve, reject) => {
  throw new Error('err message')
}).then(res => {
  console.log('res:', res)
}).catch(err => {
  console.log('err:', err)
})
// err: Error: err message

// 3.捕获then抛出的错误
new Promise((resolve, reject) => {
  resolve('resolve message')
}).then(res => {
  console.log('res:', res)
  throw new Error('err message')
}).catch(err => {
  console.log('err:', err)
})
// res: resolve message
// err: Error: err message

// 捕获then中返回的promise且调用reject状态
new Promise((resolve, reject) => {
  resolve('resolve message')
}).then(res => {
  console.log('res:', res)
  return new Promise((resolve, reject) => {
    reject('then返回一个promise')
  })
}).catch(err => {
  console.log('err:', err)
})
// res: resolve message
// err: then返回一个promise

// 捕获then中返回的thenable且调用reject状态
new Promise((resolve, reject) => {
  resolve('resolve message')
}).then(res => {
  console.log('res:', res)
  return {
    then: function (resolve, reject) {
      reject('ccc')
    }
  }
}).catch(err => {
  console.log('err:', err)
})
// res: resolve message
// err: ccc
```

这样会报错

```js
const promise = new Promise((resolve, reject) => {
  reject('reject message')
})

promise.then(res => {
  console.log(res)
})

node:internal/process/promises:288
            triggerUncaughtException(err, true /* fromPromise */);
            ^

[UnhandledPromiseRejection: This error originated either by throwing inside of an async function without a catch block, or by rejecting a promise which was not handled with .catch(). The promise rejected with the reason "reject message".] {
  code: 'ERR_UNHANDLED_REJECTION'
}
// 当我们这样写的时候，node在处理Promise回调的时候会找到.then，但是在then里面没有找到异常处理回调，所以他会直接抛出异常，并且说node异常退出
```

所以在node中必须在then里面有异常处理函数或者有catch处理函数

```js
const promise = new Promise((resolve, reject) => {
  reject('reject message')
})

promise.then(res => {
  console.log('res:', res)
}).catch(err => {
  console.log('err:', err)
})
// 或者
promise.then(res => {
  console.log('res:', res)
}, (err) => {
  console.log('err:', err)
})
```

错误捕获也是有顺序的，先捕获executor中的错误在catch中回调，或者调用reject的时候会现在catch中回调，如果捕获了executor中的错误或者在executor中调用了reject，那么.catch前的then的回调函数都不会调用，但是.catch后的.then会接着调用

```js
const promise = new Promise((resolve, reject) => {
  // 如果这里出现错误，catch会捕获，且不会执行任何.then函数
  reject('err-message')
})
promise.then(res => {
  console.log('res1:', res)
}).then(res => {
  console.log('res2:', res)
}).catch(err => {
  console.log('err1:', err)
}).then(res => {
  console.log('res3:', res)
}).catch(err => {
  console.log('err2:', err)
}).then(res => {
  console.log('res4:', res)
}).catch(err => {
  console.log('err3:', err)
})
// err1: err-message
// res3: undefined
// res4: undefined

const promise = new Promise((resolve, reject) => {
  resolve('resolve-message')
})
promise.then(res => {
  console.log('res1:', res)
}).then(res => {
  throw new Error('then error')
  console.log('res2:', res)
}).catch(err => {
  console.log('err1:', err)
}).then(res => {
  console.log('res3:', res)
}).catch(err => {
  console.log('err2:', err)
}).then(res => {
  console.log('res4:', res)
}).catch(err => {
  console.log('err3:', err)
})
// res1: resolve-message
// err1: Error: then error
// res3: undefined
// res4: undefined
```





## catch方法 – 返回值

事实上catch方法也是会返回一个Promise对象的，所以catch方法后面我们可以继续调用then方法或者catch方法：

- 下面的代码，后续是catch中的err2打印，还是then中的res打印呢？
- 答案是res打印，这是因为catch传入的回调在执行完后，默认状态依然会是fulfilled的；

```js
const promise = new Promise((resolve, reject) => {
  reject('reject-message')
})
promise.catch(res => {
  console.log('err1:', res)	// catch里面的返回值也会包裹一个Promise并且也分为普通的数据和.thenable和new Promise
}).catch(err => {
  console.log('err2:', err)
}).then(res => {
  console.log('res:', res)	
})
// err1: reject-message
// res2: undefined
```

那么如果我们希望后续继续执行catch，那么需要抛出一个异常：

```js
const promise = new Promise((resolve, reject) => {
  reject('reject-message')
})
promise.catch(res => {
  console.log('err1:', res)
  throw new Error('error message')
}).then(res => {
  console.log('res:', res)
}).catch(err => {
  console.log('err2:', err)
})
// err1: reject-message
// err2: Error: error message
```



## finally方法

finally是在ES9（ES2018）中新增的一个特性：表示无论Promise对象无论变成fulfilled还是reject状态，最终都会被执行的代码。

finally方法是不接收参数的，因为无论前面是fulfilled状态，还是reject状态，它都会执行。

```js
const promise = new Promise((resolve, reject) => {
  reject('reject')
  // resolve('fulfilled')
})
promise.then(res => {
  console.log('res:', res)
}).catch(err => {
  console.log('err:', err)
}).finally(() => {
  console.log('finally action')
})
// err: reject
// finally action
```



# 类方法

## resolve方法

前面我们学习的then、catch、finally方法都属于Promise的实例方法，都是存放在Promise的prototype上的。

- 下面我们再来学习一下Promise的类方法。

有时候我们已经有一个现成的内容了，希望将其转成Promise来使用，这个时候我们可以使用 Promise.resolve 方 法来完成。

- Promise.resolve的用法相当于new Promise，并且执行resolve操作：

```js
Promise.resolve('wts')
// 等价于
new Promise((resolve) => resolve('wts'))
```



resolve参数的形态：

- 情况一：参数是一个普通的值或者对象
- 情况二：参数本身是Promise
- 情况三：参数是一个thenable

```js
const promise = Promise.resolve({name: 'why'});	// 普通的值/promise/thenable对象

// 一样通过then来获取
promise.then(res => {
    console.log(res);
})

// { name: 'why' }
```

resolve方法支持,new Promise和thenable



## reject方法

reject方法类似于resolve方法，只是会将Promise对象的状态设置为reject状态。

Promise.reject的用法相当于new Promise，只是会调用reject：

```js
const promise = Promise.reject('reject message');	// 普通的值/promise/thenable对象
// 相当于
const promise2 = new Promise((resolve, reject) => {
    reject({name: 'why'})
})
```

Promise.reject传入的参数无论是什么形态，都会直接作为reject状态的参数传递到catch的。

```js
const promise = Promise.reject('reject message');	// 普通的值/promise/thenable对象

// 一样通过then来获取
promise.then(res => {
    console.log(res);
}, (err) => {
    console.log(err)
})
// 或者
promise.then(res => {
    console.log(res)
}).catch(err => {
    console.log(err)
})

// reject message
// reject message
```

reject方法不支持,new Promise和thenable



## all方法

另外一个类方法是Promise.all：

- 它的作用是将多个Promise包裹在一起形成一个新的Promise；
- 新的Promise状态由包裹的所有Promise共同决定：
  - 当所有的Promise状态变成fulfilled状态时，新的Promise状态为fulfilled，并且会将所有Promise的返回值组成一个数组；
  - 当有一个Promise状态为reject时，新的Promise状态为reject，并且会将第一个reject的返回值作为参数；

```js
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
        resolve(333)
    }, 3000)
})

// 需求： 所有的promise都变成fulfilled时，再拿到结果
// aaa如果加在里面，它也会变成promise
Promise.all([p2, p1, p3, 'aaa']).then(res => {
    console.log(res)
})

// [ 222, 111, 333, 'aaa' ]
```

它是按照输入数组的顺序返回的，那如果有意外错误怎么办呢？

```js
const p1 = new Promise((resolve, reject) => {
    setTimeout(() => {
        resolve(111)
    }, 1000)
})
const p2 = new Promise((resolve, reject) => {
    setTimeout(() => {
        reject(222)
    }, 2000)
})
const p3 = new Promise((resolve, reject) => {
    setTimeout(() => {
        resolve(333)
    }, 3000)
})

// 需求： 所有的promise都变成fulfilled时，再拿到结果
// aaa如果加在里面，它也会变成promise
// 如果有一个promise的状态是rejected, 那么整个promise.all就会变成rejected
Promise.all([ p2, p1, p3, 'aaa']).then(res => {
    console.log(res)
}).catch(err => {
    console.log(err)
})
// 222
```



## allSettled方法

all方法有一个缺陷：当有其中一个Promise变成reject状态时，新Promise就会立即变成对应的reject状态。

- 那么对于resolved的，以及依然处于pending状态的Promise，我们是获取不到对应的结果的；

在ES11（ES2020）中，添加了新的API Promise.allSettled：

- 该方法会在所有的Promise都有结果（settled），无论是fulfilled，还是reject时，才会有最终的状态；
- 并且这个Promise的结果一定是fulfilled的；
- 不管有没有rejected，都不会进入catch

```js
const p1 = new Promise((resolve, reject) => {
    setTimeout(() => {
        resolve(111)
    }, 1000)
})
const p2 = new Promise((resolve, reject) => {
    setTimeout(() => {
        reject(222)
    }, 2000)
})
const p3 = new Promise((resolve, reject) => {
    setTimeout(() => {
        resolve(333)
    }, 3000)
})

Promise.allSettled([p2, p1, p3]).then(res => {
  console.log(res)
}).catch(err => {
  console.log(err)
})

// [
//   { status: 'rejected', reason: 222 },
//   { status: 'fulfilled', value: 111 },
//   { status: 'fulfilled', value: 333 }
// ]
```

我们来看一下打印的结果：

- allSettled的结果是一个数组，数组中存放着每一个Promise的结果，并且是对应一个对象的；
- 这个对象中包含status状态，以及对应的value值；

```js
const p1 = new Promise((resolve, reject) => {
    setTimeout(() => {
        resolve(111)
    }, 1000)
})
const p2 = new Promise((resolve, reject) => {
    setTimeout(() => {
        reject(222)
    }, 2000)
})
const p3 = new Promise((resolve, reject) => {
    setTimeout(() => {
        resolve(333)
    }, 3000)
})

// 需求： 所有的promise都变成fulfilled时，再拿到结果
// aaa如果加在里面，它也会变成promise
// 如果有一个promise的状态是rejected, 那么整个promise.all就会变成rejected
Promise.allSettled([p2, p1, p3]).then(res => {
    console.log(res)
}).catch(err => {
    console.log(err)
})

// [
//   { status: 'rejected', reason: 222 },
//   { status: 'fulfilled', value: 111 },
//   { status: 'fulfilled', value: 333 }
// ]
```

会在结果里面标志每一个promise的状态，比如有些promise的状态就是rejected



## race方法

race: 竞技，竞赛的意思

如果有一个Promise有了结果，我们就希望决定最终新Promise的状态，那么可以使用race方法：

- race是竞技、竞赛的意思，表示多个Promise相互竞争，谁先有结果，那么就使用谁的结果；
- 也就是只要有一个有结果了，race方法就判断出最终的结果了

```js
const p1 = new Promise((resolve, reject) => {
    setTimeout(() => {
        resolve(111)
    }, 1000)
})
const p2 = new Promise((resolve, reject) => {
    setTimeout(() => {
        reject(222)
    }, 2000)
})
const p3 = new Promise((resolve, reject) => {
    setTimeout(() => {
        resolve(333)
    }, 3000)
})

Promise.race([p2, p1, p3]).then(res => {
    console.log(res)
}).catch(err => {
    console.log(err)
})

// 111
```

如果有promise先拒绝了，那么整个的结果就拒绝了

```js
const p1 = new Promise((resolve, reject) => {
    setTimeout(() => {
        resolve(111)
    }, 1000)
})
const p2 = new Promise((resolve, reject) => {
    setTimeout(() => {
        reject(222)
    }, 2000)
})
const p3 = new Promise((resolve, reject) => {
    setTimeout(() => {
        resolve(333)
    }, 500)
})

Promise.race([p2, p1, p3]).then(res => {
    console.log(res)
}).catch(err => {
    console.log(err)
})

// 333
```

那如果就想要等到一个结果，不管你reject，我就要等一个resolve, 怎么办呢



## any方法

any方法是ES12中新增的方法，和race方法是类似的：

- any方法会等到一个fulfilled状态，才会决定新Promise的状态；
- 如果所有的Promise都是reject的，那么也会等到所有的Promise都变成rejected状态；

```js
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
        resolve(333)
    }, 3000)
})

Promise.any([p2, p1, p3]).then(res => {
    console.log(res)
}).catch(err => {
    console.log(err)
})

// 111
```

如果所有的Promise都是reject的，那么会报一个AggregateError的错误。

```js
const p1 = new Promise((resolve, reject) => {
    setTimeout(() => {
        reject(111)
    }, 1000)
})
const p2 = new Promise((resolve, reject) => {
    setTimeout(() => {
        reject(222)
    }, 2000)
})
const p3 = new Promise((resolve, reject) => {
    setTimeout(() => {
        reject(333)
    }, 3000)
})

Promise.any([p2, p1, p3]).then(res => {
    console.log(res)
}).catch(err => {
    console.log('err1:', err)
  	console.log('err2:', err.errors)
})

// err1: [AggregateError: All promises were rejected] {
//   [errors]: [ 222, 111, 333 ]
// }
// err2: [ 222, 111, 333 ]
```

可以通过err.errors来拿到错误信息
