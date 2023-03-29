

#### 1. 在useEffect中更新state, 经常会碰到的一个问题是,子组件更新state会依赖props或者store等数据

如果`state`只是在组件初次渲染的时候,可以给useEffect的依赖项空数组  
但是如果这个`state`需要在某些值(props, store, 自定义hook返回数据)改变时也需要更新
