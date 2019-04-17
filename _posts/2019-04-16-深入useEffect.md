参考文章：https://overreacted.io/zh-hans/a-complete-guide-to-useeffect/

## 什么时候执行useEffect
准确的说，React会在函数组件每次得到需要渲染的html，且作用于DOM之后，才会调用useEffect中的函数。

形象的过程可以参考如下：
你的组件: 喂 React, 把我的状态设置为1。  
React: 给我状态为 1时候的UI。  
你的组件:  
  
给你需要渲染的内容: <p>You clicked 1 times</p>。  
记得在渲染完了之后调用这个effect： () => { document.title = 'You clicked 1 times' }。  
React: 没问题。开始更新UI，喂浏览器，我修改了DOM。  
Browser: 酷，我已经将更改绘制到屏幕上了。  
React: 好的， 我现在开始运行属于这次渲染的effect  
  
运行 () => { document.title = 'You clicked 1 times' }。  

## effect函数中使用的变量的值
都是该effect函数被声明的那次渲染时，组件的该变量的值

## effect函数返回的清除函数什么时候执行？
为了性能考虑，清除函数不会在下一次渲染前执行，而是在下一次渲染后执行。（因为在之前之后并不重要，都能清除掉）

形象的过程可参考如下：  
React 渲染{id: 20}的UI。  
浏览器绘制。我们在屏幕上看到{id: 20}的UI。  
React 清除{id: 10}的effect。  
React 运行{id: 20}的effect。  

## useEffect函数第二个参数有什么用？
第二个参数是一个数组，如果这个数组里的变量的值和上一次渲染都没有变化，则该effect在本次渲染中不会被执行。（可以理解为，告诉react，如果这些值没变，我在effect中做的事情没必要再做一遍）。  


## 你可能会陷入的误区
有一种很普遍的（也可能只有我自己）错误理解，就是认为effect函数的执行时机等同于以前的DidMount和didUpdate。  
但其实完全不能这么理解，因为mount和update其实都是类（class）组件才有的概念，对于函数组件来说，渲染一次就会执行一次，第一次渲染和之后的渲染并没有区别。  

所以在使用hooks的时候，一定要谨记这一点：  
在hooks的模型中，每一次渲染都是相同的（如果你想要第一次渲染和之后的渲染有不同的行为，那么你正在违反hooks的模型）