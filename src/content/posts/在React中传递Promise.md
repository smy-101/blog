---
  title: 在React中传递Promise
  pubDate: 2024-10-18
  categories: [前端,React]
  description: '介绍了Promise在React中的一种应用'
---

在React中，处理异步操作和错误是一个常见而重要的任务。React提供了两个核心组件——`Suspense`和`ErrorBoundary`，用于处理这些情况。
### Suspense与ErrorBoundary的主要区别

1. **用途不同**：
    - **Suspense**：用于处理异步加载的组件。在加载完成前，它会显示fallback内容。
    - **ErrorBoundary**：用于捕获子组件树中的JavaScript错误，并显示备用UI。
2. **处理的情况不同**：
    - **Suspense**：处理加载状态（loading state）。
    - **ErrorBoundary**：处理错误状态（error state）。
3. **触发机制不同**：
    - **Suspense**：当子组件抛出promise时，它会触发。
    - **ErrorBoundary**：当子组件抛出错误时，它会触发。

这两个组件可以组合使用，以便兼顾加载状态和错误状态的处理。

```
<ErrorBoundary fallback={<div>Error</div>}>
  <Suspense fallback={<div>Loading...</div>}>
    <AsyncComponent />
  </Suspense>
</ErrorBoundary>
```
## 获取 Promise 的加载态 & 错误态

### 使用 Promise

使用 `useState` 或 `useMemo` 来初始化 Promise，以避免重复请求：

```
export default () => {
  const [resultP, setResultP] = useState(() => fetcher('xxx'));
  // 或者
  const resultP = useMemo(() => fetcher('xxx'), []);
  // ...
}
```

在组件中传递和使用 Promise，而不是立即消费它：

```
const promise = fetcher('xxx');

export default () => {
  return (
    <Suspense fallback={<div>loading</div>}>
      <ErrorBoundary fallback={<div>error</div>}>
        <Await resolver={promise}>
          <DataComponent />
        </Await>
      </ErrorBoundary>
    </Suspense>
  );
}
```

使用 useState 或 useMemo 来初始化 Promise,避免重复请求

```
export default () => {
  const [resultP, setResultP] = useState(() => fetcher('xxx'));
  // 或者
  const resultP = useMemo(() => fetcher('xxx'), []);

  // ...
}
```

### 使用自定义的 `Await` 组件

创建一个自定义的 `Await` 组件来处理 Promise：

```
const promise = fetcher('xxx');

export default () => {
    return <Suspense fallback={<div>loading</div>}>
        <ErrorBoundery fallback={<div>error</div>}>
            <Await resolver={promise}></Await>
        </ErrorBoundery>
    </Suspense>
}
```

1. 开始加载数据
2. 在数据加载过程中显示 "loading"
3. 如果加载成功，显示数据（通过 Await 组件）
4. 如果在过程中发生错误，显示 "error"
### 使用 `use` API

我们可以使用一个 `use` API 来简化 Promise 的使用过程：

```
//React的use API
function use(promise) {
  if (promise.status === 'fulfilled') {
    return promise.value;
  } else if (promise.status === 'rejected') {
    throw promise.reason;
  } else if (promise.status === 'pending') {
    throw promise;
  } else {
    promise.status = 'pending';
    promise.then(
      result => {
        promise.status = 'fulfilled';
        promise.value = result;
      },
      reason => {
        promise.status = 'rejected';
        promise.reason = reason;
      },
    );
    throw promise;
  }
}
```
## 总结

通过使用 `Suspense`、`ErrorBoundary` 和自定义的 `Await` 组件，我们可以轻松地管理React应用中的异步加载和错误处理。以下是一个使用示例：

```ts
import { ReactNode, createContext, useContext, useRef, useState } from 'react';

type PromiseCanUse<T> = Promise<T> & {
  status?: 'pending' | 'fulfilled' | 'rejected';
  reason?: unknown;
  value?: T;
};

/**
 * 在当前组件使用 loading/error
 */
export function usePromise<T>(promise?: PromiseCanUse<T>) {
  const [_, forceUpdate] = useState({});
  const ref = useRef<PromiseCanUse<T>>();
  if (!promise) return { loading: false, data: promise };
  ref.current = promise;
  if (!promise.status) {
    promise.status = 'pending';
    promise
      .then(
        result => {
          promise.status = 'fulfilled';
          promise.value = result;
        },
        reason => {
          promise.status = 'rejected';
          promise.reason = reason;
        },
      )
      .finally(() => {
        setTimeout(() => {
          if (ref.current === promise) {
            forceUpdate({});
          }
        }, 0);
      });
  }
  return {
    loading: promise.status === 'pending',
    data: promise.value,
    error: promise.reason,
  };
}

/**
 * 在父级/祖父级组件中使用 Suspense/ErrorBoundery 接收 loading/error
 */
export function use<T>(promise?: PromiseCanUse<T>) {
  if (!promise) return promise;
  if (promise.status === 'fulfilled') {
    return promise.value;
  } else if (promise.status === 'rejected') {
    throw promise.reason;
  } else if (promise.status === 'pending') {
    throw promise;
  } else {
    promise.status = 'pending';
    promise.then(
      result => {
        promise.status = 'fulfilled';
        promise.value = result;
      },
      reason => {
        promise.status = 'rejected';
        promise.reason = reason;
      },
    );
    throw promise;
  }
}

const AsyncDataContext = createContext<unknown>(undefined);

/**
 * 在当前组件或父级/祖父级组件中使用 Suspense/ErrorBoundery 接收 loading/error
 */
export const Await = <T,>(props: {
  resolver?: Promise<T>;
  children?: ReactNode | undefined | ((data?: T) => ReactNode);
}) => {
  const { resolver, children } = props;
  const data = use(resolver);
  if (typeof children === 'function') {
    return children(data);
  }
  return (
    <AsyncDataContext.Provider value={data}>
      {children}
    </AsyncDataContext.Provider>
  );
};

/**
 * 在当前组件接收来自父级 <Await /> 组件的 data
 * @deprecated 不推荐使用, 会丢失 ts 类型
 */
export const useAsyncValue = () => {
  return useContext(AsyncDataContext);
};
```

