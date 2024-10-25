---
  title: React组件设计模式
  pubDate: 2024-10-25
  categories: [前端,React]
  description: '介绍设计组件的几种模式'
---
## 复合组件模式

- 这是一种允许创建富有表现力和声明性组件的模式
- 通过 Context 共享状态，避免 prop drilling 问题
- 提供更好的关注点分离和易于理解的 API

- 使用 Context (Provider + useContext) 在父子组件间共享状态和处理函数
- 允许开发者灵活组合和排序子组件
- 将逻辑集中在父组件中，通过 Context 分发给子组件

```
import React from "react";
import { Counter } from "./Counter";

function Usage() {
  const handleChangeCounter = (count) => {
    console.log("count", count);
  };

  return (
    <Counter onChange={handleChangeCounter}>
      <Counter.Decrement icon="minus" />
      <Counter.Label>Counter</Counter.Label>
      <Counter.Count max={10} />
      <Counter.Increment icon="plus" />
    </Counter>
  );
}

export { Usage };
```

- 降低 API 复杂性：避免将所有 props 集中在一个巨大的父组件中
- 灵活的标记结构：允许开发者自由组合和排序子组件
- 关注点分离：逻辑集中在父组件，UI 分布在子组件中
- 直观的 API：组件使用方式类似于原生 HTML

- 过度灵活：可能导致不可预期的使用方式
- JSX 代码量增加：特别是在使用代码格式化工具时
- 可能出现错误的组件组合：由于灵活性，开发者可能会错误地组合组件
### 使用场景

- 需要灵活UI结构的组件
- 需要避免 prop drilling 的场景
- 需要良好关注点分离的场景
- 复杂的表单控件或UI组件
### 最佳实践

- 提供清晰的组件命名
- 确保子组件的独立性
- 文档化组件的组合规则
- 考虑添加类型检查
- 提供合理的默认行为
## 受控属性模式

- 这是一种将组件转换为受控组件的模式
- 使用外部状态作为"单一事实来源"
- 允许开发者通过外部状态直接控制组件行为

- 组件接受外部状态作为props
- 组件的行为由外部状态控制
- 通过回调函数通知外部状态更新
- 开发者可以在状态更新过程中插入自定义逻辑

```
import React, { useState } from "react";
import { Counter } from "./Counter";

function Usage() {
  const [count, setCount] = useState(0);

  const handleChangeCounter = (newCount) => {
    setCount(newCount);
  };
  return (
    <Counter value={count} onChange={handleChangeCounter}>
      <Counter.Decrement icon={"minus"} />
      <Counter.Label>Counter</Counter.Label>
      <Counter.Count max={10} />
      <Counter.Increment icon={"plus"} />
    </Counter>
  );
}

export { Usage };
```

- 更强的控制能力：开发者可以完全控制组件的状态
- 灵活性：可以在状态更新过程中添加自定义逻辑
- 可预测性：状态变化更加可控和可预测
- 与React 生态系统的一致性：遵循 React 的受控组件模式

- 实现复杂度增加：需要在多个地方管理状态（JSX、useState、handleChange）
- 代码量增加：相比非受控组件需要更多的样板代码
- 状态管理的责任转移：将状态管理的责任转移给了使用该组件的开发者
### 使用场景

- 需要精确控制组件状态的场景
- 需要在状态更新时执行额外逻辑的场景
- 需要与其他组件或系统集成的场景
- 表单控件和输入组件
### 最佳实践

- 提供清晰的 props 接口
- 保持状态更新的一致性
- 提供适当的默认值
- 添加必要的类型检查
- 文档化组件的受控行为

## 自定义 hooks 模式

- 这是一种将组件逻辑抽离到自定义 Hook 中的模式
- 通过暴露内部状态和处理函数，让开发者获得更多控制权
- 实现了逻辑层和视图层的完全分离

- 将组件的主要逻辑封装在自定义 Hook 中
- Hook 返回必要的状态和处理函数
- 开发者可以在 Hook 和组件之间添加自定义逻辑
- 组件负责渲染，Hook 负责逻辑处理

```
import React from "react";
import { Counter } from "./Counter";
import { useCounter } from "./useCounter";

function Usage() {
  const { count, handleIncrement, handleDecrement } = useCounter(0);
  const MAX_COUNT = 10;

  const handleClickIncrement = () => {
    //Put your custom logic
    if (count < MAX_COUNT) {
      handleIncrement();
    }
  };

  return (
    <>
      <Counter value={count}>
        <Counter.Decrement
          icon={"minus"}
          onClick={handleDecrement}
          disabled={count === 0}
        />
        <Counter.Label>Counter</Counter.Label>
        <Counter.Count />
        <Counter.Increment
          icon={"plus"}
          onClick={handleClickIncrement}
          disabled={count === MAX_COUNT}
        />
      </Counter>
      <button onClick={handleClickIncrement} disabled={count === MAX_COUNT}>
        Custom increment btn 1
      </button>
    </>
  );
}

export { Usage };
```

- 提供更多控制权：开发者可以访问和修改内部逻辑
- 逻辑复用：Hook 可以在多个组件间共享
- 关注点分离：清晰地区分了业务逻辑和UI渲染
- 灵活性：允许在 Hook 和组件之间插入自定义逻辑

- 实现复杂度增加：需要手动连接逻辑部分和渲染部分
- 学习成本：需要充分理解组件的工作原理
- 使用门槛：开发者需要正确理解如何使用返回的状态和方法
### 使用场景

- 需要在多个组件间共享逻辑时
- 需要给开发者更多控制权时
- 复杂的状态管理场景
- 需要将业务逻辑与UI完全分离的场景

### 最佳实践

- 保持 Hook 的单一职责
- 提供清晰的命名和类型定义
- 考虑添加错误处理和验证
- 提供详细的文档说明
- 保持返回值的一致性
## Props Getter 模式

- Props Getters Pattern 是一种通过提供预配置的 props 集合来简化组件使用的模式
- 通过 getter 函数封装组件所需的所有必要属性和行为
- getter 函数返回预配置的 props 对象，可以直接展开到对应的 JSX 元素上

- 定义具有语义化名称的 getter 函数（如 getCounterProps, getIncrementProps）
- 每个 getter 函数返回相关的 props 集合
- 用户通过调用对应的 getter 函数获取所需的 props
- 支持通过传入自定义 props 来覆盖默认行为

```
import React from "react";
import { Counter } from "./Counter";
import { useCounter } from "./useCounter";

const MAX_COUNT = 10;

function Usage() {
  const {
    count,
    getCounterProps,
    getIncrementProps,
    getDecrementProps
  } = useCounter({
    initial: 0,
    max: MAX_COUNT
  });

  const handleBtn1Clicked = () => {
    console.log("btn 1 clicked");
  };

  return (
    <>
      <Counter {...getCounterProps()}>
        <Counter.Decrement icon={"minus"} {...getDecrementProps()} />
        <Counter.Label>Counter</Counter.Label>
        <Counter.Count />
        <Counter.Increment icon={"plus"} {...getIncrementProps()} />
      </Counter>
      <button {...getIncrementProps({ onClick: handleBtn1Clicked })}>
        Custom increment btn 1
      </button>
      <button {...getIncrementProps({ disabled: count > MAX_COUNT - 2 })}>
        Custom increment btn 2
      </button>
    </>
  );
}

export { Usage };
```

- 简化使用复杂度：隐藏了内部实现细节，用户只需关注getter 函数
- 提供语义化的 API：getter 函数名称直观地表明其用途
- 保持灵活性：允许通过覆盖默认 props 来自定义行为
- 集中管理：所有相关的 props 和行为被统一管理

- 抽象层级增加：getter 函数增加了一层抽象，可能使代码行为不够直观
- 需要良好文档：用户需要清楚了解每个 getter 函数的作用和可覆盖的属性
- 调试难度增加：当出现问题时，可能较难追踪具体原因
### 使用场景

- 组件具有复杂的内部状态和行为时
- 需要提供简化的 API 接口时
- 希望在保持灵活性的同时降低使用复杂度时
### 最佳实践

- 为 getter 函数提供清晰的命名
- 允许通过参数覆盖默认行为
- 提供详细的 TypeScript 类型定义
- 编写清晰的文档说明每个 getter 的用途和可配置项
## State reducer

- 在自定义 Hook 的基础上添加 reducer 参数
- 允许开发者访问和重写组件的所有内部 actions
- 可以拦截和修改任何状态更新操作

```
import React from "react";
import { Counter } from "./Counter";
import { useCounter } from "./useCounter";

const MAX_COUNT = 10;
function Usage() {
  const reducer = (state, action) => {
    switch (action.type) {
      case "decrement":
        return {
          //The decrement delta was changed for 2 (Default is 1)
          count: Math.max(0, state.count - 2)
        };
      default:
        return useCounter.reducer(state, action);
    }
  };

  const { count, handleDecrement, handleIncrement } = useCounter(
    { initial: 0, max: 10 },
    reducer
  );

  return (
    <>
      <Counter value={count}>
        <Counter.Decrement icon={"minus"} onClick={handleDecrement} />
        <Counter.Label>Counter</Counter.Label>
        <Counter.Count />
        <Counter.Increment icon={"plus"} onClick={handleIncrement} />
      </Counter>
      <button onClick={handleIncrement} disabled={count === MAX_COUNT}>
        Custom increment btn 1
      </button>
    </>
  );
}

export { Usage };
```

- 提供最大程度的控制灵活性
- 允许开发者完全自定义组件的内部行为
- 所有内部 actions 都可以从外部访问和重写
- 可以与其他模式（如 Compound Components、Custom Hook、Props Getters）结合使用

- 实现复杂度最高，对组件开发者和使用者都具有挑战性
- 需要深入理解组件的内部逻辑才能正确使用
- 由于可以修改任何reducer action，增加了出错的风险

### 使用场景

- 当需要对组件行为进行深度定制时
- 需要处理复杂的状态逻辑时
- 需要完全控制组件内部运行机制时

### 最佳实践

- 在使用此模式前，需要权衡实现复杂度和控制需求
- 建议提供详细的文档说明内部 actions 的作用
- 考虑使用 TypeScript 来提供更好的类型提示和安全性
