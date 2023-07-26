

#### 1. 在useEffect中更新state, 经常会碰到的一个问题是,子组件更新state会依赖props或者store等数据

如果`state`只是在组件初次渲染的时候,可以给useEffect的依赖项空数组  
但是如果这个`state`需要在某些值(props, store, 自定义hook返回数据)改变时也需要更新


#### 2. redux redux-thunk

https://segmentfault.com/a/1190000013998403
https://redux.js.org/understanding/history-and-design/middleware#understanding-middleware

```js

import {legacy_createStore as createStore, applyMiddleware} from 'redux'
// import thunk from 'redux-thunk'
// 支持异步 ,使用thunk


function thunk(store) {
  return function (next) {
    // 其实就是redux的 这里传入的store.dispatch
    // https://github.com/reduxjs/redux/blob/master/src/applyMiddleware.ts#L70
    return function (action) {
      // 从store中解构出dispatch, getState
      const { dispatch, getState } = store;

      // 如果action是函数，将它拿出来运行，参数就是dispatch和getState
      console.log('...', action)
      // 当action为函数时, 汇之星action,action又有dispatch发起, 所以又会从新走一遍thunk,这时候的action为plain text 所以会下面的next
      if (typeof action === 'function') {
        return action(dispatch, getState);
      }

      // 否则按照普通action处理
      let result = next(action);
      return result
    }
  }
}

function logger1(store) {
  return function(next) {
    // 这里的next其实就是指向的loggger2返回的函数
    return function(action) {
      console.group(action.type);
      console.info('logger1 dispatching', action);
      let result = next(action);
      console.log('logger1 next state', store.getState());
      console.groupEnd();
      return result
    }
  }
}

function logger2(store) {
  return function(next) {
    // 这里的next,指向thunk
    return function(action) {
      console.group(action.type);
      console.info('logger2 dispatching', action);
      let result = next(action);
      console.log('logger2 next state', store.getState());
      console.groupEnd();
      return result
    }
  }
}


const store = createStore(counterReducer, applyMiddleware(logger1, logger2,thunk))

function counterReducer(state = 0, action) {
  switch(action.type){
    case 'add':
      return ++state
    case 'sub':
        return --state
    default:
      return state
  }
}

export default store

```


```jsx
import logo from './logo.svg';
import './App.css';

import { useDispatch, useSelector } from 'react-redux'

function App() {
  const dispatch = useDispatch()
  const count = useSelector(state => state)
  const handleAdd = () => {
    dispatch({
      type: 'add'
    })
  }

  const handleSub = () => {
    dispatch({
      type: 'sub'
    })
  }

  const handleAddAsync = () => {
      console.log('异步增加')
      setTimeout(() => {
        dispatch({type: 'add'})
      },2000)
  }

  const handleAddWithThunk = () => {
    function incrementAsync() {
      return (dispatch) => {
        // setTimeout(() => {
          dispatch({type: 'add'});
        // }, 1000);
      }
    }

    dispatch(incrementAsync())
    
  }

  const handleAddAsyncWithThunk = () => {
    function incrementAsync() {
      return (dispatch) => {
          Promise.resolve().then(() => {
            dispatch({type: 'add'});
          })
      }
    }
    dispatch(incrementAsync())
  }
  return (
    <div className="App">
      <header className="App-header">
        <img src={logo} className="App-logo" alt="logo" />
        <p>
          Edit <code>src/App.js</code> and save to reload.
        </p>

        <button onClick={handleAdd}>+</button>
        <button onClick={handleSub}>-</button>
        <button onClick={handleAddAsync}>不使用thunk异步增加</button>
        <button onClick={handleAddWithThunk}>使用thunk同步增加</button>
        <button onClick={handleAddAsyncWithThunk}>使用thunk异步增加</button>
        <span>{count}</span>
        <a
          className="App-link"
          href="https://reactjs.org"
          target="_blank"
          rel="noopener noreferrer"
        >
          Learn React
        </a>
      </header>
    </div>
  );
}

export default App;


```