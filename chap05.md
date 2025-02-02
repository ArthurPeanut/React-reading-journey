# 恰当地获取数据

## 数据流

每次需要让两个没有关联的组件相互通信时，都要找到它们的共有父组件来保存状态。

eg.**需求场景**
有两个独立的组件：
InputComponent：允许用户输入一段文本。
DisplayComponent：实时显示用户输入的文本。
这两个组件没有直接的父子关系。
**代码实现**
```jsx
import React, { useState } from "react";

function inputComponent({ onInputChange }) {
    return (
        <div>
            <h3>Input Component</h3>
            <input
                type="text"
                onChange = {(e) => onInputChange(e.target.value)}
                placeholder = "type something" 
            />
        </div>
    );
}


function outputComponent({ outputContent }) {
    return (
        <div>
            <h3>Output Component</h3>
            <p>Text: {outputContent}</p>
        </div>
    );
}

function ParentComponent() {
    const  [inputText, setInputText] = useState("")

    return (
        <div>
            <h2>Parent Component</h2>
            <inputComponent onInputChange={setInputText} />
            <outputComponent outputContent = {inputText} />
        </div>
    );
}
```
共用父组件保存共享状态`inputText`。

## 数据获取

在组件中移除数据逻辑并在整个应用中复用，可以通过创建高阶组件来解决。
在这里，首先列出一个使用HOC实现复用的例子，再通过自定义hook解决该问题。

### HOC方案

1. 高阶组件：`withDataFetching`
这个 HOC 负责通过 fetch 加载数据，并将加载结果（数据和加载状态）作为 props 提供给子组件。
```jsx
import React from "react";

function withDataFetching(WrappedComponent, url) {
  return class extends React.Component {
    state = {
      data: [],
      loading: true,
      error: null,
    };

    componentDidMount() {
      // fetch data
      fetch(url)
        .then((response) => {
          if (!response.ok) {
            throw new Error("Network response was not ok");
          }
          return response.json();
        })
        .then((data) => this.setState({ data, loading: false }))
        .catch((error) => this.setState({ error: error.message, loading: false }));
    }

    render() {
      const { data, loading, error } = this.state;

      // pass loading status and data to wrapped component
      return <WrappedComponent data={data} loading={loading} error={error} {...this.props} />;
    }
  };
}
```
2. 被增强的子组件：`DataDisplay`
这个组件专注于展示数据，不负责加载逻辑。
```jsx
function DataDisplay({ data, loading, error }) {
  if (loading) {
    return <p>Loading...</p>;
  }

  if (error) {
    return <p>Error: {error}</p>;
  }

  return (
    <ul>
      {data.map((item) => (
        <li key={item.id}>{item.name}</li>
      ))}
    </ul>
  );
}
```

3.使用 HOC 增强组件
将 `DataDisplay` 组件与 `withDataFetching HOC` 结合，为其提供数据加载能力。

```jsx
const EnhancedDataDisplay = withDataFetching(
  DataDisplay,
  "https://jsonplaceholder.typicode.com/users"
);

export default function App() {
  return (
    <div>
      <h1>Data Fetching with HOC</h1>
      <EnhancedDataDisplay />
    </div>
  );
}
```

### 自定义Hook

以下是通过自定义Hook实现该功能。

1. 自定义 Hook：`useDataFetching`
`useDataFetching` 是一个自定义 Hook，它负责加载数据，并返回加载状态、数据和错误信息。
```jsx
import { useState, useEffect } from "react";

function useDataFetch(url) {
    const [data, setData] = useState([]);
    const [loading, setLoading] = useState([]);
    const [error, setError] = useState([]);

    useEffect(() => {
        let isMounted = true;
        setLoading(true);

        fetch(url)
            .then((response) => {
                if (!response.ok) {
                    throw new Error("Failed with fetching");
                }

                return response.json();
            })
            .then((data) => {

            })
    })
}
```


2.被增强的子组件`DataDisplay`专注于展示数据，不负责加载数据。
```jsx
function DataDisplay({ data, loading, error }) {
  if (loading) {
    return <p>Loading...</p>;
  }

  if (error) {
    return <p>Error: {error}</p>;
  }

  return (
    <ul>
      {data.map((item) => (
        <li key={item.id}>{item.name}</li>
      ))}
    </ul>
  );
}
```

3.使用HOC增强组件
将`DataDisplay`与`withDataFetching`HOC结合，为其提供数据加载能力。
```jsx
const EnhancedDataDisplay = withDataFetching(
  DataDisplay,
  "https://jsonplaceholder.typicode.com/users"
);

export default function App() {
  return (
    <div>
      <h1>Data Fetching with HOC</h1>
      <EnhancedDataDisplay />
    </div>
  );
}
```
通过增强组件，分离了数据获取和展示的逻辑，使组件更易复用和测试。

### react-refetch
为什么 react-refetch 不再流行？
**HOC 方式已经过时**

react-refetch 主要依赖 HOC（高阶组件） 来管理数据获取，而现代 React 推荐使用 Hooks（如 useEffect + fetch） 或 数据获取库（如 react-query、SWR）。
HOC 会导致组件嵌套过深（Wrapper Hell），增加调试难度，而 Hooks 方式更直观、简洁。
官方未积极维护

react-refetch 的最新版本已经很久未更新，在 GitHub 上的维护活跃度较低。
目前官方推荐使用 react-query 或 SWR 作为更现代的替代方案。
Hooks 提供更好的解决方案

useEffect 和 useState 让开发者可以直接在函数组件中控制数据加载，而不需要通过 HOC 进行数据注入。
useReducer 和 useContext 也提供了更灵活的全局状态管理方式。
现代数据获取库的崛起

React Query（react-query）
SWR（由 Next.js 团队开发）
这些库提供了 缓存、数据同步、自动刷新、错误处理 等功能，比 react-refetch 更强大、更易用。

1. `react-query`：
使用`useQuery`获取数据：
```jsx
import { useQuery } from "@tanstack/react-query";

function FetchData() {
  const { data, isLoading, error } = useQuery({
    queryKey: ["posts"],
    queryFn: () =>
      fetch("https://jsonplaceholder.typicode.com/posts").then((res) => res.json()),
  });

  if (isLoading) return <p>Loading...</p>;
  if (error) return <p>Error: {error.message}</p>;

  return (
    <ul>
      {data.slice(0, 10).map((post) => (
        <li key={post.id}>{post.title}</li>
      ))}
    </ul>
  );
}

export default FetchData;
```
`react-query` 提供：
* 自动缓存和数据同步
* 后台数据刷新
* 内置错误处理
* 轻松实现分页、懒加载、轮询
2. `SWR`
使用`useSWR`获取数据
```jsx
import useSWR from "swr";

const fetcher = (url) => fetch(url).then((res) => res.json());

function FetchData() {
  const { data, error } = useSWR("https://jsonplaceholder.typicode.com/posts", fetcher);

  if (!data) return <p>Loading...</p>;
  if (error) return <p>Error: {error.message}</p>;

  return (
    <ul>
      {data.slice(0, 10).map((post) => (
        <li key={post.id}>{post.title}</li>
      ))}
    </ul>
  );
}

export default FetchData;
```
API更简单。