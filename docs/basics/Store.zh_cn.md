---
id: store
title: Store
sidebar_label: Store
hide_title: true
---

# Store

In the previous sections, we defined the [actions](Actions.md) that represent the facts about “what happened” and the [reducers](Reducers.md) that update the state according to those actions.

在前面的章节，我们定义了 [actions](Actions.md) 用来代表 “发生了什么”，也定义了 [reducers](Reducers.md) 来根据对应的 action 对 state 进行相应的更新。

The **Store** is the object that brings them together. The store has the following responsibilities:

这里的**Store**是一个将它们合并在一起的对象。它负责做以下事情：

- Holds application state;
- 管理应用状态
- Allows access to state via [`getState()`](../api/Store.md#getState);
- 让你可以通过 [`getState()`](../api/Store.md#getState) 来获取 state；
- Allows state to be updated via [`dispatch(action)`](../api/Store.md#dispatch);
- 让你可以通过 [`dispatch(action)`](../api/Store.md#dispatch) 来跟新 state；
- Registers listeners via [`subscribe(listener)`](../api/Store.md#subscribe);
- 让你可以通过 [`subscribe(listener)`](../api/Store.md#subscribe) 来注册监听器；
- Handles unregistering of listeners via the function returned by [`subscribe(listener)`](../api/Store.md#subscribe).
- 让你可以通过 [`subscribe(listener)`](../api/Store.md#subscribe) 返回的函数来注销对应的监听器。

It's important to note that you'll only have a single store in a Redux application. When you want to split your data handling logic, you'll use [reducer composition](Reducers.md#splitting-reducers) instead of many stores.

值得注意的是，每个 Redux 应用只应该有一个 store。当你希望拆分数据处理逻辑时，你应该使用 [reducer composition](Reducers.md#splitting-reducers) 而不是选择创建多个 stores。

It's easy to create a store if you have a reducer. In the [previous section](Reducers.md), we used [`combineReducers()`](../api/combineReducers.md) to combine several reducers into one. We will now import it, and pass it to [`createStore()`](../api/createStore.md).

如果你已经有 reducer 了，那么创建 store 就非常简单。在 [之前的章节](Reducers.md) 中，我们使用 [`combineReducers()`](../api/combineReducers.md) 来将多个 reducers 组合到一起。我们现在将它 import 进来，然后传递给 [`createStore()`](../api/createStore.md)。

```js
import { createStore } from 'redux'
import todoApp from './reducers'
const store = createStore(todoApp)
```

You may optionally specify the initial state as the second argument to [`createStore()`](../api/createStore.md). This is useful for hydrating the state of the client to match the state of a Redux application running on the server.

你可以给 [`createStore()`](../api/createStore.md) 传递第二个参数，它作为 state 的初始值。这在保持客户端和服务端 Redux 应用 state 一致时非常有用。

```js
const store = createStore(todoApp, window.STATE_FROM_SERVER)
```

## Dispatching Actions

Now that we have created a store, let's verify our program works! Even without any UI, we can already test the update logic.

既然我们已经创建好了 store，那就来验证一下程序是否能工作起来！即使没有 UI，我们也可以测试 state 的更新逻辑。

```js
import {
  addTodo,
  toggleTodo,
  setVisibilityFilter,
  VisibilityFilters
} from './actions'

// Log the initial state
console.log(store.getState())

// Every time the state changes, log it
// Note that subscribe() returns a function for unregistering the listener
const unsubscribe = store.subscribe(() => console.log(store.getState()))

// Dispatch some actions
store.dispatch(addTodo('Learn about actions'))
store.dispatch(addTodo('Learn about reducers'))
store.dispatch(addTodo('Learn about store'))
store.dispatch(toggleTodo(0))
store.dispatch(toggleTodo(1))
store.dispatch(setVisibilityFilter(VisibilityFilters.SHOW_COMPLETED))

// Stop listening to state updates
unsubscribe()
```

You can see how this causes the state held by the store to change:

你可以看到被 store 管理的 state 是如何发生改变的：

<img src='https://i.imgur.com/zMMtoMz.png' width='70%'>

We specified the behavior of our app before we even started writing the UI. We won't do this in this tutorial, but at this point you can write tests for your reducers and action creators. You won't need to mock anything because they are just [pure](../introduction/ThreePrinciples.md#changes-are-made-with-pure-functions) functions. Call them, and make assertions on what they return.

我们甚至在编写 UI 之前就已经明确了应用的行为。此时，你可以为你的 reducers 和 action 创建器编写测试代码，但我们并不会在本教程中做这些。你不需要做任何模拟操作，因此它们本身就是 [纯函数](../introduction/ThreePrinciples.md#changes-are-made-with-pure-functions)。直接调用它们，并对返回值进行断言即可。

## Source Code

#### `index.js`

```js
import { createStore } from 'redux'
import todoApp from './reducers'

const store = createStore(todoApp)
```

## Next Steps

Before creating a UI for our todo app, we will take a detour to see [how the data flows in a Redux application](DataFlow.md).

在为我们的待办事项 app 创建 UI 之前，我们将看看 [Redux 应用的数据流是怎样的](DataFlow.md)。
