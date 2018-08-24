JavaScript是一种单线程语言，所谓单线程，即同一时间只能做一件事情。

因此JavaScript代码的执行需要按照一定的规定来调度，Eventloop就是负责调度执行顺序的。

***

我们先来看一个基本的JavaScript代码：

```javascript
console.log("1");

console.log("2");

console.log("3");
```

毫无疑问，如果执行这一段代码，得到的是：

```javascript
1
2
3
```

对于同步代码来说，执行是按照先后顺序的。

***

如果我们加上一点异步代码呢？

```javascript
console.log("1");

setTimeout( function fn() {
    console.log("2");
}, 1000);

console.log("3");

//输出结果
1
3
2
```

如果简单地来理解，setTimeout设置了一个定时器（在这个例子中设置了1000ms），当1000ms后，才会执行fn。

***

然而事情并不是这么简单，我们再看一个简单的例子。

```javascript
console.log("1");

setTimeout( function fn() {
    console.log("2");
}, 0);

console.log("3");

//输出结果
1
3
2
```

我们设置定时器定时0s，然而还是先执行定时器后面的代码，这是为什么？



在JavaScript中，其实有这样几个管理代码执行顺序的结构：

##### 调用栈（栈）

##### 宏任务队列（队列）： 宏任务有：setTimeout, setInterval, I/O, UI渲染

##### 微任务队列（队列）： 微任务有：Promise， process.NextTick(node.js)

##### webapis（堆）

按照这样的规定：

当一个块级作用域（函数，或者全局作用域块）被调用时，会加入`调用栈`。

当这个表达式执行完成，或函数被返回后，会弹出`调用栈`。

setTimeout（setInterval等）会设置一个定时器任务，定时器存放在`webapis`这个堆上

Promise会开启一个异步任务，这个操作也会放在`webapis`上

我们创建DOM事件监听的时候，监听函数也会存放在`webapis`上

当`webapis`上的某个任务结束了，会将其加入到对应的`任务队列`，。

当`调用栈`为空时，`微任务队列`中的项会加入`调用栈`（`宏任务队列`中的不会）。

当`调用栈`为空，且`微任务队列`为空时，`宏任务队列`中的项会加入`调用栈`



因此，再来看看刚才的代码，这次我们带上注释：

```javascript
//下面这个block将会被压入调用栈

//------- a block --------
console.log(1) //console log: 1

setTimeout( function fn() { //创建一个宏任务，放到webapis上
    console.log("2");
}, 0);
//这里W3C规定当设置事件不能小于4ms时，设置0其实会变成设置4ms


console.log(3); //console log: 3
//------------------------

//至此这个block执行结束，被弹出调用栈
//所以栈空了

//宏任务队列中第一个宏任务fn会入栈
//console log: 2
```

***

上面我们讲了Promise会创建一个微任务，微任务的优先级和宏任务是不同的，所以我们来看这样一个例子：

```javascript
//进入调用栈
//-------block-----------
console.log(1)
//console: 1
const fn = setTimeout(() => {  //4ms左右后加入宏任务队列
  console.log("Timeout");
}, 0)

for (let i = 0; i < 20000; i++) {
  Promise.resolve().then((res) => {  //加入微任务队列
    console.log("Promise")
  });
}

console.log(3);
//console: 3
//-----------------------
//退出调用栈，调用栈为空

//微任务队列开始逐个加入调用栈
//console: Promise
//console: ...
//console: Promise
//微任务队列执行完，为空，宏任务队列开始逐个加入调用栈
//console: Timeout
```

输出结果：

```javascript
1
3
Promise
...
Promise
Timeout
```

这个例子证明了微任务不执行完，宏任务是不会加入执行栈的。

***

我这里推荐几个比较有权威也比较容易理解的事件循环的相关参考资料 ：

[MDN文档](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/EventLoop)

[Philip Roberts: What the heck is the event loop anyway? ](https://www.youtube.com/watch?v=8aGhZQkoFbQ&t=1293s)：youtube视频

[THE EVENT LOOP](https://www.youtube.com/watch?v=0IsjjMRyIF8&t=1953s)：youtube视频