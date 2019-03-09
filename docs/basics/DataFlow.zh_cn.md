---
id: data-flow
title: Data flow
sidebar_label: Data flow
hide_title: true
---

# Data Flow

Redux architecture revolves around a **strict unidirectional data flow**.

Redux 架构始终围绕着 **严格单向数据流** 的做法。

This means that all data in an application follows the same lifecycle pattern, making the logic of your app more predictable and easier to understand. It also encourages data normalization, so that you don't end up with multiple, independent copies of the same data that are unaware of one another.

这意味着应用中的所有数据遵循相同的生命周期模式，使得应用程序的逻辑具有可预测性和更易于理解。同时它还鼓励数据规范化，这样你就不会在不知情的情况下得到多个相同数据的独立副本。

If you're still not convinced, read [Motivation](../introduction/Motivation.md) and [The Case for Flux](https://medium.com/@dan_abramov/the-case-for-flux-379b7d1982c6) for a compelling argument in favor of unidirectional data flow. Although [Redux is not exactly Flux](../introduction/PriorArt.md), it shares the same key benefits.

如果你仍然不信，请查看关于 [Motivation](../introduction/Motivation.md) 和 [The Case for Flux](https://medium.com/@dan_abramov/the-case-for-flux-379b7d1982c6) 部分以获得支持单向数据流的令人信服的论据。

The data lifecycle in any Redux app follows these 4 steps:

在任何 Redux 应用这数据生命周期遵循以下四步：

1. **You call** [`store.dispatch(action)`](../api/Store.md#dispatch).

**你开始调用**[`store.dispatch(action)`](../api/Store.md#dispatch)

An [action](Actions.md) is a plain object describing _what happened_. For example:

一个 [action](Actions.md) 就是一个描述 _发生了什么_ 的纯对象。如：

```js
 { type: 'LIKE_ARTICLE', articleId: 42 }
 { type: 'FETCH_USER_SUCCESS', response: { id: 3, name: 'Mary' } }
 { type: 'ADD_TODO', text: 'Read the Redux docs.' }
```

Think of an action as a very brief snippet of news. “Mary liked article 42.” or “'Read the Redux docs.' was added to the list of todos.”

把一个 action 想象成一个非常简洁的新闻片段。“Mary 喜欢第 42 篇文章。” 或者 “'阅读 Redux 文档。' 已经被添加到待办事项列表中。”

You can call [`store.dispatch(action)`](../api/Store.md#dispatch) from anywhere in your app, including components and XHR callbacks, or even at scheduled intervals.

你可以在应用的任何位置调用 [`store.dispatch(action)`](../api/Store.md#dispatch)，包括组件和 XHR 回调甚至是在任务间隔中。

2. **The Redux store calls the reducer function you gave it.**

**Redux store 调用你给的 reducer 函数。**

The [store](Store.md) will pass two arguments to the [reducer](Reducers.md): the current state tree and the action. For example, in the todo app, the root reducer might receive something like this:

[store](Store.md) 会传递两个参数到 [reducer](Reducers.md) 中：当前 state 树和 action。例如，在待办事项应用中，根 reducer 接收到的数据可能像下面这样：

```js
// The current application state (list of todos and chosen filter)
let previousState = {
  visibleTodoFilter: 'SHOW_ALL',
  todos: [
    {
      text: 'Read the docs.',
      complete: false
    }
  ]
}

// The action being performed (adding a todo)
let action = {
  type: 'ADD_TODO',
  text: 'Understand the flow.'
}

// Your reducer returns the next application state
let nextState = todoApp(previousState, action)
```

Note that a reducer is a pure function. It only _computes_ the next state. It should be completely predictable: calling it with the same inputs many times should produce the same outputs. It shouldn't perform any side effects like API calls or router transitions. These should happen before an action is dispatched.

注意 reducer 是一个纯函数。它只会 _计算_ 下一个 state。它应该是完全可预测的：即每次给定相同的输入会返回相同的输出。它不应该执行任何像 API 调用或者路由切换这样具有副作用的操作。这些处理应该发生在 action 被 dispatch 之前。

3. **The root reducer may combine the output of multiple reducers into a single state tree.**

**根 reducer 会将多个子 reducers 的输出合并到一个 state 树中。**

How you structure the root reducer is completely up to you. Redux ships with a [`combineReducers()`](../api/combineReducers.md) helper function, useful for “splitting” the root reducer into separate functions that each manage one branch of the state tree.

如何组织根 reducer 的结构完全取决于你。Redux 带有一个 [`combineReducers()`](../api/combineReducers.md) 函数，它可以方便的将根 reducer 拆分进独立的函数中，每个函数负责管理 state 树中的一个分支。

Here's how [`combineReducers()`](../api/combineReducers.md) works. Let's say you have two reducers, one for a list of todos, and another for the currently selected filter setting:

这就是 [`combineReducers()`](../api/combineReducers.md) 的工作原理。假如你有两个 reducers，一个处理 todos 列表，另一个负责当前选中的过滤条件：

```js
function todos(state = [], action) {
  // Somehow calculate it...
  return nextState
}

function visibleTodoFilter(state = 'SHOW_ALL', action) {
  // Somehow calculate it...
  return nextState
}

let todoApp = combineReducers({
  todos,
  visibleTodoFilter
})
```

When you emit an action, `todoApp` returned by `combineReducers` will call both reducers:

当你发出一个 action 时，`combineReducers` 返回的 `todoApp` 会调用每一个 reducer：

```js
let nextTodos = todos(state.todos, action)
let nextVisibleTodoFilter = visibleTodoFilter(state.visibleTodoFilter, action)
```

It will then combine both sets of results into a single state tree:

然后它会将每个 reducer 返回的结果组合成一个 state 树：

```js
return {
  todos: nextTodos,
  visibleTodoFilter: nextVisibleTodoFilter
}
```

While [`combineReducers()`](../api/combineReducers.md) is a handy helper utility, you don't have to use it; feel free to write your own root reducer!

虽然 [`combineReducers()`](../api/combineReducers.md) 是一个很方便的工具，但你并非必须使用它；你完全可以编写自己的根 reducer！

4. **The Redux store saves the complete state tree returned by the root reducer.**

**Redux store 保存了从根 reducer 返回的完整的 state 树。**

This new tree is now the next state of your app! Every listener registered with [`store.subscribe(listener)`](../api/Store.md#subscribe) will now be invoked; listeners may call [`store.getState()`](../api/Store.md#getState) to get the current state.

这个新的 state 树就成了应用的下一个 state！通过 [`store.subscribe(listener)`](../api/Store.md#subscribe) 注册的每一个监听器此时都将被调用。监听器可以调用 [`store.getState()`](../api/Store.md#getState) 来取得当前 state。

Now, the UI can be updated to reflect the new state. If you use bindings like [React Redux](https://github.com/gaearon/react-redux), this is the point at which `component.setState(newState)` is called.

现在，你可以通过更新 UI 来对新的 state 的变化做出反馈了。如果你使用了像 [React Redux](https://github.com/gaearon/react-redux)这样的视图绑定库，那此时就可以通过调用 `component.setState(newState)` 来达到目的。

## Next Steps

Now that you know how Redux works, let's [connect it to a React app](UsageWithReact.md).

既然你已经知道了 Redux 是如何工作的，现在就将它[与 React 应用进行关联(connect)](UsageWithReact.md)

> ##### Note for Advanced Users
>
> If you're already familiar with the basic concepts and have previously completed this tutorial, don't forget to check out [async flow](../advanced/AsyncFlow.md) in the [advanced tutorial](../advanced/README.md) to learn how middleware transforms [async actions](../advanced/AsyncActions.md) before they reach the reducer.
>
> 如果您已经熟悉了基本概念，并且已经完成了本教程，请不要忘了查看  [高级部分](../advanced/README.md) 的 [异步工作流](../advanced/AsyncFlow.md) 来了解在 [异步 actions](../advanced/AsyncActions.md) 到达 reducer 之前中间件是如何对它进行处理和转换的。
