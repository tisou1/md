

### vue3 的响应式

草稿版本的响应式

```js
const targetMap = new WeakMap()
// 保存当前的的effect回调, 解决放入依赖数组时的时候的,匿名函数问题
let activeEffect

function trackEffects(dep) {
  if (!dep.has(activeEffect))
    dep.add(activeEffect)
}

function triggerEffects(dep) {
  for (const effect of dep)
    effect()
}

// 关键函数 -- 收集依赖
function track(target, type, propsKey) {
  console.log(`触发 track -> target: ${target} type:${type} key:${propsKey}`)

  let depsMaps = targetMap.get(target)
  if (!depsMaps) {
    depsMaps = new Map()
    targetMap.set(target, depsMaps)
  }

  let dep = depsMaps.get(propsKey)
  if (!dep) {
    dep = new Set()
    depsMaps.set(propsKey, dep)
  }

  trackEffects(dep)
}

// 关键函数 -- 触发回调
function trigger(target, type, propsKey) {
  console.log('trigger', target, type, propsKey)

  const deps = []
  const depsMap = targetMap.get(target)
  if (!depsMap)
    return

  const dep = depsMap.get(propsKey)
  deps.push(dep)

  const effects = []
  deps.forEach((dep) => {
    // 这里解构 dep 得到的是 dep 内部存储的 effect
    effects.push(...dep)
  })

  triggerEffects(new Set(effects))
}

const handler = {
  get(target, propsKey, receiver) {
    const res = Reflect.get(target, propsKey, receiver)

    // 在触发 get 的时候进行依赖收集
    track(target, 'get', propsKey)

    if (isObject(res)) {
        // 将所有是object的属性也进行包裹,变成响应式
        return createReactiveObject(res, handler)
    }

    return res
  },
  set(target, propsKey, value, receiver) {
    const res = Reflect.set(target, propsKey, value, receiver)

    // 触发set的时候进行依赖响应, 比如更新数据,更新试图...
    trigger(target, 'set', propsKey)

    return res
  },
}

function createReactiveObject(target, handler) {
  const proxy = new Proxy(target, handler)

  return proxy
}

function effect(fn) {
  activeEffect = fn
  // 执行回调函数
  fn()
}

function reactive(target) {
  return createReactiveObject(target, handler)
}
```

用法
```js 
  let obj = {
    name: 'siry',
    age: 18
  }

  let newProxy = reactive(obj)

  effect(() => {
    console.log(newProxy.name)
  })

  effect(() => {
    console.log(newProxy.age)
  })

  setTimeout(() => {
    newProxy.name = 'ti'
  },1000)
   setTimeout(() => {
    newProxy.age = 19
  },2000)

  
```