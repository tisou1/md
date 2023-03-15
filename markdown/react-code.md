

#### 1. React如何对比更新?

- 只对同级节点更新,如果DOM节点跨越层级了,则不会复用
- 不同类型的元素产出不同结构
- 可以通过key来复用原有的dom
    当key和type相同时,会复用
- 类型一致的节点才有继续diff的必要性



#### 2. React的类组件和函数组件是如何保存state和props等信息的

`react 16.8.0`之后,函数组件和类组件都会被转换为`fiber`节点,`fiber`节点上会保存组件的`state`,`propss`等信息.  
不同的是,类组件在实例化之后,实例上也会保存组件的信息,当我们调用`this.setState()`时,会将新的状态存放在实例上,并在`fiber`上记录组件的更新操作,在下一次更新时，React会使用新的state值创建一个新的Fiber节点，并将其与旧的Fiber节点进行比较，从而计算出哪些部分需要更新，哪些部分需要重新渲染。

**由于Class组件的实例和Fiber节点都保存了组件的状态和props等信息，因此在组件更新时，React需要进行额外的同步和协调操作，以确保组件的状态和props值的一致性。**

React会在Function Component首次调用时，创建一个对应的Fiber节点，并在该节点中保存Function Component的props和其他信息。当组件被重新渲染时，React会重用这个Fiber节点，并将更新后的props传递给Function Component。  

而对于Function Component中的状态，React通过Hooks来管理。Hooks是一些特殊的函数，用于管理Function Component中的状态、副作用和其他功能
`fiber`节点的`memoizedState`属性在函数组件中,会指向`Hook`队列(`Hook`队列保存了`function`类型的组件状态).

每次Function Component被调用时，React会执行对应的Hooks，来获取或更新Function Component中的状态。Hooks中的状态值是保存在与Function Component对应的Fiber节点中的，而不是保存在组件实例上的。这样，在Function Component的多次渲染期间，每个Hook的状态都会得到正确的更新和保存。

**虽然Function Component中的状态和props都不是保存在组件实例上的，但我们可以通过某些手段来模拟类似于类组件的实例变量和实例方法.**



#### 3. 