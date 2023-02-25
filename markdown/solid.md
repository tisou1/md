# solid

> solid函数组件,应该被当做constrctor来看待,而不是像react当中当做render函数看待

**一个React计时器**,每次count改变,Counter组件(函数)都会重新执行

```jsx
function Counter() {
  const [count, setCount] = createSignal(0);
  console.log('Counter组件')
  setInterval(() => {
    setCount(count() + 1);
  }, 1000);
  console.log('The Counter function was called!');
  return <div>The count is: {count()}</div>;
}
```

**这是一个solid计时器**,当count改变是,Counter组件(函数)不会重新执行了

```jsx
function Counter() {
  const [count, setCount] = createSignal(0);
  setInterval(() => {
    setCount(count() + 1);
  }, 1000);
  console.log(`The count is ${count()}`);
  return <div>The count is: {count()}</div>;
}
```