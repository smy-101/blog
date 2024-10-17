---
  title: React渲染原理与useEffect使用技巧
  pubDate: 2024-10-17
  categories: [前端,React]
  description: '介绍了React的渲染和useEffect的相关'
---
React 的渲染原理从根本上来说非常简单，即 `v = f(s)`：数据变了，React 就重新渲染页面。这是因为 React 会调用你的函数组件，最终的目的是更新视图。
## 渲染流程

React 渲染组件会发生两件事情：

1. **创建组件快照**：React 会创建一个组件的快照，它捕获了 React 在那个特定时刻更新视图所需的一切。这个快照中包含了 props、state、event handlers 以及 UI 的描述（基于这些 prop 和 state）。
2. **更新视图**：React 会获取 UI 的描述并使用它来更新视图。为了提高性能，React 使用了虚拟 DOM，它会先将所有更改应用于一棵树，然后再将这些更改批量渲染到真实 DOM 上。这种方法大大减少了直接操作 DOM 的次数，提高了性能。

## 重新渲染

**React 只会在组件状态发生变化时重新渲染**，在 React 中唯一可以触发组件重新渲染的是 state 更改。当事件处理程序调用 updater 函数，使得 React 看到的新状态与快照中的状态不同时，React 才会重新渲染。此时，React 会重新渲染组件，创建新的快照并更新视图。

### 批处理

React 将用户的多个更新请求合并（批处理），并且只有最后一次调用的结果将用作新状态。例如：

```js
const handleClick = () => {
  setCount(1);
  setCount(2);
  setCount(3); // 最终 count = 3
}
```

另一种方法可以告诉 React 使用上一次调用 updater 函数的值，而不是替换它。为此，您需要向 updater 函数传递一个函数本身，该函数将采用最近调用的值作为其参数

```js
const handleClick = () => {
  setCount(1);
  setCount(2);
  setCount(c => c + 3); // 最终 count = 5
}
```

### 组件更新的范围

每当 state 发生变化时，React 都会重新渲染拥有该 state 的组件及其所有子组件——无论这些子组件是否接受任何 props。为了避免不必要的渲染，可以使用 `React.memo` 来优化组件性能。

### StrictMode

`StrictMode` 会导致组件额外渲染一次, 以帮助检测潜在问题。
## `useEffect` 使用注意事项

`useEffect` 是 React 中引起重复渲染的原因之一。以下是一些 `useEffect` 的使用建议：

- 尽量在主程序中避免使用 `useEffect`，如果需要使用，可以通过 hooks 的形式来获取数据，例如 `const {data} = useData()`，将数据隔离在 hooks 中，保持主程序尽量干净只包含数据逻辑。
- 尽量减少在 `useEffect` 中设置数据，不要在 UI 事件中请求数据，通过 UI 事件的回调再请求数据。
	- `const {user, mutate} = useResource('/api/currentUser')`
	- `const {user, mutate} = useResource('/api/currentUser', id)`, 只要 id 变化 user 就变化
- 可以考虑以下几种数据获取方式：
  1. 在事件处理函数中获取数据。
  2. 使用 React Query 库进行数据管理。
  3. 使用 Suspense 和 use。
- 可以容忍的 `useEffect` 一般是需要改变环境的情况，`useEffect(()=>{changeTitle(user.name)}, [user.name])`，改变外部环境，不要改变内部的 state
## 结论

通过这些方式，你可以有效地管理组件的副作用和状态，避免不必要的重新渲染。对于合格的 React 文件，我们应当尽可能避免担心数据是异步还是同步的，做到清晰而简洁。
