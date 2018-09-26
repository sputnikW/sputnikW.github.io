第一篇链接： [redux真的不复杂——源码解读](https://juejin.im/post/5b9617835188255c781c9e2f)
> 预备知识：1. 了解redux的基本使用； 2. [Context API](https://reactjs.org/docs/context.html)
>
> 了解redux原理更好，如果不了解，不妨先看看第一篇博客。

redux是一个状态管理的工具，本质上是一个js对象（包含状态，以及一些处理状态的方法）。

所以redux具有很强的适应性，可以配合其他工具/框架一起使用。

react-redux则是一个让你更容易地在react中使用redux的工具。

### 为什么需要redux

我们使用redux的目的是存储状态，在react中有存储状态的东西吗？

有，state。但是state有一些局限性。

对于一个组件来说，（用setState触发）state的变化会触发这个组件以及它所有子组件的更新，所以为了优化考虑，我们往往将state放在更“局部”的组件中，这样state的变化只会引起最少的（有必要的）更新。

那么如果我们的react应用有一些状态在多个地方都可能用到，特别是一些全局数据（比如当前用户信息，全局的通知等等），对于这些数据我们有两个选择：

1. 在各个局部组件中各存一份

   **优点是**：保证了数据的变化只会引起最小的组件更新，

   **缺点是**：

   - 难以保证各处的数据同步
   - 可能各处会重复请求相同的API，损失了一定性能

2. 在全局组件中存一份

   **优点是**：只需要在一个地方请求API，数据是完全同步的

   **缺点是**：数据的变化会引起整个应用大量的更新。

你会发现这两个选择各有优缺点，但仔细想想，你会发现其实我们有第三个选择：

**利用Context API。在全局组件外包一个Provider，将数据存在Provider上。子组件通过访问context来使用这些数据**

这样就兼顾了各个优点：

- 数据易同步
- 只需在一个地方请求API
- 全局数据的更新不会引起组件的更新（Context api的特性）

***

一切仿佛变得美好了，不是吗？

但是问题又来了：

- 全局（Provider上）的数据你如何管理（特别是当数据的结构复杂了之后）？
- 子组件如何根据需要更新这些数据？



这个时候你就会想到redux了，redux提供了一套优雅的管理状态的方案。

优雅在什么地方？接着往下看。

### 为什么需要react-redux

想要使用context api的方案，并且还要使用redux，那么你需要做的事情有：

1. 用redux创建一个store
2. 在react应用的最外层包一个Provider，store放在Provider上。
3. 子组件想要获取数据时：在组件外包一个Consumer，Consumer获取到store，传递给组件。
4. 当子组件想要更新数据时：调用store的dispatch方法触发store的更新。

差不多就这些，是不是也挺简单？

但是从技术上来讲，你需要做的事情有：

1. 自己写一个provider
2. 每次想要使用provider的数据的时候，自己写一个Consumer组件
3. 每次想要更新数据的时候，自己调用store的diapatch方法。

难受吗?

你需要react-redux，它帮你把这些操作都封装了起来。

### react-redux源码分析

为了源码更清晰，分析时只展示了一些核心代码，省略了错误处理，通用性处理等代码。建议你参照着[真正的源码](https://github.com/reduxjs/react-redux)阅读。

#### react-redux的使用

在看源码之前，先简单回忆一下react-redux的用法：

1. 从react-redux库引入一个Provider组件，将redux创建的store作为属性传递给这个Provider组件。
2. 给connect传递`mapStateToProps`和`mapDispatchToProps`两个参数（还有其他可选参数`mergeProps`,`options`），得到一个高阶组件【注1】。
3. 用高阶组件“包装”我们自己的组件，就能在自己的组件中得到对应的props。

> 【注1】高阶组件：输入为组件，输出为另一个组件的**函数**。

ok，我们来看看源码的结构：

![](https://user-gold-cdn.xitu.io/2018/9/26/1661435261042538?w=208&h=467&f=jpeg&s=38841)

还挺复杂，不过没关系，看看index.js:

```javascript
//index.js

import Provider, { createProvider } from './components/Provider'
import connectAdvanced from './components/connectAdvanced'
import connect from './connect/connect'

export { Provider, createProvider, connectAdvanced, connect }
```

createProvider和connectAdvanced这两个方法是定制react-redux的工作方式时使用的，我们一般不会直接用到，所以我们就从Provider和connect这两个模块入手。

#### Provider

`Provider.js`的源码比较少，结构也很简单:

```javascript
//Provider.js

//import略
//输出创建Provider的函数
export function createProvider(storeKey = 'store'){
    class Provider extends Component {
        // ...
    }
    
    return Provider
}

//默认输出创建后的Provider
export default createProvider()
```

这个文件提供了两个输出，一个是创建Provider的函数，还有一个是创建过的Provider。

下面我们来详细看看Provider这个组件具体是如何实现的：

```javascript
class Provider extends Component {
    // 访问context的钩子函数【注2】
    getChildContext() {
        // 返回一个对象，键是store的标识（可自定义），值是store
        return { [storeKey]: this[storeKey] }
    }
    
    constructor(props, context) {
        super(props, context)
        // 将我们（通过props）传进来的store存在自己的实例中。
        this[storeKey] = props.store
    }
    
    render() {
        // Children.only是react提供的API函数，
        // 作用是限制this.props.children只能是一个React元素，否则会报错
        return Children.only(this.props.children)
    }
}

Provider.childContextTypes = {
    // ...
}
```

看到这里，可以发现Provider的实现非常简单，只是**将我们（通过props）传进去的store，创建了一个context**而已。

> 注2：React可以通过在一个组件中设置`getChildContext`和`childContextTypes`来创建context，在其子组件中设置`contextTypes`属性来接收context

原来，react-redux只是将store放到Provider组件的context上。那么问题来了，

**问题1**：

**Provider的子组件如何使用store**?

***

#### Connect

顾名思义，connect函数的作用是——”连接“，**将一个正常的组件与我们的store连接起来**，这样子组件就可以“使用”store了。

然而实际上是如何实现连接的呢？如果你使用过react-redux的话，你就知道是：

1. 使用connect创建一个高阶组件
2. 然后用高阶组件包裹一个组件，向组件传递额外的props
3. 组件内部通过props，能够：
   - 读取store
   - 触发（dispatch）store中的action。



我们使用connect创建高阶组件时通常会传入`mapStateToProps`，`mapDispatchToProps`，然而高阶组件并没有直接将其通过props传进被包裹组件——而是经过**筛选和包装**后，再通过props传入一些东西（store的一部分分支，或者自动dispatch的action creator）。

**回答问题1**（Provider的子组件如何使用store？）：

**通过将“筛选和包装”后的与store相关的东西，通过props传入组件，来使用store**。



可见，**筛选和包装**是一个非常重要的任务，那么它是在connect中实现的吗？

我们来看看connect的源码：

```javascript
//connect.js

export function createConnect({
    connectHOC = connectAdvanced,  // 记住这个函数，后面会讲
    
    // 选择器工厂：根据一些配置生成一个选择器
    // （选择器工厂的实现，以及选择器的作用后面我们会讲）
    selectorFactory = defaultSelectorFactory  
    
    // ... 一些处理connect参数的工厂函数
} = {}) {
    // 这里才是connect函数
    return function connect({ 
        mapStateToProps,
        mapDispatchToProps,
        mergeProps,
        options={}
    }){ 
        // connect的参数有：mapStateToProps,mapDispatchToProps，mergeProps，options
        // 然而这些参数并不能直接使用，它们可能是对象，也可能是函数
        // 所以在这里进行通用性处理，是他们可以直接使用
        
        // connect返回了一个高阶组件（由connectHOC创建）
        return connectHOC(selectorFactory, { /*配置对象*/ })
    }
}

// 默认输出的connect
export default createConnect()
```

似乎connect并没有做**筛选和包装**这件事，仅仅返回了一个高阶组件，而这个高阶组件默认是由`connectAdvanced`创建的。

所以也就是说：

`connect`只是`connectAdvanced`（这个函数一会再看）的一个包装，`connect`本身只是一个预处理函数，真正的“筛选和包装”其实是在`connectAdvanced`这个函数里进行。

`connect`做的事情仅仅是：

将**选择器工厂**和**配置对象**传给`connectAdvanced`进行进一步处理。

（似乎“筛选和包装”的功能是通过选择器工厂和配置对象实现的，下面我们来看看是不是如此。）

**问题2**：

**“筛选和包装”的功能是如何实现的？**

***

#### connectAdvanced

从上面的代码可以看出，`connectAdvanced`的作用是：

根据`selectorFactory`和配置对象，创建一个高阶组件。

下面看看源码：

```javascript
function connectAdvanced(selectorFactory, { /*options配置对象*/ }) {
    // ... 根据options初始化一些内部变量
    
    //返回一个高阶组件（输入为一个组件，输出为另一个组件的函数）
    return function wrapWithConnect(WrappedComponent){
        // 高阶组件返回的组件Connect
        class Connect extends Component {
            constructor(props, context) {
                super(props, context)

                this.state = {}
                // 从props或context读取store
                //（因为WrappedComponent自己可能也是一个Provider）
                // 在本文的分析中我们忽略Provider嵌套的这种情况
                this.store = props[storeKey] || context[storeKey]
                // ...
            	this.initSelector() // 初始化一个selector（后面会讲）
            }
            
            // 初始化selector的函数
            initSelector() {/*...*/}
            
            // ... 其他一些生命周期函数和工具函数
        }
        Connect.contextTypes = contextTypes // 获取外层的context
        Connect.propTypes = contextTypes
                            
        // 返回Connect组件的时候多了一步处理
        // 这个函数的作用是将WrappedComponent上的静态方法拷贝到Connect上,并返回Connect
        // 这样就可以完全把Connect当作一个“WrappedComponent”使用
        return hoistNonReactStatics(Connect, WrappedComponent);
    }
}
```

结构依旧很简单，就是返回一个高阶组件。所谓高阶组件，其实是一个函数。

高阶组件接收被包裹的组件，返回一个Connect组件，下面我们的重点就放在这个Connect组件是如何创建的。



constructor看起来很普通，只不过从context中获取了store，然后将store存下来。还调用了一个`initSelector()`函数，初始化选择器？**选择器**是什么东西？？？别急我们一步一步来看。

看看其源码：

```javascript
initSelector() {
    // 传入dispatch方法和配置对象，得到一个原始选择器
    const sourceSelector = selectorFactory(this.store.dispatch, selectorFactoryOptions)
    // 对这个原始选择器进行进一步处理，得到一个最终的"stateful"的选择器selector
    this.selector = makeSelectorStateful(sourceSelector, this.store)
    // 调用selector的一个方法
    this.selector.run(this.props)
}
```

看完不免又有疑惑了：

- selectorFactory到底做了什么？
- makeSelectorStateful做了什么？添加的run方法是做什么的？

一个一个来看。

***

##### 1. selectorFactory----------------------------------

```javascript
// ...

function impureFinalPropsSelectorFactory({ /*配置对象*/ }){
    // ...
}

function pureFinalPropsSelectorFactory({ /*配置对象*/ }){
    // ...
}

export default function finalPropsSelectorFactory(dispatch, { /*配置对象*/ }) {
  // 从配置对象拿到initMapStateToProps，initMapDispatchToProps，initMergeProps
  // 这些方法是对connect函数参数（mapStateToProps，mapDispatchToProps，mergeProps）的包装
  // 所以调用后返回的是增强后，可以直接使用的同名函数
  const mapStateToProps = initMapStateToProps(dispatch, options)
  const mapDispatchToProps = initMapDispatchToProps(dispatch, options)
  const mergeProps = initMergeProps(dispatch, options)

  // 根据options的pure字段确定使用哪种工厂函数
  const selectorFactory = options.pure
    ? pureFinalPropsSelectorFactory
    : impureFinalPropsSelectorFactory

  // 使用选择器工厂，返回一个选择器
  return selectorFactory(
    mapStateToProps,
    mapDispatchToProps,
    mergeProps,
    dispatch,
    options //areStatesEqual, areOwnPropsEqual, areStatePropsEqual
  )
}
```

我们来看看两种工厂函数：

```javascript
// pure为false时的选择器工厂
// 功能及其简单，每次调用都返回一个新组装的props
function impureFinalPropsSelectorFactory(
  mapStateToProps,
  mapDispatchToProps,
  mergeProps,
  dispatch
) {
  return function impureFinalPropsSelector(state, ownProps) {
    return mergeProps(
      mapStateToProps(state, ownProps),
      mapDispatchToProps(dispatch, ownProps),
      ownProps
    )
  }
}

// pure为true时的选择器工厂
function pureFinalPropsSelectorFactory({
    mapStateToProps,
    mapDispatchToProps,
    mergeProps,
    dispatch,
    { areStatesEqual, areOwnPropsEqual, areStatePropsEqual }
}){
    let hasRunAtLeastOnce = false // 是否是第一次使用选择器
    let state // 这个state并不是组件的状态，而是redux的store
    let ownProps // 存储上一次传入的ownProps
    let stateProps // 通过mapStateToProps筛选出的要放进props的数据
    let dispatchProps // 通过mapDispatchToProps筛选出的要放进props的数据
    let mergedProps // 合并后的props，最终选择器返回的就是这个参数。
    
    // 第一次调用选择器的函数
    function handleFirstCall(firstState, firstOwnProps){
        state = firstState
        ownProps = firstOwnProps
        stateProps = mapStateToProps(state, ownProps)
        dispatchProps = mapDispatchToProps(dispatch, ownProps)
        mergedProps = mergeProps(stateProps, dispatchProps, ownProps)
        hasRunAtLeastOnce = true
        return mergedProps
    }
    
    // 处理不同情况时返回什么props
    // 这三个函数也没什么稀奇的骚操作，就不展开了
    // 仅仅是根据需要调用mapStateToProps或mapDispatchToProps得到stateProps和dispatchProps
    // 然后调用mergeProps得到mergedProps
    function handleNewPropsAndNewState(){}
    function handleNewProps(){}
    function handleNewState(){}
    
    function handleSubsequentCalls(nextState, nextOwnProps) {
        // areOwnPropsEqual,areStatesEqual用于比较新旧state，props是否相同
        // 这是从配置对象中拿到的方法，实现方式就只是一个浅比较
        const propsChanged = !areOwnPropsEqual(nextOwnProps, ownProps)
        const stateChanged = !areStatesEqual(nextState, state)
        state = nextState
        ownProps = nextOwnProps

        if (propsChanged && stateChanged) return handleNewPropsAndNewState()
        if (propsChanged) return handleNewProps()
        if (stateChanged) return handleNewState()
        return mergedProps
    }
    
    // 这里是返回的选择器
    return function pureFinalPropsSelector(nextState, nextOwnProps) {
        return hasRunAtLeastOnce // 判断是否是第一次使用选择器
        	? handleSubsequentCalls(nextState, nextOwnProps)
        	: handleFirstCall(nextState, nextOwnProps) 
    }
}
```

可以看出选择器工厂返回了一个函数`pureFinalPropsSelector`，这就一个选择器。

可以看出，选择器的功能，就是接收`nextState`和`nextOwnProps`，返回一个经过“筛选和包装”的props。返回的props可以直接传给被包裹的组件。

**回答问题2**（“筛选和包装”的功能是如何实现的？）：

**使用选择器工厂，根据我们传进去的配置项（经过处理的mapXXXToProps，dispatch，浅比较方法），生成一个具有“筛选和包装”功能的选择器**



在`connectAvanced`中使用的`selectorFactory`已经弄明白了，下面看看另一个`makeSelectorStateful`函数。

##### 2. makeSelectorStateful---------------------------------

```javascript
function makeSelectorStateful(sourceSelector, store) {
  // 创建了一个selector对象，这个对象有一个run方法
  const selector = {
    // run方法接收原始props（外部传给被包裹组件的props），
    // 并且调用了一次原始选择器，得到调用后的props，
    // 将新props和内部缓存的旧props比较，
    // 根据结果，设置selector的shouldComponentUpdate属性。
    run: function runComponentSelector(props) {
      try {
        const nextProps = sourceSelector(store.getState(), props)
        if (nextProps !== selector.props || selector.error) {
          selector.shouldComponentUpdate = true
          selector.props = nextProps
          selector.error = null
        }
      } catch (error) {
        selector.shouldComponentUpdate = true
        selector.error = error
      }
    }
  }

  return selector
}
```

***

现在再回到connectAdvanced的源码：

```javascript
function connectAdvanced(selectorFactory, { /*options配置对象*/ }) {
    //返回一个高阶组件（输入为一个组件，输出为另一个组件的函数）
    return function wrapWithConnect(WrappedComponent){
        // 高阶组件返回的组件Connect
        class Connect extends Component {
            constructor(props, context) {
                super(props, context)

                this.state = {}
                // 从props或context读取store
                this.store = props[storeKey] || context[storeKey]
            	this.initSelector() // 初始化一个选择器
            }
            
            initSelector() {/*...*/}
            
            // 我们现在要重点看这里！！！！！！！！
            // ... 其他一些生命周期函数和工具函数
        }
        // ...
        return hoistNonReactStatics(Connect, WrappedComponent);
    }
}
```

我们已经知道选择器的功能是：获取“筛选和包装”后的props，现在我们看看Connect组件是如何使用选择器的。将关注点放在Connect这个组件的生命周期函数是如何使用的：

```javascript
class Connect extends Component {
    constructor(props, context) {
        // ...
        this.initSelector()
    }
            
    
    componentDidMount() {
        // 向store中添加监听器，监听器函数在下面
        // 就不详细看这个函数的实现了，就是简单的调用store的subscribe方法
        this.subscription.trySubscribe() 
        
        // 运行选择器，根据选择器运行后的结果判断是否需要更新
        this.selector.run(this.props)
        if (this.selector.shouldComponentUpdate) this.forceUpdate()
    }
    // 当接收新的props时，运行选择器
    componentWillReceiveProps(nextProps) {
        this.selector.run(nextProps)
    }
    // 根据选择器运行后的结果判断是否需要更新
    shouldComponentUpdate() {
        return this.selector.shouldComponentUpdate
    }
    // 卸载组件时清理内存
    componentWillUnmount() {
        // 卸载监听器
        if (this.subscription) this.subscription.tryUnsubscribe()
        this.store = null
        this.selector.run = noop // 空函数function noop() {}
        this.selector.shouldComponentUpdate = false
    }
    
    // 监听器,将会用subscribe方法添加到store上，每当store被dispatch会被调用
    onStateChange() {
        this.selector.run(this.props)
    }
    
    render() {
        const selector = this.selector
        selector.shouldComponentUpdate = false

        if (selector.error) {
          throw selector.error
        } else {
          // addExtraProps方法的作用时将selector筛选后的props，添加到原本的props上。
          return createElement(WrappedComponent, this.addExtraProps(selector.props))
        }
   }
}
```

原来这么简单啊，就只是：

- 在需要的时候：

  - Connect组件第一次装载组件时
  - Connect组件接收props时
  - 监听store的监听器被触发时

  运行选择器，得到需要添加的额外的props

- 根据运行的结果确定是否更新Connect组件

- 渲染时向被包裹组件添加额外的props。

### 总结

如果你看到了最后，你会发现，react-redux的实现方式和我们文章开头的解决方案一毛一样：

- store存在父组件的context上
- 给子组件添加额外的props，以实现和store的交互

实现的亮点在于，react-redux用了高阶组件这种优雅的方式，将这种需求进行了封装。



如果有疑问，或者想要交流的地方，欢迎在评论区讨论。
