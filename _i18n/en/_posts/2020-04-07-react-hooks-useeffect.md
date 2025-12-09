---
layout: post
title: Solving the useEffect repeat call problem
date: 2020-04-07 11:23:19
tags: [React]
categories:
 - web-framework
---

`useEffect` is the [Effect Hook](https://zh-hans.reactjs.org/docs/hooks-effect.html) of the React hooks that lets you perform side effect operations in function components.

I also documented a piece on [recognizing react hooks](https://seminelee.github.io/2019/03/04/react-hooks/) when React hooks first came out. In the process of using them, I often encountered the problem of `useEffect` repeated calls, so I borrowed this article to summarize it.

<!-- more -->

# 1 Why is there a problem with duplicate requests?

To summarize the reasons could be:

## 1.1 You didn't set the effect dependency parameter

For example, in the example below, it executes after the first render and after each update.

``` js
const [count, setCount] = useState(0)

useEffect(() => {
  document.title = `You clicked ${count} times`;
})
```

This is because each re-rendering has its own Props and State, and each function within the component (including event handlers, effects, timers or API calls, etc.) captures the Props and State defined in a particular rendering. in a sense, the effect is more like a part of the rendering result --__Each effect "belongs" to a particular rendering__.

In fact this is exactly why we can get the latest value of `count` in effect without worrying about it expiring.
If you don't have an effect dependency parameter set, just set the dependency in the second parameter of `useEffect`.

## 1.2 The dependencies you set up change frequently

Sometimes we have set up a dependency but find that it still repeats indefinitely. It's possible that your dependencies just change frequently, i.e. they use state in methods that change state, for example:

``` js
function Counter() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    const id = setInterval(() => {
      setCount(count + 1); // 这个 effect 依赖于 `count` state
    }, 1000);
    return () => clearInterval(id);
  }, [count]);

  return <h1>{count}</h1>;
}
```

To solve this problem, we can use the [functional update](https://zh-hans.reactjs.org/docs/hooks-reference.html#functional-updates) form of `setState`. This allows us to specify how the state should change without referencing the current state:

``` js
function Counter() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    const id = setInterval(() => {
      setCount(c => c + 1); // ✅ 在这不依赖于外部的 `count` 变量
    }, 1000);
    return () => clearInterval(id);
  }, []); // ✅ 我们的 effect 不适用组件作用域中的任何变量

  return <h1>{count}</h1>;
}
```

For details, please see [FAQ](https://zh-hans.reactjs.org/docs/hooks-faq.html#what-can-i-do-if-my-effect-dependencies-change-too-often) on the official website.

## 1.3 Dependencies set are referenced data types

This actually falls under the second reason, if we set a dependency that references a data type, we'll find that the set dependency will always change.

For example, if you open the console in the example below, you will see the output at least 2 times. As mentioned above, the function component has its own Props and State each time it is re-rendered, so React will conclude that the dependency is not the same each time when comparing. Even though it looks like the content is the same, the reference address is different each time, i.e. `[] ! == []`.

``` js
const [data, setData] = useState([] as any)
useEffect(() => {
  setTimeout(() => {
    setData([])
  }, 100)
}, [])

useEffect(() => {
  setTimeout(() => {
    console.log(data)
  }, 200);
}, [data])
```

The solution to the third reason, which we will explore in detail next.

> **Don't lie to React about the dependencies**
> If you set up dependencies, all values within components used in effect should be included in the dependency. This includes props, states, functions - anything within a component. The solution to the problem is not to remove the dependency. Only if the dependency contains all the values used in the effect will React know when it needs to run it.

# 2 Functions as dependencies

## 2.1 Check if the function must be used as a dependency

It is generally recommended to mention functions that don't depend on props and state outside your component, and to put functions that are only used by effects inside effects.

``` js
// ✅ Not affected by the data flow
function getFetchUrl(query) {
  return 'https://hn.algolia.com/api/v1/search?query=' + query;
}

function SearchResults() {
  useEffect(() => {
    const url = getFetchUrl('react');
    // ... Fetch data and do something ...
  }, []); // ✅ Deps are OK
  // ...
}
```

## 2.2 useCallback

If it turns out that your effect does need to use functions within the component (including functions passed in via props), you can wrap them where they are defined with [`useCallback`](https://zh-hans.reactjs.org/docs/hooks-reference.html# usecallback) to wrap a layer. Why? Because these functions have access to props and state, so they participate in the data flow.

`useCallback` essentially adds a layer of dependency checking. It solves the problem in a different way - we make the function itself change only when needed, rather than removing dependencies on the function.

``` js
function SearchResults() {
  const [query, setQuery] = useState('react');

  // ✅ Preserves identity until query changes
  const getFetchUrl = useCallback(() => {
    return 'https://hn.algolia.com/api/v1/search?query=' + query;
  }, [query]);  // ✅ Callback deps are OK

  useEffect(() => {
    const url = getFetchUrl();
    // ... Fetch data and do something ...
  }, [getFetchUrl]); // ✅ Effect deps are OK

  // ...
}
```

In the case of a function passed in by props, `getFetchUrl` in the example above could be written as follows. the function passed in by props has access to both props and state. wrap its definition into the useCallback Hook. this ensures that it doesn't change with rendering unless its own dependencies have changed.

``` js
const getFetchUrl = useCallback(props.fetchData, [query])
```

[``useMemo``](https://zh-hans.reactjs.org/docs/hooks-reference.html#usememo) can do something similar to avoid non-essential rendering. `useCallback(fn, deps)` is equivalent to `useMemo(() => fn, deps)`. I won't describe it here.

# 3 Objects as dependencies

## 3.1 Checking if an object must be used as a dependency

The first thing you can check is whether you have to use that object as a dependency, for example:

* Only a property of a non-reference type of this object is needed;
* is a JSON object, which can be passed in as a string with `JSON.stringify()`. The subcomponent then parses the JSON string passed in by props with `JSON.parse()`.

## 3.2 useRef

If none of the above works, hopefully `useRef` will solve your problem.

So far, we know that every function within a component (including event handlers, effects, timers or API calls, etc.) captures the props and states defined in a particular rendering, so the key to solving the problem lies in the fact that the most recent values are read in the effect's callback function instead of the captured ones, i.e., the future props and states are read from the function in the past rendering. The guide visualizes this as [moving against the tide](https://overreacted.io/zh-hans/a-complete-guide-to-useeffect/#%E9%80%86%E6%BD%AE%E8%80%8C%E5%8A%A8) The guide compares this to [moving against the tide] ().

`useRef` does just that. Unlike effect, which captures the props and state defined in a particular rendering, `useRef`'s `.current` property acts as a `box' that holds a variable value, getting the most recent value. And `useRef` doesn't notify you when the contents of the ref object change. Changing the `.current` property does not trigger a re-rendering of the component.

The example in 1.3 can be rewritten like this. Open the console and you can see that only the latest value `[]` is output.

``` js
const [data, setData] = useState([] as any)
const dataRef = useRef(data)
useEffect(() => {
  setTimeout(() => {
    setData([])
  }, 100);
}, [])

useEffect(() => {
  dataRef.current = data
})

useEffect(() => {
  setTimeout(() => {
    console.log(dataRef.current)
  }, 200);
}, [])
```

# Reference

* [Official React documentation](https://zh-hans.reactjs.org/docs/getting-started.html)
* [useEffect Complete Guide](https://overreacted.io/zh-hans/a-complete-guide-to-useeffect/)
