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

最好不要使用链式调用`Effects`来更新`state`  
这样会导致最坏情况下(setCard → render → setGoldCardCount → render → setRound → render → setIsGameOver → render) 
这回产生不必要的渲染.

```js
  function Game() {
  const [card, setCard] = useState(null);
  const [goldCardCount, setGoldCardCount] = useState(0);
  const [round, setRound] = useState(1);
  const [isGameOver, setIsGameOver] = useState(false);

  // 🔴 Avoid: Chains of Effects that adjust the state solely to trigger each other
  useEffect(() => {
    if (card !== null && card.gold) {
      setGoldCardCount(c => c + 1);
    }
  }, [card]);

  useEffect(() => {
    if (goldCardCount > 3) {
      setRound(r => r + 1)
      setGoldCardCount(0);
    }
  }, [goldCardCount]);

  useEffect(() => {
    if (round > 5) {
      setIsGameOver(true);
    }
  }, [round]);

  useEffect(() => {
    alert('Good game!');
  }, [isGameOver]);

  function handlePlaceCard(nextCard) {
    if (isGameOver) {
      throw Error('Game already ended.');
    } else {
      setCard(nextCard);
    }
  }

  // ...
```

### 获取数据


通常我们会在`useEffects`中加入类似下面的代码

```js
  function SearchResults({ query }) {
  const [results, setResults] = useState([]);
  const [page, setPage] = useState(1);

  useEffect(() => {
    // 🔴 Avoid: Fetching without cleanup logic
    fetchResults(query, page).then(json => {
      setResults(json);
    });
  }, [query, page]);

  function handleNextPageClick() {
    setPage(page + 1);
  }
  // ...
}
```

这里通常回头`竟态`问题,当我们在输入框输入`query`的值的时候,比如我们输入了`hello`,每次输入都会触发`query`的改变,那么`Effect`就会重复调用几次,每次的`query`值为`h`->`he`->`hel`->`hell`->`hello`.但是我们不能保证每次的异步请求返回顺序也是正确的. 有可能`hel`那次请求最后一次返回,这时`results`中拿到的值就不是最新的.

为了解决这个问题,我们可以作出如下改造  
这可以保证除了最后一次请求意外,所有的响应都会被忽略.
```js
  useEffect(() => {
    let ignore = false;
    fetchResults(query, page).then(json => {
      if(!ignore) {
         setResults(json);
      }
    });

    return () => {
      // 在组件卸载的时候,如果将ignore设为true,这样即使组件卸载之后,上面的回调中也不会赋值
      ignore = true
    }
  }, [query, page]);

}
```

**这里还有一个点,就是如何缓存数据**当用户点击返回时,如何在不显示`spinner`的同时展示先前的数据.  
缓存的问题, 如何在第一次渲染的时候不展示`spinner`直接显示数据(服务端渲染), 这些问题都不好解决.

如果不借助库的话,我们可以用编写一个自定义`hook`

```js
function useData(url) {
  const [data, setData] = useState(null);

  useEffect(() => {

    let ignore = false;

    fetch(url)
      .then(res => res.json())
      .then(json => {
        if (!ignore) {
          setData(json)
          }
      }) 
  }, [url])

  return () =>{
    ignore = true
  }

  return data``
}
```

一般来说当你不得不使用`Effect`时,注意能不能将某项功能抽取到一个自定义hook中.  
组件中原始`useEffect`使用越少,代码维护越容易.

**以下时不适用Effect的情况**
- 如果在渲染阶段计算一些值
- 缓存额外的计算, 可以使用`useMemo`来实现
- 重置组件树状态,可以通过传递key值来实现
- 重置特定的state,在响应props改变时, 应该在渲染阶段来做
- 当代码运行发生在响应用户行为时,放在事件回调中, 只有当组件显示才需要运行的代码放在Effects中
- 如果更新几个组件的状态,最好在同一个事件中完成
- 如果您需要同步不同组件的状态时,考虑状态提升,用一个变量来控制
- 可以再Effects中获取数据,但是应该写清楚函数



## react Effects的生命周期

与组件的生命周期不同, 一个Effect只能做两件事
- 开始同步(synchronize)某些东西 (Effects)内容体
- 然后停止同步(stop  synchronize) (清除函数)

组件的生命周期
- 挂载(mount) - 当组件被添加到页面时
- 更新(update) - 当组件的state或者props发生改变时
- 卸载(unmount) - 当组件从页面上移除时

![](/images/react/2023-02-25-17-48-44.png)

对比组件对的行为和`Effects`的行为, 我们可以发现他们还是有一定的心智区别的  
所以后面不要将`Effect`用组件的生命周期来解释其执行  

> 组件渲染之后,Effect的开始同步和停止同步...[]

每次只专注与一个单一的**启动() 停止**周期.一个组件是在挂载,更新,还是卸载都不是很重要对于`Effect`来说

不要把`Effects`当做组件的生命周期回调函数来看待.不然会很容易迷惑的



**Each Effect in your code should represent a separate and independent synchronization process.**


### Effect的依赖项为空空数组时

如果以组件的角度来看的话, 就是`Effect`只有在组件挂载时才会链接到聊天室,只有在组件卸载时才会断开连接(clear up)  


[Effect的行为](https://beta.reactjs.org/learn/lifecycle-of-reactive-effects#thinking-from-the-effects-perspective)
如果以Effect的角度来看, 不需要考虑组件挂载和卸载问题,重要的是,你指定了`Effect`开始和停止同步的做法.


## 所有在组件内部的变量都是响应性的

`props`和`state`并不是唯一的响应性值, 用他们计算出来的值也是有响应性的.
`props`和`state`发生变化时,组件重新渲染,由他们计算出的值也将发生变化.也就是说也会放到`Effects`的依赖项当中


外部变量值和useRef()的返回值,也不能当做`Effect`的依赖项,因为当他们不是一个响应性的变量,即使他们发生变化,React也不会重新执行`Effect`