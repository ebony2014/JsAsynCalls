## 一、Javascript实现异步编程的过程以及原理
### 1、为什么要用Javascript异步编程
> 众所周知，Javascript是单线程，单线程即任务是串行的，后一个任务需要等待前一个任务的执行，这就可能出现长时间的等待。但由于类似ajax网络请求、setTimeout时间延迟、DOM事件的用户交互等，这些任务并不消耗 CPU，是一种空等，资源浪费，因此出现了异步。
通过将任务交给相应的异步模块去处理，主线程的效率大大提升，可以并行的去处理其他的操作。当异步处理完成，主线程空闲时，主线程读取相应的callback，进行后续的操作，最大程度的利用CPU。
  此时出现了同步执行和异步执行的概念，同步执行是主线程按照顺序，串行执行任务；异步执行就是cpu跳过等待，先处理后续的任务（CPU与网络模块、timer等并行进行任务）。由此产生了任务队列与事件循环，来协调主线程与异步模块之间的工作。

### 2、Javascript的任务队列和事件循环机制
![事件循环机制](https://upload-images.jianshu.io/upload_images/6148106-eefd4c8301dec9a7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
如上图为事件循环示例图（或JS运行机制图），流程如下：
> - step1：主线程读取JS代码，此时为同步环境，形成相应的堆和执行栈；  
> - step2:  主线程遇到异步任务，指给对应的异步进程进行处理（WEB API）;   
> - step3:  异步进程处理完毕（Ajax返回、DOM事件处罚、Timer到等），将相应的异步任务推入任务队列；    
> - step4: 主线程执行完毕，查询任务队列，如果存在任务，则取出一个任务推入主线程处理（先进先出）；    
> - step5: 重复执行step2、3、4；称为事件循环。    

执行的大意：    
> - 同步环境执行(step1) -> 事件循环1(step4) -> 事件循环2(step4的重复)…    


```ruby
let syncMethed = () => {
    console.log('我是同步任务1')
    setTimeout(() => {
        console.log('我是异步任务1')
    }, 3000)
    setTimeout(() => {
        console.log('我是异步任务2')
    }, 200)
    setTimeout(() => {
        console.log('我是异步任务3')
    }, 0)
    console.log('我是同步任务2')
}

syncMethed()
```
其中的异步进程有：    
1、类似onclick等，由浏览器内核的DOM binding模块处理，事件触发时，回调函数添加到任务队列中；    
2、setTimeout等，由浏览器内核的Timer模块处理，时间到达时，回调函数添加到任务队列中；     
3、Ajax，由浏览器内核的Network模块处理，网络请求返回后，添加到任务队列中。     
 
## 二、浅谈ES6里的Promise
### 1、Promise的简介
Promise 是异步编程的一种解决方案，比传统的解决方案——回调函数和事件——更合理和更强大。它由社区最早提出和实现，ES6 将其写进了语言标准，统一了用法，原生提供了Promise对象。
### 3、Promise的特点
- promise对象一旦建立就会立即执行
- 对象的状态不受外界影响。promise代表一个异步操作，有三种状态：
1、pending（进行中）
2、fulfilled（已成功）
3、rejected（已失败）
只有异步操作的结果，可以决定当前是哪一种状态，任何其他操作都无法改变这个状态。
- 一旦状态改变，就不会再变，任何时候都可以得到这个结果。
### 4、Promise的优缺点

优点：

- promise可以将异步操作以同步操作的流程表达出来，避免层层嵌套的回调函数。
- promise提供统一的接口，使得控制异步操作更容易

缺点：

- promise一旦建立就会立即执行，无法中途取消。
- 如果不设置回调函数，promise内部抛出的错误，不会反应到外部。（详细解说？）
- promise在pending状态时，无法知道当前进展到哪一个阶段。

### 5、Promise的基本用法
> - ES6 规定，Promise对象是一个构造函数，用来生成Promise实例。
> - Promise构造函数接受一个函数作为参数，该函数的两个参数分别是resolve和reject。它们是两个函数，由 JavaScript 引擎提供，不用自己部署。
> - resolve函数的作用是，将Promise对象的状态从“未完成”变为“成功”（即从 pending 变为 resolved），在异步操作成功时调用，并将异步操作的结果，作为参数传递出去；reject函数的作用是，将Promise对象的状态从“未完成”变为“失败”（即从 pending 变为 rejected），在异步操作失败时调用，并将异步操作报出的错误，作为参数传递出去。
> - Promise实例生成以后，可以用then方法分别指定resolved状态和rejected状态的回调函数。

```ruby
const promise = new Promise(function(resolve, reject) {
  if (/* 异步操作成功 */){
    resolve('成功');
  } else {
    reject('失败');
  }
});
promise.then(function(value) {
  console.log('success')
}, function(error) {
  console.log('fail')
});
```
或者可以这样写：
```ruby
promise.then(function(value) {
  console.log('success')
}).catch(function (error) {
  console.log('fail')
});

// catch为then的语法糖
```
> 上面的代码，只是new了一个对象实例，并没有调用，就执行了；对于这个情况，一般是把其嵌套在一个函数里面，避免立即执行，在需要的时候去运行这个函数（如下）。
```ruby
function timeout(ms) {
  return new Promise((resolve, reject) => {
    setTimeout(resolve('OK'), 0);
  });
}

timeout(100).then((value) => {
  console.log(value);
});
```
### 6、Promise的方法都有哪些
1.  [Promise.prototype.then()](http://es6.ruanyifeng.com/?search=%E7%AE%AD%E5%A4%B4%E5%87%BD%E6%95%B0&x=0&y=0#docs/promise#Promise.prototype.then())
> Promise 实例具有then方法，也就是说，then方法是定义在原型对象Promise.prototype上的。它的作用是为 Promise 实例添加状态改变时的回调函数。
> then方法返回的是一个新的Promise实例（注意，不是原来那个Promise实例）。因此可以采用链式写法，即then方法后面再调用另一个then方法。
2.  [Promise.prototype.catch()](http://es6.ruanyifeng.com/?search=%E7%AE%AD%E5%A4%B4%E5%87%BD%E6%95%B0&x=0&y=0#docs/promise#Promise.prototype.catch())
> Promise.prototype.catch方法是.then(null, rejection)的别名，用于指定发生错误时的回调函数。
3.  [Promise.prototype.finally()](http://es6.ruanyifeng.com/?search=%E7%AE%AD%E5%A4%B4%E5%87%BD%E6%95%B0&x=0&y=0#docs/promise#Promise.prototype.finally())
> finally方法用于指定不管 Promise 对象最后状态如何，都会执行的操作。该方法是 ES2018 引入标准的。
4.  [Promise.all()](http://es6.ruanyifeng.com/?search=%E7%AE%AD%E5%A4%B4%E5%87%BD%E6%95%B0&x=0&y=0#docs/promise#Promise.all())
> Promise.all方法用于将多个 Promise 实例，包装成一个新的 Promise 实例。多个 Promise 状态都改变成fulfilled或者有一个状态变成rejected，才会传递给then或者catch的回调函数里
5.  [Promise.race()](http://es6.ruanyifeng.com/?search=%E7%AE%AD%E5%A4%B4%E5%87%BD%E6%95%B0&x=0&y=0#docs/promise#Promise.race())
> Promise.race方法同样是将多个 Promise 实例，包装成一个新的 Promise 实例。只要多个 Promise 实例之中有一个实例率先改变状态，Promise.race的状态就跟着改变。那个率先改变的 Promise 实例的返回值，就传递给Promise.race的回调函数
6.  [Promise.resolve()](http://es6.ruanyifeng.com/?search=%E7%AE%AD%E5%A4%B4%E5%87%BD%E6%95%B0&x=0&y=0#docs/promise#Promise.resolve())
> 有时需要将现有对象转为 Promise 对象，Promise.resolve方法就起到这个作用。
```ruby
Promise.resolve('foo')
// 等价于
new Promise(resolve => resolve('foo'))
```
7.  [Promise.reject()](http://es6.ruanyifeng.com/?search=%E7%AE%AD%E5%A4%B4%E5%87%BD%E6%95%B0&x=0&y=0#docs/promise#Promise.reject())
> Promise.reject(reason)方法也会返回一个新的 Promise 实例，该实例的状态为rejected。
```ruby
Promise.reject('出错了');
// 等同于
new Promise((resolve, reject) => reject('出错了'))
```
### 7、Promise的应用
用promise实现多图预加载
```ruby
var imgs = ['image1.jpg', 'timg.jpg']
function perLoadImage(urls) {
    let pr = []
    urls.forEach((url) => {
        let p = loadImage(url).then((img) => {
            console.log(img)
            return img
        }).catch(err => err)
        pr.push(p)
    })

    Promise.all(pr).then((res)=>{
        console.log('图片加载完成', res)
        var imghtml = ''
        imgs.forEach(function (url) {
            imghtml += '<img src="'+ url +'"/>'
        })
        document.getElementById('images').innerHTML = imghtml
    }).catch((error)=> {
        console.log(error)
    }) 
}

function loadImage(url) {
    return new Promise((resolve, reject) => {
        let img = new Image()
        img.onload = () => resolve(url)
        img.onerror = () => reject('error')
        img.src = url
    })
}

perLoadImage(imgs)
```



### 7、Promise使用总结

- Promise确实很好的解决了回调地狱的问题，目前也已经很广泛的在使用。
- 但是它不太好理解，不管什么样的异步操作，被Promise一包装，看上去都是一堆then，语义方面还不够清晰。
- 另外一点，引用阮老师的说法，Promise 的写法只是回调函数的改进，使用then方法以后，异步任务的两段执行看得更清楚了，除此以外，并无新意，也就是说，它没有从思想上去改进异步编程的实现方案。
- Promise 的最大问题是代码冗余，原来的任务被 Promise 包装了一下，不管什么操作，一眼看去都是一堆then，原来的语义变得很不清楚。

## 三、初识Generator 
Promise只是一种代码组织结构，但Generator具有全新的语法!!!
### 1、Generator 基本语法
> Generator 函数是 ES6 提供的一种异步编程解决方案，语法行为与传统函数完全不同。

从一个简单的Generator函数开始：
![Generator基本功](https://upload-images.jianshu.io/upload_images/6148106-10994880befc6897.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

> 总结一下，调用 Generator 函数，返回一个遍历器对象，代表 Generator 函数的内部指针。以后，每次调用遍历器对象的next方法，就会返回一个有着value和done两个属性的对象。value属性表示当前的内部状态的值，是yield表达式后面那个表达式的值；done属性是一个布尔值，表示是否遍历结束。


### 2、Generator的简单应用

Generator是实现状态机的最佳结构。比如，下面的clock函数就是一个常规写法的状态机。
```ruby
var ticking = true;
var clock = function() {
  if (ticking)
    console.log('Tick!');
  else
    console.log('Tock!');
  ticking = !ticking;
}
```
上面代码的clock函数一共有两种状态（Tick和Tock），每运行一次，就改变一次状态。这个函数如果用Generator实现，就是下面这样。
```ruby
var clock = function*() {
  while (true) {
    console.log('Tick!');
    yield;
    console.log('Tock!');
    yield;
  }
};
```
可以看到，Generator 函数实现的状态机不用设初始变量，不用切换状态，上面的Generator函数实现与ES5实现对比，可以看到少了用来保存状态的外部变量ticking，这样就更简洁，更安全（状态不会被非法篡改）、更符合函数式编程的思想，在写法上也更优雅。Generator之所以可以不用外部变量保存状态，是因为它本身就包含了第一个状态和第二个状态。
### 3、Generator的自动执行
> Generator 就是一个异步操作的容器。它的自动执行需要一种机制，当异步操作有了结果，能够自动交回执行权。

> 两种方法可以做到这一点。
（1）回调函数。将异步操作包装成 Thunk 函数，在回调函数里面交回执行权。
（2）Promise 对象。将异步操作包装成 Promise 对象，用then方法交回执行权。

1、基于 Thunk 函数的自动执行
```ruby
function run(fn) {
  var gen = fn();
  function next(err, data) {
    var result = gen.next(data);
    if (result.done) return;
    result.value(next);
  }
  next();
}
```
2、基于 Promise 对象的自动执行
```ruby
function run(gen){
  var g = gen();
  function next(data){
    var result = g.next(data);
    if (result.done) return result.value;
    result.value.then(function(data){
      next(data);
    });
  }
  next();
}
run(gen);
```
3、co 模块
> [co 模块](https://github.com/tj/co)是著名程序员 TJ Holowaychuk 于 2013 年 6 月发布的一个小工具，用于 Generator 函数的自动执行。

> co 就是上面那个自动执行器的扩展，它的源码只有几十行，非常简单。

## 四、终结者async函数
### 1、什么是 async 函数
> async 函数算是Generator的一个语法糖，使异步函数、回调函数在语法上看上去更像同步函数
```
async function asyncLoadData (urlOne, urlTwo) {
    let dataOne = await loadData (urlOne)
    let dataTwo = await loadData (urlTwo)
}

```
在上面的代码中，loadData方法是异步获取数据的方法
可以看到，在 async 函数中，出现了一个陌生的关键字await——这个关键字只能够在 async 函数中使用，否则将会报错，它的意思是紧跟在其后面的表达式需要被等待执行结果
写成 generator 的话，应该是类似下面的函数：

```
function * asyncLoadData (urlOne, urlTwo) {
    let dataOne = yield loadData (urlOne)
    let dataTwo = yield loadData (urlTwo)
}
```
但是 generator 与 async 的区别并不仅仅是将*改为async，将yield改为await
> generator 和 async 的区别:
> - **内置执行器:** Generator 函数的执行必须靠执行器，所以才有了co模块，而async函数自带执行器。也就是说，async函数的执行，与普通函数一模一样，只要一行。
> - **更好的语义:**  async和await，比起星号和yield，语义更清楚了。async表示函数里有异步操作，await表示紧跟在后面的表达式需要等待结果。
> - **更广的适用性:** co模块约定，yield命令后面只能是 Thunk 函数或 Promise 对象，而async函数的await命令后面，可以是 Promise 对象和原始类型的值（数值、字符串和布尔值，但这时等同于同步操作）。
> - **返回值是 Promise:** async函数的返回值是 Promise 对象，这比 Generator 函数的返回值是 Iterator 对象方便多了。你可以用then方法指定下一步的操作。

进一步说，async函数完全可以看作多个异步操作，包装成的一个 Promise 对象，而await命令就是内部then命令的语法糖。
### 2、async 语法
- async函数返回一个 Promise 对象。
- async函数内部return语句返回的值，会成为then方法回调函数的参数。
```
async function f() {
  return 'hello world';
}

f().then(v => console.log(v))
// "hello world"
```
同样，当 async 函数内部抛出一个错误时，也会被 catch 到，下面三种 catch 错误的方式都可以：
```
async function errorTest () {
    throw new Error('this is an error');
}

// 在 then 的回调中捕获错误
errorTest().then(
    resolve => console.log(resolve),
    error => console.log(error)
)

// 在 Promise 的 catch 方法中捕获
errorTest().catch(
    error => console.log(error)
)

// 在 try...catch 语句中捕获
try{
    errorTest()
} catch (error) {
    console.log(error)
}
```

### 3、await 命令
> 前面说了await命令后面可以是Promise也可以是普通数据类型，但如果是普通数据类型的话，会自动转换成状态为resolve的Promise

如果await后面的Promise状态转变成了reject，那么整个 async 函数都会停止执行，并且抛出相应的错误。即使这里没有return，也一样可以传入错误回调的参数

所以当一个 async 函数中有多个 await命令时，如果不想因为一个出错而导致其与的都无法执行，应将await放在try...catch语句中执行
```
async function testAwait () {
    try {
        var p1 = await fun(1, 2)
        console.log(p1)
    } catch (error) {
        console.log(error, 'try')
    }
    var p2 = await fun(3, 3)
    console.log(p2)
}
```
**并发执行 await 命令**
```
// 方法 1
let [res1, res2] = await Promise.all([func1(), func2()])

// 方法 2
let func1Promise = func1()
let func2Promise = func2()
let res1 = await func1Promise
let res2 = await func2Promise
```
### 4、与其他异步处理方法的比较
> Async 函数的实现最简洁，最符合语义，几乎没有语义不相关的代码。它将 Generator 写法中的自动执行器，改在语言层面提供，不暴露给用户，因此代码量最少。如果使用 Generator 写法，自动执行器需要用户自己提供。
