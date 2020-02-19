---
title: "Node.js异步流程控制"
date: 2017-09-20T11:09:28+08:00
draft: false
tags: ["node.js", "nonblocking"]
categories: ["language"]
---

Node.js使用事件驱动、非阻塞I/O模型，写出来的应用响应迅速、扩展性好。虽然异步调用在Node.js中是一把利器，但是需要我们合理的使用才能事半功倍。
<!--more-->

# Callback

第一种比较常见的异步调用方式就是注册回调函数，比如说setTimeout：

``` {.javascript}
setTimeout(() => {
  console.log('Hello!');
}, 1000);
```

其中注册的是
[箭头函数](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions)
，这个函数并不会马上执行，而是在大约1秒后执行。\
像读写文件这些很可能阻塞的操作，[fs模块](https://nodejs.org/api/fs.html)
就提供了很多接口，而这些接口可以注册回调函数，在操作有结果之后才通知你进行处理。\
不难理解，正是由于不用忙等待操作结果(non-blocking)，程序才有更多的机会处理用户的请求以及其它事务，不会卡在界面一动不动。\
当然回调函数并不是完美的，经常会嵌套太深，造成\"callback
hell\"，比如下面的伪代码：

``` {.javascript}
function handler(request, response) {
    User.get(request.user, function(err, user) {
        if (err) {
            response.send(err);
        } else {
            Contact.get(user.contact, function(err, contact) {
                if (err) {
                    return response.send(err);
                } else  {
                    ...
                }
            });
        }
    })
}
```

可以看出来，回调函数嵌套太深会造成可读性变差、可维护性变弱等。

# Promise

为了解决前面提到的问题，一个比较简洁的方案提出来了，那就是
[Promise](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise)
。\
Promise代表了异步操作的最终结果，可能成功，也可能失败，但一定会通知你。\
而你只要通过Promise的then方法就可以在成功的时候得到终值，失败的时候处理错误。\
Promise的then方法返回的仍然是一个Promise对象，也就是说你可以继续调用then方法，达到链式调用的效果：

``` {.javascript}
function(request, response) {
    var user, contact;

    User.get(request.user)
    .then(function(aUser) {
        user = aUser;
        return Contact.get(user.contact);
    })
    .then(function(aContact) {
        contact = aContact;
        ...
    })
    .catch(err => response.send(err));
}
```

可以看到，嵌套已经消失了，代码变得扁平化了。\
另外还有一个好处就是通过Promise的catch方法，错误得到了集中的处理，不再东一块、西一块。\
Promise提供了一个解决回调地狱的基础方法，但是对于稍微复杂的情况(循环、并发、集合等)，就有点有心无力，需要新的工具。

# Generator/Yield

作为一个过渡方案，一些模块(比如说 [co)](https://github.com/tj/co)通过
[Generator](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Generator)
和
[yield](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/yield)
使得多个异步调用可以像同步调用那样书写，比较自然、直观。\
虽然Generator的本意并不是解决前面提到的问题，不过通过Promise和yield的配合，使得问题得到了巧妙的解决。\
其中起到关键作用的是可以改变执行流程的yield，可以根据需要让渡执行权，然后在合适的时机以及可选的输入下继续执行。

``` {.javascript}
co(function *(){
  // resolve multiple promises in parallel
  var a = Promise.resolve(1);
  var b = Promise.resolve(2);
  var c = Promise.resolve(3);
  var res = yield [a, b, c];
  console.log(res);
  // => [1, 2, 3]
}).catch(onerror);
```

除了比较怪异的function\*和yield的写法之外，这个解决方案已经基本让人满意了。\
另外说一点题外话就是，Generator和yield在其他方面还有不少潜力可以挖，可能给人带来新的惊喜。

# Async/Await

前面之所以称generator/yield为过渡方案，是因为标准的解决方案已经出来，也就是async/await方案，但是需要较新版本的Node.js才支持。\
从某种意义来讲，async/await方案可以称为generator/yield的语法糖，使用形式上十分相似。

``` {.javascript}
function handler(req, res) {
  return co(function*() {
    yield new Promise((resolve, reject) => reject(new Error('Hang!')));
    res.send('Hello, World!');
  });
}

async function handler(req, res) {
  await new Promise((resolve, reject) => reject(new Error('Hang!')));
  res.send('Hello, World!');
}
```

async/await去除了怪异的function\*和yield，使用了含义更明确的async和await，另外还有不少细节上的改进。\
比如说以下几点：

-   通过赋值获取终值，通过try/catch处理异常
-   对于for循环、if条件能够跟同步调用一样使用，不用特别的处理
-   更精确、可读性更强的栈回溯信息

需要注意的是，await只能在async函数内使用，否则会产生语法错误。

# 小结

虽然说本文大部分内容都在讲异步调用，但是并不是说一定要异步调用，而是要根据业务逻辑来确定，同步调用与异步调用搭配使用，效果更佳。\
另外就是还有不少其他比较优秀的模块能够很好的控制异步调用流程，比如说
[Async](https://github.com/caolan/async) ,
[Bluebird](https://github.com/petkaantonov/bluebird)
等模块，大家可以根据需要选用。
