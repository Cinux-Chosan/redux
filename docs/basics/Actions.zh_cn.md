---
id: actions
title: Actions
sidebar_label: Actions
hide_title: true
---

# Actions

First, let's define some actions.

首先，我们来定义一些 actions。

**Actions** are payloads of information that send data from your application to your store. They are the _only_ source of information for the store. You send them to the store using [`store.dispatch()`](../api/Store.md#dispatch).

**Actions** 是将数据从应用程序发送到 store 的信息载体。它们是 store _唯一_ 的信息来源。使用 [`store.dispatch()`](../api/Store.md#dispatch) 来将它们发送给 store。

Here's an example action which represents adding a new todo item:

下面是一个添加待办事项的 action 示例：

```js
const ADD_TODO = 'ADD_TODO'
```

```js
{
  type: ADD_TODO,
  text: 'Build my first Redux app'
}
```

Actions are plain JavaScript objects. Actions must have a `type` property that indicates the type of action being performed. Types should typically be defined as string constants. Once your app is large enough, you may want to move them into a separate module.

Actions 是纯 JavaScript 对象。Actions 必须有一个 `type` 属性用来区分需要执行的 action 类型。该字段通常被定义为字符串常量。一旦你的应用增长得足够大，你可能会希望将它们放到单独的模块中。

```js
import { ADD_TODO, REMOVE_TODO } from '../actionTypes'
```

> ##### Note on Boilerplate

> You don't have to define action type constants in a separate file, or even to define them at all. For a small project, it might be easier to just use string literals for action types. However, there are some benefits to explicitly declaring constants in larger codebases. Read [Reducing Boilerplate](../recipes/ReducingBoilerplate.md) for more practical tips on keeping your codebase clean.
> 
> 并不是必须要把 action 类型常量放到独立的文件中，甚至根本不需要定义它们。对于一个小型项目，直接使用字符串来表示 action 类型可能会更加方便。然而，如果项目中有大量代码，那么显式声明这些常量会更好。阅读[Reducing Boilerplate](../recipes/ReducingBoilerplate.md)了解更多关于保持代码库干净整洁的实用技巧。

Other than `type`, the structure of an action object is really up to you. If you're interested, check out [Flux Standard Action](https://github.com/acdlite/flux-standard-action) for recommendations on how actions could be constructed.

除了 `type` 之外，action 对象的结构该如何定义完全取决于你。如果你感兴趣，可以查看 [Flux Standard Action](https://github.com/acdlite/flux-standard-action) 了解一些关于如何构建 action 的建议。

We'll add one more action type to describe a user ticking off a todo as completed. We refer to a particular todo by `index` because we store them in an array. In a real app, it is wiser to generate a unique ID every time something new is created.

我们再添加一个 action 类型用来描述用户完成了待办事项。由于我们使用数组来存放待办事项，因此我们通过 `index` 来引用特定的某一项待办事项。在真实的应用程序中，最好是在每当新事物被创建的时候赋予它一个唯一的 ID。

```js
{
  type: TOGGLE_TODO,
  index: 5
}
```

It's a good idea to pass as little data in each action as possible. For example, it's better to pass `index` than the whole todo object.

在每个 action 中应该传递尽可能少的数据。例如，只传递 `index` 就比传递整个待办事项对象要好。

Finally, we'll add one more action type for changing the currently visible todos.

最后，我们还需要添加一个 action 类型来改变代办事项是否可见。

```js
{
  type: SET_VISIBILITY_FILTER,
  filter: SHOW_COMPLETED
}
```

## Action Creators

**Action creators** are exactly that—functions that create actions. It's easy to conflate the terms “action” and “action creator”, so do your best to use the proper term.

**Action 创建器** 就是创建 action 的函数。由于很容易将 "action" 和 "action creator" 混淆，因此一定要准确区分和使用它们。

In Redux, action creators simply return an action:

在 Redux 中，action 创建器就只是返回一个 action：

```js
function addTodo(text) {
  return {
    type: ADD_TODO,
    text
  }
}
```

This makes them portable and easy to test.

这使得它们具有更好的可移植性和更易于测试。

In [traditional Flux](http://facebook.github.io/flux), action creators often trigger a dispatch when invoked, like so:

在 [traditional Flux](http://facebook.github.io/flux) 中， action 创建器被调用的时候通常会触发 dispatch，像下面这样：

```js
function addTodoWithDispatch(text) {
  const action = {
    type: ADD_TODO,
    text
  }
  dispatch(action)
}
```

In Redux this is _not_ the case.  
Instead, to actually initiate a dispatch, pass the result to the `dispatch()` function:

在 Redux 中就 _不是_ 这样。如果需要发起 dispatch，只需要将（action 创建器返回的）结果传递给 `dispatch()` 函数即可：

```js
dispatch(addTodo(text))
dispatch(completeTodo(index))
```

Alternatively, you can create a **bound action creator** that automatically dispatches:

你也可以创建一个 **整合了 dispatch 的 action 创建器** 来自动 dispatch：

```js
const boundAddTodo = text => dispatch(addTodo(text))
const boundCompleteTodo = index => dispatch(completeTodo(index))
```

Now you'll be able to call them directly:

这样你就可以直接调用它们了：

```js
boundAddTodo(text)
boundCompleteTodo(index)
```

The `dispatch()` function can be accessed directly from the store as [`store.dispatch()`](../api/Store.md#dispatch), but more likely you'll access it using a helper like [react-redux](http://github.com/gaearon/react-redux)'s `connect()`. You can use [`bindActionCreators()`](../api/bindActionCreators.md) to automatically bind many action creators to a `dispatch()` function.

`dispatch()` 函数直接通过 [`store.dispatch()`](../api/Store.md#dispatch) 来访问，但大多数情况你可能会使用像 [react-redux](http://github.com/gaearon/react-redux) 的 `connect()` 这样的 helper 来访问它。你可以使用 [`bindActionCreators()`](../api/bindActionCreators.md) 来自动绑定多个 action 创建器到一个 `dispatch()` 函数。

Action creators can also be asynchronous and have side-effects. You can read about [async actions](../advanced/AsyncActions.md) in the [advanced tutorial](../advanced/README.md) to learn how to handle AJAX responses and compose action creators into async control flow. Don't skip ahead to async actions until you've completed the basics tutorial, as it covers other important concepts that are prerequisite for the advanced tutorial and async actions.

Action 创建器也可以是异步的并且可以有副作用（即不纯）。你可以阅读 [高级手册](../advanced/README.md) 中关于 [异步 actions](../advanced/AsyncActions.md) 的部分来学习如何处理 AJAX 响应和将 action 创建器组合到异步控制流中。不过不要在没有完成基本概念部分的时候就跳到异步 action 部分去，因为基本概念中还包含了一些其他重要概念，这些概念是高级教程和异步操作的先决条件。

## Source Code

### `actions.js`

```js
/*
 * action types
 */

export const ADD_TODO = 'ADD_TODO'
export const TOGGLE_TODO = 'TOGGLE_TODO'
export const SET_VISIBILITY_FILTER = 'SET_VISIBILITY_FILTER'

/*
 * other constants
 */

export const VisibilityFilters = {
  SHOW_ALL: 'SHOW_ALL',
  SHOW_COMPLETED: 'SHOW_COMPLETED',
  SHOW_ACTIVE: 'SHOW_ACTIVE'
}

/*
 * action creators
 */

export function addTodo(text) {
  return { type: ADD_TODO, text }
}

export function toggleTodo(index) {
  return { type: TOGGLE_TODO, index }
}

export function setVisibilityFilter(filter) {
  return { type: SET_VISIBILITY_FILTER, filter }
}
```

## Next Steps

Now let's [define some reducers](Reducers.md) to specify how the state updates when you dispatch these actions!

接下来我们会 [定义一些 reducers](Reducers.md) 来指定当 dispatch action 的时候如何更新 state。
