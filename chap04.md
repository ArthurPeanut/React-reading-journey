# 组合一切

## 组件间通信

* children
children 是一个特殊的 props，用于表示组件的子节点。它允许开发者将任意 React 元素或组件嵌套在一个组件内部。
```js
function Container(props) {
  return <div className="container">{props.children}</div>;
}

// Call Container
<Container>
  <p>This is inside the container.</p>
</Container>;
```
这里`<Container>`是一个组件，内部嵌套了`<p>`元素作为子节点。React 自动将`<p>This is inside the container.</p>`作为 children 传递给 Container 组件的 props。在Container 组件中，props.children 就代表了这个`<p>`元素。

## 容器组件和表现组件模式
容器组件（Container Components） 和 表现组件（Presentational Components） 是一种常见的分离逻辑和 UI 的设计模式。它将组件分为两类：
* 表现组件：负责显示内容和样式。
    * 以props的形式从父组件接收数据
    * 通常写作无状态函数式组件
* 容器组件：负责处理数据和逻辑。

## 高阶组件（Higher-Order Component, HOC）
高阶组件就是函数，它接收组件作为参数，对组件增强后返回。
```js
const EnhancedComponent = higherOrderComponent(WrappedComponent);
```
* higherOrderComponent：高阶组件函数，用于增强原组件。
* WrappedComponent：原始组件，需要增强的组件。
* EnhancedComponent：返回的新组件，包含了增强后的功能。

以下创建一个加载状态管理的高阶组件。
定义一个HOC:
```js
function withLoading(WrappedComponent) {
  return function EnhancedComponent({ isLoading, ...props }) {
    if (isLoading) {
      return <div>Loading...</div>;
    }

    return <WrappedComponent {...props} />;
  };
}
```
使用HOC：
```js
function UserList(props) {
  return (
    <ul>
      {props.users.map((user) => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  );
}

const UserListWithLoading = withLoading(UserList);

// 在应用中使用
<UserListWithLoading isLoading={true} />;
<UserListWithLoading isLoading={false} users={[{ id: 1, name: "Alice" }]} />;
```
当 isLoading 为 true：显示 Loading...。
当 isLoading 为 false：显示用户列表。

## recompose

Recompose 是一个为 React 提供高阶组件（HOC）的库，它在 React Hooks 引入之前非常流行，因为它提供了一些工具来简化高阶组件的创建和组合。但随着 React 16.8 引入 Hooks，Recompose 的使用频率显著下降。

## Hooks

书中推荐的mixin，HOC和recompose都随着Hooks的普及而逐渐淘汰，这里补充Hooks实现逻辑复用的内容。

### 计数器

自定义一个Hook来封装计数器的状态管理。
```js
import { useState } from "react";

function useCounter(initialValue = 0) {
  const [count, setCount] = useState(initialValue);

  const increment = () => setCount((prev) => prev + 1);
  const decrement = () => setCount((prev) => prev - 1);

  return { count, increment, decrement };
}
```
在两个组件中使用该hook：
```js
function CounterA() {
  const { count, increment, decrement } = useCounter(0);

  return (
    <div>
      <h2>Counter A</h2>
      <p>Count: {count}</p>
      <button onClick={increment}>Increment</button>
      <button onClick={decrement}>Decrement</button>
    </div>
  );
}

function CounterB() {
  const { count, increment, decrement } = useCounter(5);

  return (
    <div>
      <h2>Counter B</h2>
      <p>Count: {count}</p>
      <button onClick={increment}>Increment</button>
      <button onClick={decrement}>Decrement</button>
    </div>
  );
}
```

### 窗口大小监听
自定义Hook：
```js
import { useState, useEffect } from "react";

function useWindowSize() {
  const [windowSize, setWindowSize] = useState({
    width: window.innerWidth,
    height: window.innerHeight,
  });

  useEffect(() => {
    const handleResize = () => {
      setWindowSize({
        width: window.innerWidth,
        height: window.innerHeight,
      });
    };

    window.addEventListener("resize", handleResize);
    return () => window.removeEventListener("resize", handleResize); // 清理事件
  }, []);

  return windowSize;
}
```
`useEffect`第一个参数的返回值是清理函数，它在组件重新渲染时执行，以清理上一次的副作用。它在组件卸载时执行，以清理所有副作用。
清理函数执行顺序：组件重新渲染时：先清理上一次的副作用，然后执行新的副作用逻辑->组件卸载时：执行最后一次清理函数。
使用自定义hook：
```js
function DisplayWidth() {
  const { width } = useWindowSize();
  return <p>Current width: {width}px</p>;
}

function DisplayHeight() {
  const { height } = useWindowSize();
  return <p>Current height: {height}px</p>;
}
```
