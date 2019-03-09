---
id: reducers
title: Reducers
sidebar_label: Reducers
hide_title: true
---

# Reducers

**Reducers** specify how the application's state changes in response to [actions](./Actions.md) sent to the store. Remember that actions only describe _what happened_, but don't describe how the application's state changes.

**Reducers** 指定了当 [actions](./Actions.md) 被发送到 store 时，应用的 state 应该发生何种改变。记住，actions 只用于描述 _发生了什么_ ，但并不包含应用的 state 应该如何改变。

## Designing the State Shape

In Redux, all the application state is stored as a single object. It's a good idea to think of its shape before writing any code. What's the minimal representation of your app's state as an object?

在 Redux 中， 应用的所有 state 都被存放在单个对象中。在开始编码之前应该先确定好这个对象的结构。怎样才是使用一个对象来存放应用 state 的最小方式。

For our todo app, we want to store two different things:

对于我们的待办事项 app 来说，我们希望存放两种数据：

- The currently selected visibility filter.
- 当前过滤条件
- The actual list of todos.
- 待办事项列表

You'll often find that you need to store some data, as well as some UI state, in the state tree. This is fine, but try to keep the data separate from the UI state.

你经常会发现需要在状态树中存储一些数据以及一些 UI 状态。没问题，但需要将数据与 UI 状态分开。

```js
{
  visibilityFilter: 'SHOW_ALL',
  todos: [
    {
      text: 'Consider using Redux',
      completed: true
    },
    {
      text: 'Keep all state in a single tree',
      completed: false
    }
  ]
}
```

> ##### Note on Relationships

> In a more complex app, you're going to want different entities to reference each other. We suggest that you keep your state as normalized as possible, without any nesting. Keep every entity in an object stored with an ID as a key, and use IDs to reference it from other entities, or lists. Think of the app's state as a database. This approach is described in [normalizr's](https://github.com/paularmstrong/normalizr) documentation in detail. For example, keeping `todosById: { id -> todo }` and `todos: array<id>` inside the state would be a better idea in a real app, but we're keeping the example simple.
>
> 在一些更复杂的应用中，你需要不同的实体之间进行相互引用。我们建议你保持 state 越常规化越好，不要有任何嵌套。将每个实体以带 ID 为主键的对象进行存储，并用 ID 来进行相互引用。把应用的 state 想象成一个数据库。[normalizr's](https://github.com/paularmstrong/normalizr) 的文档中详细描述了这种方法。例如，在真实的应用中使用 `todosById: { id -> todo }` 和 `todos: array<id>` 这种结构会更好，但在这里我们为了简化示例而没有这么做。

## Handling Actions

Now that we've decided what our state object looks like, we're ready to write a reducer for it. The reducer is a pure function that takes the previous state and an action, and returns the next state.

既然我们已经确定了状态树的结构，那我们就可以为它编写一个 reducer 了。reducer 是一个纯函数，它接收之前的 state 和一个 action 并返回之后的 state。

```
(previousState, action) => newState
```

It's called a reducer because it's the type of function you would pass to [`Array.prototype.reduce(reducer, ?initialValue)`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/Reduce). It's very important that the reducer stays pure. Things you should **never** do inside a reducer:

它之所以被称作 reducer 是因为它正是传递给 [`Array.prototype.reduce(reducer, ?initialValue)`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/Reduce) 这样的函数类型。保持 reducer 纯净是非常重要的。reducer 中 **不应该** 做以下事情：

- Mutate its arguments;
- 对参数进行修改；
- Perform side effects like API calls and routing transitions;
- 执行具有副作用（异步调用）的操作如 API 调用和路由切换；
- Call non-pure functions, e.g. `Date.now()` or `Math.random()`.
- 调用不纯的函数，如 `Date.now()` 或 `Math.random()` 之类的

We'll explore how to perform side effects in the [advanced walkthrough](../advanced/README.md). For now, just remember that the reducer must be pure. **Given the same arguments, it should calculate the next state and return it. No surprises. No side effects. No API calls. No mutations. Just a calculation.**

在 [高级部分](../advanced/README.md) 我们会讲到如何执行具有副作用的操作。但是现在，只需要记住 reducer 必须保持纯净。即**给定相同的参数，它就会计算并返回下一个 state。没有彩蛋，没有副作用，没有 API 调用，不会修改参数，只是执行计算。**

With this out of the way, let's start writing our reducer by gradually teaching it to understand the [actions](Actions.md) we defined earlier.

以此为前提，我们开始编写 reducer，并逐步教会它理解我们之前定义的 [actions](Actions.md)。

We'll start by specifying the initial state. Redux will call our reducer with an `undefined` state for the first time. This is our chance to return the initial state of our app:

我们将从给 state 指定初始值开始。在 Redux 第一次调用 reducer 时，state 为 `undefined`。我们可以利用这一点来返回应用所需的初始 state 值：

```js
import { VisibilityFilters } from './actions'

const initialState = {
  visibilityFilter: VisibilityFilters.SHOW_ALL,
  todos: []
}

function todoApp(state, action) {
  if (typeof state === 'undefined') {
    return initialState
  }

  // For now, don't handle any actions
  // and just return the state given to us.
  return state
}
```

One neat trick is to use the [ES6 default arguments syntax](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Functions/default_parameters) to write this in a more compact way:

一个简洁的技巧是使用 [ES6 默认参数语法](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Functions/default_parameters) 以编写更紧凑的代码：

```js
function todoApp(state = initialState, action) {
  // For now, don't handle any actions
  // and just return the state given to us.
  return state
}
```

Now let's handle `SET_VISIBILITY_FILTER`. All it needs to do is to change `visibilityFilter` on the state. Easy:

我们现在来处理 `SET_VISIBILITY_FILTER`。它需要做的就是改变 state 中的 `visibilityFilter`，这很简单：

```js
function todoApp(state = initialState, action) {
  switch (action.type) {
    case SET_VISIBILITY_FILTER:
      return Object.assign({}, state, {
        visibilityFilter: action.filter
      })
    default:
      return state
  }
}
```

Note that:

1. **We don't mutate the `state`.** We create a copy with [`Object.assign()`](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/Object/assign). `Object.assign(state, { visibilityFilter: action.filter })` is also wrong: it will mutate the first argument. You **must** supply an empty object as the first parameter. You can also enable the [object spread operator proposal](../recipes/UsingObjectSpreadOperator.md) to write `{ ...state, ...newState }` instead.

**我们没有修改 `state`。**而是使用 [`Object.assign()`](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/Object/assign) 创建了一份新的拷贝。`Object.assign(state, { visibilityFilter: action.filter })` 也是错误的做法：它会对第一个参数做出修改。在这里你**必须**将第一个参数指定为一个空对象。你也可以启用 [object spread operator proposal](../recipes/UsingObjectSpreadOperator.md) 从而可以写成 `{ ...state, ...newState }`。

2. **We return the previous `state` in the `default` case.** It's important to return the previous `state` for any unknown action.

**在 `default` 的情况下我们返回了之前的 `state`。** 对于未知的 action，我们应该返回之前的 `state`。

> ##### Note on `Object.assign`

> [`Object.assign()`](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/Object/assign) is a part of ES6, and is not supported by older browsers. To support them, you will need to either use a polyfill, a [Babel plugin](https://www.npmjs.com/package/babel-plugin-transform-object-assign), or a helper from another library like [`_.assign()`](https://lodash.com/docs#assign).
>
> [`Object.assign()`](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/Object/assign) 是 ES6 的语法，在老式浏览器中并没有得到支持。如果你想要它能够在老式浏览器中得到支持，你要么使用 polyfill —— 一类 [Babel plugin](https://www.npmjs.com/package/babel-plugin-transform-object-assign) ，要么使用第三方库提供的函数，如 [`_.assign()`](https://lodash.com/docs#assign)。

> ##### Note on `switch` and Boilerplate

> The `switch` statement is _not_ the real boilerplate. The real boilerplate of Flux is conceptual: the need to emit an update, the need to register the Store with a Dispatcher, the need for the Store to be an object (and the complications that arise when you want a universal app). Redux solves these problems by using pure reducers instead of event emitters.
>
> `switch` 语句 _并非_ 真正的 boilerplate。真正的 boilerplate 在 Flux 中是概念性的：需要发出更新请求，需要使用一个 Dispatcher 注册到 Store，Store 需要是一个对象（当你只需要一个简单的应用时会出现一些复杂的情况）。Redux 通过使用纯 reducers 去替代事件发生器来解决了这些问题。

> It's unfortunate that many still choose a framework based on whether it uses `switch` statements in the documentation. If you don't like `switch`, you can use a custom `createReducer` function that accepts a handler map, as shown in [“reducing boilerplate”](../recipes/ReducingBoilerplate.md#reducers).
>
> 但不幸的是仍然有许多人在选择框架时会基于文档中是否使用 `switch` 语句来做考虑。如果你不喜欢 `switch`，你可以自定义一个 `createReducer`，它接收一个处理函数的映射，就像 [“reducing boilerplate”](../recipes/ReducingBoilerplate.md#reducers) 中展示的这样。

## Handling More Actions

We have two more actions to handle! Just like we did with `SET_VISIBILITY_FILTER`, we'll import the `ADD_TODO` and `TOGGLE_TODO` actions and then extend our reducer to handle `ADD_TODO`.

我们还有另外两个 actions 需要处理！参照我们处理 `SET_VISIBILITY_FILTER` 的方式，我们将 import `ADD_TODO` 和 `TOGGLE_TODO` 这两个 actions，然后对 reducer 进行扩展以便处理 `ADD_TODO`。

```js
import {
  ADD_TODO,
  TOGGLE_TODO,
  SET_VISIBILITY_FILTER,
  VisibilityFilters
} from './actions'

...

function todoApp(state = initialState, action) {
  switch (action.type) {
    case SET_VISIBILITY_FILTER:
      return Object.assign({}, state, {
        visibilityFilter: action.filter
      })
    case ADD_TODO:
      return Object.assign({}, state, {
        todos: [
          ...state.todos,
          {
            text: action.text,
            completed: false
          }
        ]
      })
    default:
      return state
  }
}
```

Just like before, we never write directly to `state` or its fields, and instead we return new objects. The new `todos` is equal to the old `todos` concatenated with a single new item at the end. The fresh todo was constructed using the data from the action.

像之前那样，我们并不会直接添加或修改 `state` 的字段，而是返回一个新对象。新 `todos` 等同于在原来的 `todos` 后面追加了新的一项，这一项是根据 action 传递进来的数据来构建的。

Finally, the implementation of the `TOGGLE_TODO` handler shouldn't come as a complete surprise:

最后，对于 `TOGGLE_TODO` 的实现也在情理之中。

```js
case TOGGLE_TODO:
  return Object.assign({}, state, {
    todos: state.todos.map((todo, index) => {
      if (index === action.index) {
        return Object.assign({}, todo, {
          completed: !todo.completed
        })
      }
      return todo
    })
  })
```

Because we want to update a specific item in the array without resorting to mutations, we have to create a new array with the same items except the item at the index. If you find yourself often writing such operations, it's a good idea to use a helper like [immutability-helper](https://github.com/kolodny/immutability-helper), [updeep](https://github.com/substantial/updeep), or even a library like [Immutable](http://facebook.github.io/immutable-js/) that has native support for deep updates. Just remember to never assign to anything inside the `state` unless you clone it first.

因为我们希望更新数组中特定项而不影响原数组，因此我们必须创建一个新的数组，它除了 index 指定的那一项外，其他项都一样。如果你经常写这样的代码，你可以尝试使用 [immutability-helper](https://github.com/kolodny/immutability-helper), [updeep](https://github.com/substantial/updeep),甚至是原生支持深层更新的第三方库 [Immutable](http://facebook.github.io/immutable-js/)。记住，除非你一开始对 `state` 进行了克隆，否则不要对 `state` 内部属性赋任何值。

## Splitting Reducers

Here is our code so far. It is rather verbose:

这里是我们目前的代码，看起来有点冗余：

```js
function todoApp(state = initialState, action) {
  switch (action.type) {
    case SET_VISIBILITY_FILTER:
      return Object.assign({}, state, {
        visibilityFilter: action.filter
      })
    case ADD_TODO:
      return Object.assign({}, state, {
        todos: [
          ...state.todos,
          {
            text: action.text,
            completed: false
          }
        ]
      })
    case TOGGLE_TODO:
      return Object.assign({}, state, {
        todos: state.todos.map((todo, index) => {
          if (index === action.index) {
            return Object.assign({}, todo, {
              completed: !todo.completed
            })
          }
          return todo
        })
      })
    default:
      return state
  }
}
```

Is there a way to make it easier to comprehend? It seems like `todos` and `visibilityFilter` are updated completely independently. Sometimes state fields depend on one another and more consideration is required, but in our case we can easily split updating `todos` into a separate function:

是否有一种方式可以让它更易于理解？似乎 `todos` 和 `visibilityFilter` 都是完全独立的更新。有时 state 之间的字段相互依赖，需要做更多的考虑，但目前我们的场景是可以轻易的将 `todos` 拆分成多个函数的：

```js
function todos(state = [], action) {
  switch (action.type) {
    case ADD_TODO:
      return [
        ...state,
        {
          text: action.text,
          completed: false
        }
      ]
    case TOGGLE_TODO:
      return state.map((todo, index) => {
        if (index === action.index) {
          return Object.assign({}, todo, {
            completed: !todo.completed
          })
        }
        return todo
      })
    default:
      return state
  }
}

function todoApp(state = initialState, action) {
  switch (action.type) {
    case SET_VISIBILITY_FILTER:
      return Object.assign({}, state, {
        visibilityFilter: action.filter
      })
    case ADD_TODO:
      return Object.assign({}, state, {
        todos: todos(state.todos, action)
      })
    case TOGGLE_TODO:
      return Object.assign({}, state, {
        todos: todos(state.todos, action)
      })
    default:
      return state
  }
}
```

Note that `todos` also accepts `state`—but `state` is an array! Now `todoApp` gives `todos` just a slice of the state to manage, and `todos` knows how to update just that slice. **This is called _reducer composition_, and it's the fundamental pattern of building Redux apps.**

注意 `todos` 也接收 `state` 参数 —— 但 `state` 是一个数组！现在 `todoApp` 只传给 `todos` 它需要的那一部分 state，并且 `todos` 知道如何使用这部分数据。**这被称作 _reducer 组合_，它是构建 Redux 应用的基本模式。**

Let's explore reducer composition more. Can we also extract a reducer managing just `visibilityFilter`? We can.

我们来对 reducer 组合做更进一步的探索。我们是否能够提取一个管理 `visibilityFilter` 的 reducer 呢？当然可以。

Below our imports, let's use [ES6 Object Destructuring](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment) to declare `SHOW_ALL`:

在下面我们使用了 [ES6 对象解构](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment) 来声明 `SHOW_ALL`：

```js
const { SHOW_ALL } = VisibilityFilters
```

Then:

```js
function visibilityFilter(state = SHOW_ALL, action) {
  switch (action.type) {
    case SET_VISIBILITY_FILTER:
      return action.filter
    default:
      return state
  }
}
```

Now we can rewrite the main reducer as a function that calls the reducers managing parts of the state, and combines them into a single object. It also doesn't need to know the complete initial state anymore. It's enough that the child reducers return their initial state when given `undefined` at first.

现在我们可以将主 reducer 重写为一个函数，它调用其它 reducers 来管理 state 中各自对应的那部分数据，并将它们合并到一个对象中。它也不需要知道完整的初始 state 了。每个子 reducers 在一开始接收到 `undefined` 作为 state 时返回各自的初始状态就够了。

```js
function todos(state = [], action) {
  switch (action.type) {
    case ADD_TODO:
      return [
        ...state,
        {
          text: action.text,
          completed: false
        }
      ]
    case TOGGLE_TODO:
      return state.map((todo, index) => {
        if (index === action.index) {
          return Object.assign({}, todo, {
            completed: !todo.completed
          })
        }
        return todo
      })
    default:
      return state
  }
}

function visibilityFilter(state = SHOW_ALL, action) {
  switch (action.type) {
    case SET_VISIBILITY_FILTER:
      return action.filter
    default:
      return state
  }
}

function todoApp(state = {}, action) {
  return {
    visibilityFilter: visibilityFilter(state.visibilityFilter, action),
    todos: todos(state.todos, action)
  }
}
```

**Note that each of these reducers is managing its own part of the global state. The `state` parameter is different for every reducer, and corresponds to the part of the state it manages.**

**注意，每个 reducers 只管理全局 state 中自己的那一部分。对每个 reducer 来说 `state` 参数都不一样，它只对应的是 state 中自己管理的那一部分。**

This is already looking good! When the app is larger, we can split the reducers into separate files and keep them completely independent and managing different data domains.

这看起来好了很多！当应用变得越来越大时，我们完全可以将 reducers 拆分到多个独立的文件中，并各自管理不同的数据域。

Finally, Redux provides a utility called [`combineReducers()`](../api/combineReducers.md) that does the same boilerplate logic that the `todoApp` above currently does. With its help, we can rewrite `todoApp` like this:

最终，Redux 提供了一个叫 [`combineReducers()`](../api/combineReducers.md) 的工具函数来实现与上面 `todoApp` 中的 boilerplate 相同的逻辑。在它的帮助下，我们可以向下面这样重写 `todoApp`：

```js
import { combineReducers } from 'redux'

const todoApp = combineReducers({
  visibilityFilter,
  todos
})

export default todoApp
```

Note that this is equivalent to:

它等同于：

```js
export default function todoApp(state = {}, action) {
  return {
    visibilityFilter: visibilityFilter(state.visibilityFilter, action),
    todos: todos(state.todos, action)
  }
}
```

You could also give them different keys, or call functions differently. These two ways to write a combined reducer are equivalent:

你当然也可以给它们指定键名，或者调用不同的函数。这两种方式都一样：

```js
const reducer = combineReducers({
  a: doSomethingWithA,
  b: processB,
  c: c
})
```

```js
function reducer(state = {}, action) {
  return {
    a: doSomethingWithA(state.a, action),
    b: processB(state.b, action),
    c: c(state.c, action)
  }
}
```

All [`combineReducers()`](../api/combineReducers.md) does is generate a function that calls your reducers **with the slices of state selected according to their keys**, and combines their results into a single object again. [It's not magic.](https://github.com/reduxjs/redux/issues/428#issuecomment-129223274) And like other reducers, `combineReducers()` does not create a new object if all of the reducers provided to it do not change state.

[`combineReducers()`](../api/combineReducers.md) 做的所有操作就仅仅是生成一个函数，这个函数会根据 reducers 对象中每个 reducer 的键名筛选出 state 中对应的那一部分数据并作为参数调用对应的 reducer，然后把它们返回的结果合并到单个对象中。 [这并不神奇。](https://github.com/reduxjs/redux/issues/428#issuecomment-129223274) 并且就像其它 reducers 一样，如果传递给 `combineReducers()` 的每个 reducer 都没有返回新的 state，则它也不会新创建一个对象。

> ##### Note for ES6 Savvy Users

> Because `combineReducers` expects an object, we can put all top-level reducers into a separate file, `export` each reducer function, and use `import * as reducers` to get them as an object with their names as the keys:
>
> 由于 `combineReducers` 希望得到一个对象作为参数，因此我们可以将所有的顶级 reducers 放到一个单独的文件中，并通过 `export` 导出每个 reducer，然后使用 `import * as reducers` 来获取一个每个字段键名为对应函数的对象：

> ```js
> import { combineReducers } from 'redux'
> import * as reducers from './reducers'
>
> const todoApp = combineReducers(reducers)
> ```
>
> Because `import *` is still new syntax, we don't use it anymore in the documentation to avoid [confusion](https://github.com/reduxjs/redux/issues/428#issuecomment-129223274), but you may encounter it in some community examples.
> 由于 `import *` 仍然是一个较新的语法，为了避免产生这样的[疑惑](https://github.com/reduxjs/redux/issues/428#issuecomment-129223274)，在接下来的文档中我们不会再使用它，但是你可能会在一些社区示例中看到它。

## Source Code

#### `reducers.js`

```js
import { combineReducers } from 'redux'
import {
  ADD_TODO,
  TOGGLE_TODO,
  SET_VISIBILITY_FILTER,
  VisibilityFilters
} from './actions'
const { SHOW_ALL } = VisibilityFilters

function visibilityFilter(state = SHOW_ALL, action) {
  switch (action.type) {
    case SET_VISIBILITY_FILTER:
      return action.filter
    default:
      return state
  }
}

function todos(state = [], action) {
  switch (action.type) {
    case ADD_TODO:
      return [
        ...state,
        {
          text: action.text,
          completed: false
        }
      ]
    case TOGGLE_TODO:
      return state.map((todo, index) => {
        if (index === action.index) {
          return Object.assign({}, todo, {
            completed: !todo.completed
          })
        }
        return todo
      })
    default:
      return state
  }
}

const todoApp = combineReducers({
  visibilityFilter,
  todos
})

export default todoApp
```

## Next Steps

Next, we'll explore how to [create a Redux store](Store.md) that holds the state and takes care of calling your reducer when you dispatch an action.

接下来，我们探索[如何创建一个 Redux store](Store.md)，它可以保存 state，并在你 dispatch action 时调用 reducer。
