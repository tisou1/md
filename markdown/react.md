# React

## useEffect

[you-might-not-need-an-effect](https://beta.reactjs.org/learn/you-might-not-need-an-effect)

### You Might Not Need an Effect(你或许并不需要useEffect)

- You don’t need Effects to transform data for rendering.(不要在Effects中转换数据) 
  不要在`useEffect`中更新`state`,会引起重复渲染, 因为`state`更新后,`effect`就又会重新执行.
- You don’t need Effects to handle user events.(不要在Effects中处理用户事件)

### How to tell if a calculation is expensive? 如何评估一个计算是否是昂贵的

  通常,除非你要创建或者循环处理成千上万的对象,否则可能并不昂贵

  可以通过一下方式来计算花费的时间, 如果记录的时间大于1ms, 那么缓存就是有意义的

```js
console.time('filter array');
const visibleTodos = getFilteredTodos(todos, filter);
console.timeEnd('filter array');
```
**使用`useMemo`包裹测试**
**`useMemo`不会使第一次渲染变快,反而会变慢. 他只能帮助你跳过更新时不必要的工作**

```js
console.time('filter array');
const visibleTodos = useMemo(() => {
  return getFilteredTodos(todos, filter); // Skipped if todos and filter haven't changed
}, [todos, filter]);
console.timeEnd('filter array');
```

### 当props改变是,子组件改变`state`
```jsx

// React会第一次渲染使用一个旧值,然后再发现userId改变后,重置了comment,再次渲染
  export default function ProfilePage({ userId }) {
  const [comment, setComment] = useState('');

  // 🔴 Avoid: Resetting state on prop change in an Effect
  useEffect(() => {
    setComment('');
  }, [userId]);
  // ...
}
```

```jsx
// 这种,拆分,然后传递一个key,当userId更改后, react会直接将原来的Profile卸载,然后渲染新的Profile
export default function ProfilePage({ userId }) {
  return (
    <Profile
      userId={userId}
      key={userId}
    />
  );
}

function Profile({ userId }) {
  // ✅ This and any other state below will reset on key change automatically
  const [comment, setComment] = useState('');
  // ...
}
```


```jsx
function List({ items }) {
  const [isReverse, setIsReverse] = useState(false);
  const [selection, setSelection] = useState(null);

  // 🔴 Avoid: Adjusting state on prop change in an Effect
  useEffect(() => {
    setSelection(null);
  }, [items]);
  // ...
}

// 在渲染阶段调用`setSelection`, react以返回语句退出后之后立即重新渲染List,到那时,React还没有渲染List的子节点或更新DOm,所以折让List的子节点跳过渲染陈旧的值
// TODO
// Storing information from previous renders like this can be hard to understand, but it’s better than updating the same state in an Effect. In the above example, setSelection is called directly during a render. React will re-render the List immediately after it exits with a return statement. By that point, React hasn’t rendered the List children or updated the DOM yet, so this lets the List children skip rendering the stale selection value.
function List({ items }) {
  const [isReverse, setIsReverse] = useState(false);
  const [selection, setSelection] = useState(null);

  // Better: Adjust the state while rendering
  const [prevItems, setPrevItems] = useState(items);
  if (items !== prevItems) {
    setPrevItems(items);
    setSelection(null);
  }
  // ...
}

**When you update a component during rendering, React throws away the returned JSX and immediately retries rendering.**
当你在渲染过程中更新一个组件时，React会扔掉返回的JSX并立即重试渲染。

** To avoid very slow cascading retries, React only lets you update the same component’s state during a render. If you update another component’s state during a render, you’ll see an error.**
为了避免非常缓慢的级联重试，React只允许你在渲染期间更新同一个组件的状态。如果更新另一个组件的`state`在渲染阶段,则会报错.

// 上面的在渲染阶段进行setState的做法,不是很推荐, 所以我们这里用这种方式来表示selection
function List({ items }) {
  const [isReverse, setIsReverse] = useState(false);
  const [selectedId, setSelectedId] = useState(null);
  // ✅ Best: Calculate everything during rendering
  const selection = items.find(item => item.id === selectedId) ?? null;
  // ...
}
```

![](/react/2023-02-23-14-30-17.png)