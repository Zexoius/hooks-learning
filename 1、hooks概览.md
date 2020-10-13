

# 简述

2020年了，再提到hooks的确是不该不知，既然平常使用到，就得深研之。

1、解决了什么？

> - 重复代码的抽离与复用，高阶组件的身影减少
> - 更易理解函数动机，调用简单，不复杂的场景对于新手来说显得友好
> - 减少了生命周期声明的重复性
> - 少了令人迷惑的this

2、体验？

> 我是函数式编程的簇拥者，声明式总比命令式读起来更舒服，尽管效率并不能与后者相比，但是代码是写给人看的，在不吹毛求疵的情况下，我会尽量编写函数式风格的，可读性、复用性上来说，更好点。
>
> 当然我并不排斥面向对象、过程等，相反的，在需要的时候，我还是倾向用这些针对具体问题的效率极高的写法。

3、趋势？

> 函数式编程并不是最近大火的（实际上并不是什么大火，它一直慢慢的影响很多人。。。），很早（那时候VB是啥我都不知道）便存在了，只不过这几年，是因为内存足够了（逃
>
> 总之呢，hooks是函数式的体现，写起来简单明了，望文生义，干净利落，结果可预测。



## useState

作为平常最常使用，最先接触的函数，必须先介绍它。

> 与 class 中的 `this.setState`中效果一样，触发它即可触发渲染组件，达到页面更新的效果。

```typescript
import React, { useState } from 'react';

// 顶层声明，而不是在什么方法体、循环体中
// 在这个组件内，它是相对于这个组件的全局变量。
const [ count, setCount ] = useState(0);
```

命名无所谓，但是要做到见到即能知道它起什么效用，保持良好的规范，让自己重读代码的时候能轻松点，不太想掐死当时的自己。

可以无限多个`useState()`，但是这样不是最佳实践（不好看）。



⚠️ 使用它时需要注意以下几点：

> 1. 不像 class 中的 state ，它不会合并，而是全量替换！想要达到合并的效果，可以这么做：
>
> ```typescript
> const initArray = [
>   {
>     name: 'zexus'
>   },
>   {
>     name: 'agnes'
>   }
> ];
> 
> const newArray = [
>   {
>     name: 'ori'
>   }
> ];
> 
> const [demo, setDemo] = useState(initArray);
> 
> setDemo(prev => [...prev, ...newArray]);
> ```
>
> 这里不推荐使用外部变量，哪怕用的就是它指向的变量。hooks本身是偏向函数式的，或者说，react的哲学是函数式的，我们需要引用透明，尽量少的引用外部变量，特别的，在**计时器**中绝对不要这么做（闭包问题），这不是一个很好的实践。



## useReducer

源码里，`useState ` 是从它身上剥离出去的，既然`useState` 是单例的，那么它肯定就是管理多例了。

实际上，它就像简略版的`redux`，思想上是一致的，功能没那么全罢了。

举个栗子：

你有一个列表需要进行**增删**操作，假若只用`useState`，你需要在它的参数内进行函数回调，返回数值。这样的确可以，但是如果对同一组数值进行不同的操作，定义多个操作类的函数显然不是最佳实践（重复性多，每次都是调用setState而已，区别只在于数据的操作），那么我们可以将其**数据操作的部分**提取出来，形成一串流，数据经过它就成为我们想要的了，类似于pipeline。

先举单例的例子：

```typescript
const initTodos = ['看书', '写字', '打pp'];
const [todos, setTodos] = useState<string[]>(initTodos);

// 删除操作：
const delete = (action: string) => setTodos(prev => prev.filter(v => v !== action));

// 新增：
const add = (action: string) => setTodos(prev => [...prev, action]);
```

我为什么要写那么多的setState，还要开辟函数？毕竟写在行内的确太丑了，而且不便于优化性能。

多例的来了：

```typescript
interface ActionType {
  payload: string;
  type: string;
}

function reducer(state: string[], action: ActionType) {
  switch (action.type) {
    case 'add': 
      return [...state, action.payload];
    case 'delete':
      return state.filter(v => v !== action.payload);
    default: 
      return state;
  }
}

const initTodos = ['看书', '写字', '打pp'];
const [todos, dispatch] = useReducer(reducer, initTodos);

// 调用：
dispatch({ type: 'add', payload: '看JK' });
```

状态管理提升了！且数据返回是纯的！