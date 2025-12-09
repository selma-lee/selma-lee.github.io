---
layout: post
title: Recognizing react Hooks
date: 2019-03-04 11:23:19
tags: [React]
categories:
 - web-framework
---

Hooks are a new feature in React v16.7.0-alpha. Hooks are special functions that allow you to "hook" into React features, allowing you to use states and other React features outside of the class.
To summarize, there are three main types of Hooks, each of which brings us new features:

<!-- more -->

The # state Hook lets you use state in function components.

Classes have a learning cost, you have to understand how this works in JavaScript, and classes also bring a lot of issues to today's tools. for example, classes can't be minified very well.
The state Hook lets you avoid these problems by using state in function components.

For those of you who have used React, you may know what a "stateless component" looks like:

``` js
const Example = (props) => {
  // You can use Hooks here!
  return <div />;
}
```

Previously, we called it a "stateless component" mainly because we couldn't use state like in a class.
The state Hook allows you to use state and other React features outside of the class, so now you can call it a function component instead of a "stateless component". The code is as follows:

``` js
import React, { useState } from 'react';

function Example() {
  // 声明一个新的状态变量count
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
    </div>
  );
}
```

In this code, we introduce the `useState` Hook. first we initialize the state to `{ count: 0 }`, and then when the user clicks on the callback function containing `this.setState()`, we count and change the state.
** What's the advantage of writing it this way? ** This way, we don't have to write a class, and we avoid all the problems that come with classes.

`useState` is one kind of Hook, let's move on to other Hooks.

The # Effect Hook lets you split a component into smaller functions based on related parts

However, as development progresses components become larger and more confusing, with each lifecycle hook containing a bunch of unrelated logic. The end result is that strongly related code is separated, and unrelated code is put together instead. This obviously leads to a lot of errors.
Effect Hook lets you avoid these problems by splitting a component into smaller functions based on related parts, rather than based on the lifecycle.

## Call effect on mount and update

In many cases, we want to perform the same side effect regardless of whether the component was just mounted or just updated. Conceptually, we want it to be executed after every render.
We'll reintroduce the `useEffect` Hook in the example we just showed, as follows:

``` js
import { useState, useEffect } from 'react';

function Example() {
  const [count, setCount] = useState(0);

  // 与 componentDidMount 和 componentDidUpdate 类似:
  useEffect(() => {
    // 通过浏览器自带的 API 更新页面标题
    document.title = `You clicked ${count} times`;
  });

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
    </div>
  );
}
```

In this code, every time `count` is updated, we update the page title via `document.title`. The code in `useEffect` will run after the first render and every update thereafter. It's equivalent to writing in `componentDidMount`, `componentDidUpdate`.
** So what are the benefits of using it? ** Let's think about this: if we were to write the above code in a class, we would need to repeat this code in two lifecycles, as follows:

``` js
class Example extends React.Component {
  // ...
  componentDidMount() {
    document.title = `You clicked ${this.state.count} times`;
  }

  componentDidUpdate() {
    document.title = `You clicked ${this.state.count} times`;
  }
  // ...
}
```

With `useEffect`, React records the method you pass to `useEffect` and calls it after the DOM update, which avoids duplicating code.

## Clean up effect when uninstalling

In the React class, it's typical to create a subscription in `componentDidMount` and then clear it in `componentWillUnmount`.
So what do you do with an Effect Hook? Just return the function that performs the cleanup in the `useEffect` method. Look at the example below:

``` js
import { useState, useEffect } from 'react';

function FriendStatus(props) {
  const [isOnline, setIsOnline] = useState(null);

  function handleStatusChange(status) {
    setIsOnline(status.isOnline);
  }

  useEffect(() => {
    ChatAPI.subscribeToFriendStatus(props.friend.id, handleStatusChange);
    // 明确在这个 effect 之后如何清理它
    return function cleanup() {
      ChatAPI.unsubscribeFromFriendStatus(props.friend.id, handleStatusChange);
    };
  });

  if (isOnline === null) {
    return 'Loading...';
  }
  return isOnline ? 'Online' : 'Offline';
}
```

** And what are the benefits of using it? ** Obviously, it allows us to keep the logic of adding and removing subscriptions close to each other. They are part of the same effect!
** And when does React clean up the effect? ** React performs cleanup every time a component unmounts. However, as we learned earlier, the effect is run every time it's rendered, not just once. That's why React also cleans up the effect from the previous render after the next time the effect is run.

The advantage of Effect Hook is more obvious when there are more than one Effect. You can see the example below:

``` js
function FriendStatusWithCounter(props) {
  // 关于计数器的state与effect
  const [count, setCount] = useState(0);
  useEffect(() => {
    document.title = `You clicked ${count} times`;
  });
  // 关于是否在线的state与effect
  const [isOnline, setIsOnline] = useState(null);
  useEffect(() => {
    ChatAPI.subscribeToFriendStatus(props.friend.id, handleStatusChange);
    return () => {
      ChatAPI.unsubscribeFromFriendStatus(props.friend.id, handleStatusChange);
    };
  });

  function handleStatusChange(status) {
    setIsOnline(status.isOnline);
  }
  // ...
}
```

Personally, I think it does make it clearer than splitting the logic into `componentDidMount` and `componentWillUnmount` in the class.

# Custom Hooks let you reuse stateful logic across components.

Previously, reusing stateful logic across components was difficult. Both [render props](https://react.docschina.org/docs/render-props.html) and [high-level components](https://react.docschina.org/docs/higher-order-components. (. html) both require you to reconstruct your components, which can be tricky.
Custom Hooks are used to solve this problem.
Let's change `FriendStatus` in the above example to a component that can be reused, `useFriendStatus`, and two components that reuse the stateful logic of this component, `FriendStatus` and `FriendListItem`, as follows:

``` js
import { useState, useEffect } from 'react';

function useFriendStatus(friendID) {
  const [isOnline, setIsOnline] = useState(null);

  function handleStatusChange(status) {
    setIsOnline(status.isOnline);
  }

  useEffect(() => {
    ChatAPI.subscribeToFriendStatus(friendID, handleStatusChange);
    return () => {
      ChatAPI.unsubscribeFromFriendStatus(friendID, handleStatusChange);
    };
  });

  return isOnline;
}
```

``` js
function FriendStatus(props) {
  const isOnline = useFriendStatus(props.friend.id);

  if (isOnline === null) {
    return 'Loading...';
  }
  return isOnline ? 'Online' : 'Offline';
}
```

``` js
function FriendListItem(props) {
  const isOnline = useFriendStatus(props.friend.id);
  const liStyle = { color: isOnline ? 'green' : 'black' };

  return (
    <li style={liStyle}>
      {props.friend.name}
    </li>
  );
}
```

** Do two components share state using the same Hook? ** No. A custom Hook is a mechanism for reusing stateful logic (e.g., setting up subscriptions and remembering the current value), but every time a custom Hook is used, all of its internal state and effects are completely isolated.
** How does a custom Hook get isolated state? ** Because we're calling `useFriendStatus` directly, from a React perspective, our component only calls `useState` and `useEffect`. As we learned earlier, we could call `useState` and `useEffect` multiple times in a component and they would be completely independent.

# More

Here's a brief summary of a few of the new features of React Hook, for more details you can see the official website

* [Introducing Hooks](https://react.docschina.org/docs/hooks-intro.html)
* [Using the State Hook](https://react.docschina.org/docs/hooks-state.html)
* [Using the Effect Hook](https://react.docschina.org/docs/hooks-effect.html)
* [Building Your Own Hooks](https://react.docschina.org/docs/hooks-custom.html)
