# 开发真正可复用的组件

## 创建类

### createClass

```jsx
const Button = React.createClass({     
    render() {         
        return <button />     
    },
})
```
### extends React.Component

```jsx
class Button extends React.Component {
    render() {
        return <button />
    }
}
```

### 主要区别
* prop
用`propTypes`列出能够传递给组件的所有prop，用`getDefaultProps`定义prop的默认值。

* 状态
组件初始状态的定义方式不同。
`createClass`需要调用函数，而使用ES2015的类则需要设置实例的属性。
```jsx
const Button = React.createClass({
    getInitialState() { {/* returns an object with default value */}
        return {
            text: 'Click me!',
        }
    },

    render() {
        return <button>{this.state.text}</button>
    },
})
```
如果用类来定义初始状态，则需要在构造器方法中设置实例的状态属性：
```jsx
class Button extends React.Component {
    constructor(props) {
        super(props)

        this.state = {
            text: 'Click me!'
        }
    }

    ...
}
```
* 自动绑定
createClass允许我们创建事件处理器，并当我们调用它时，this会指向组件本身。
而类中需要手动绑定函数。
可以通过bind或箭头函数解决。
```jsx
class Button extends React.Component {
  handleClick = () => {
    console.log(this);
  };

  render() {
    return <button onClick={this.handleClick}>Click Me</button>;
  }
}

```
箭头函数不会创建自己的this，它会继承外部作用域的this。故而在方法定义中使用箭头函数。

* 无状态组件
仅接收props来渲染UI，不管理自己的状态。
React Hooks（如 useState 和 useEffect）的引入模糊了无状态组件和有状态组件之间的界限。现在，函数组件也可以管理状态和副作用。
```js
import React, { useState } from 'react';

const Counter = () => {
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
    </div>
  );
};
```

## 状态
将任意函数作为`setState`的第二个参数传递，状态更新完成时会触发该函数，同时组建完成渲染。

应当总是将`setState`当作异步的。

**将满足需求的最少数据放到状态中。**

## 可复用组件

以下实例将不可复用的组件改成接口清晰通用的可复用组件。
该组件从API路径加载一个消息集合，并在屏幕上显示列表。
```js
class PostList extends React.Component {
    constructor(props) {
        super(props)

        this.state = {
            posts: [],
        }
    }

    componentDidMount() {
        Posts.fetch().then(posts => {
            this.setState({ posts })
        })
    }

    render() {
        return (
            <ul>
                {this.state.posts.map(post => (
                    <li key={post.id}>
                        <h1>{post.title}</h1>
                        {post.excerpt && <p>{post.excerpt}</p>}
                    </li>
                ))}
            </ul>
        )
    }
};
```
为了创建可复用组件，首先创建通用的列表组件List。其prop如下：
```js
List.propTypes = {
    collection: React.PropTypes.array,
    textKey: React.PropTypes.string,
    titleKey: React.PropTypes.string,
}
```
可以将其写作无状态函数式组件：
```js
const List = ({ collection, textKey, titleKey }) => {
    <ul>
        {collection.map(item => 
            <Item
                key={item.id}
                text={item[textKey]}
                title={item[titleKey]}
            />
        )}
    </ul>
}
```
List组件接收prop，对集合进行迭代。子组件传入了标题和文本这两个prop。
```js
const Item = ({ text, title }) => (
    <li>
        <h1>{title}</h1>
        {text && <p>{text}</p>}
    </li>
)
```
