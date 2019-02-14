# 路由

> [dojo/framework/src/routing/README.md](https://github.com/dojo/framework/blob/master/src/routing/README.md)
>
> commit 5e6f9d286eb3bca5da0f789f94d18a0c4a320849

<!-- start-github-only -->

Dojo 应用程序的路由

- [路由](#%E8%B7%AF%E7%94%B1)
  - [Features](#features)
    - [Route 配置](#route-%E9%85%8D%E7%BD%AE)
      - [路径参数](#%E8%B7%AF%E5%BE%84%E5%8F%82%E6%95%B0)
    - [Router](#router)
      - [History Managers](#history-managers)
        - [Hash History](#hash-history)
        - [State History](#state-history)
        - [Memory History](#memory-history)
      - [Outlet Event](#outlet-event)
    - [Router Context Injection](#router-context-injection)
    - [Outlets](#outlets)
      - [Global Error Outlet](#global-error-outlet)
    - [Link](#link)
    - [ActiveLink](#activelink)

<!-- end-github-only -->

## Features

部件（Widget）是 Dojo 应用程序的基本概念，因此 Dojo 路由提供了一组与应用程序中的部件直接集成的组件。这些组件能将部件注册到路由上，且不需要掌握路由相关的任何知识。Dojo 应用程序中的路由包括以下内容：

- `Outlet` 部件封装器指定 route 的 outlet key 和表现视图之间的映射关系
- `Route` 配置，用于在路径和 outlet key 之间建立映射关系
- `Router` 基于当前路径解析 `Route`
- `History` 提供器负责向 `Router` 通知路径的更改
- `Registry` 将 `Router` 注入到部件系统中

### Route 配置

`RouteConfig` 用于注册应用程序的路由，它定义了路由的 `path`、关联的 `outlet` 以及嵌套的子 `RouteConfig`。一个完整的路由就是由递归嵌套的 Route 构成的。

路由配置示例：

```ts
import { RouteConfig } from '@dojo/framework/routing/interfaces';

const config: RouteConfig[] = [
    {
        path: 'foo',
        outlet: 'root',
        children: [
            {
                path: 'bar',
                outlet: 'bar'
            },
            {
                path: 'baz',
                outlet: 'baz',
                children: [
                    {
                        path: 'qux',
                        outlet: 'qux'
                    }
                ]
            }
        ]
    }
];
```

此配置将注册以下路由和 outlet：

| Route          | Outlet |
| -------------- | ------ |
| `/foo`         | `root` |
| `/foo/bar`     | `bar`  |
| `/foo/baz`     | `baz`  |
| `/foo/baz/qux` | `qux`  |

#### 路径参数

在 `RouteConfig` 的 path 属性中，在 `path` 的值中使用大括号可定义路径参数。 Parameters will match any segment and the value of that segment is made available to matching outlets via the [mapParams](#mapParams) `Outlet` options. The parameters provided to child outlets will include any parameters from matching parent routes.

```ts
const config = [
    {
        path: 'foo/{foo}',
        outlet: 'foo'
    }
];
```

具有路径参数的路由，可为每个路由指定默认的参数。当用没有指定参数的 outlet 生成一个链接，或者当前路由中不存在参数时，可使用这些默认参数值。

```ts
const config = [
    {
        path: 'foo/{foo}',
        outlet: 'foo',
        defaultParams: {
            foo: 'bar'
        }
    }
];
```

可使用可选的配属属性 `defaultRoute` 来设置默认路由，如果当前路由没有匹配到已注册的路由，就使用此路由。

```ts
const config = [
    {
        path: 'foo/{foo}',
        outlet: 'foo',
        defaultRoute: true
    }
];
```

### Router

`Router` 用于注册 [route 配置](#route-配置)，将 route 配置信息传入 Router 的构造函数即可：

```ts
const router = new Router(config);
```

会自动为 router 注册一个 `HashHistory` 历史管理器（history manager）。可在第二个参数中传入其他历史管理器。

```ts
import { MemoryHistory } from '@dojo/framework/routing/MemoryHistory';

const router = new Router(config, { HistoryManager: MemoryHistory });
```

使用应用程序路由配置创建路由后，需要让应用程序中的所有组件可使用这些路由。这是通过使用 `@dojo/framework/widget-core/Registry` 中的 `Registry`，定义一个将 `invalidator` 连接到 router 的 `nav` 事件的注入器，并返回 `router` 实例实现的。这里使用 key 来定义注入器，路由器的默认 key 值为 `router`。

```ts
import { Registry } from '@dojo/framework/widget-core/Registry';
import { Injector } from '@dojo/framework/widget-core/Injector';

const registry = new Registry();

// 假设我们有一个可用的 router 实例
registry.defineInjector('router', () => {
    router.on('nav', () => invalidator());
    return () => router;
};
```

**注意：** 路由提供了 [注册 router 的快捷方法](#router-context-injection)。

最后，为了让应用程序中的所有部件都能使用 `registry`，需要将其传给 vdom `renderer` 的 `.mount()` 方法。

```ts
const r = renderer(() => v(App, {}));
r.mount({ registry });
```

#### History Managers

路由自带三个历史管理器，用于监视和更改导航状态：`HashHistory`、`StateHistory` 和 `MemoryHistory`。默认使用 `HashHistory`，但是可在创建 `Router` 时传入不同的 `HistoryManager`。

```ts
const router = new Router(config, { HistoryManager: MemoryHistory });
```

##### Hash History

基于哈希的管理器使用片段标识符（fragment identifier）来存储导航状态，是 `@dojo/framework/routing` 中的默认管理器。

```ts
import { Router } from '@dojo/framework/routing/Router';
import { HashHistory } from '@dojo/framework/routing/history/HashHistory';

const router = new Router(config, { HistoryManager: HashHistory });
```

历史管理器有 `current`、`set(path: string)` 和 `prefix(path: string)` 三个API。`HashHistory` 类假定全局对象是浏览器的 `window` 对象，但可以显式提供对象。管理器使用 `window.location.hash` 和 `hashchange` 事件的事件监听器。`current` 访问器返回当前路径，不带 # 前缀。

##### State History

基于状态的历史管理器使用浏览器的 history API：`pushState()` 和 `replaceState()`，来添加和修改历史纪录。状态历史管理器需要服务器端支持才能有效工作。

##### Memory History

`MemoryHistory` 不依赖任何浏览器 API，而是保持其自身的内部路径状态。不要在生产应用程序中使用它，但在测试路由时却很有用。

```ts
import { Router } from '@dojo/framework/routing/Router';
import { MemoryHistory } from '@dojo/framework/routing/history/MemoryHistory';

const router = new Router(config, { HistoryManager: MemoryHistory });
```

#### Outlet Event

当每次进入或离开 outlet 时，都会触发 `router` 实例的 `outlet` 事件。outlet 上下文、`enter` 或 `exit` 操作都会传给事件处理函数。

```ts
router.on('outlet', ({ outlet, action }) => {
    if (action === 'enter') {
        if (outlet.id === 'my-outlet') {
            // do something, perhaps fetch data or set state
        }
    }
});
```

### Router Context Injection

`RouterInjector` 模块导出一个帮助函数 `registerRouterInjector`，它组合了 `Router` 实例的实例化，注册 router 配置信息和为提供的 registry 定义注入器，然后返回 `router` 实例。

```ts
import { Registry } from '@dojo/framework/widget-core/Registry';
import { registerRouterInjector } from '@dojo/framework/routing/RoutingInjector';

const registry = new Registry();
const router = registerRouterInjector(config, registry);
```

可使用 `RouterInjectiorOptions` 覆盖默认值：

```ts
import { Registry } from '@dojo/framework/widget-core/Registry';
import { registerRouterInjector } from '@dojo/framework/routing/RoutingInjector';
import { MemoryHistory } from './history/MemoryHistory';

const registry = new Registry();
const history = new MemoryHistory();

const router = registerRouterInjector(config, registry, { history, key: 'custom-router-key' });
```

### Outlets

路由集成的一个基本概念是 `outlet`，它是与注册的应用程序 route 关联的唯一标识符。`Outlet` 是一个标准的 dojo 部件，可在应用程序的任何地方使用。`Outlet` 部件有一个精简的 API：

- `id`: 匹配时执行 `renderer` 的 outlet 标识。
- `renderer`: 当 outlet 匹配时调用的渲染函数。
- `routerKey` (可选): 在 registry 中定义路由时使用的 `key` - 默认为 `router`。

接收渲染的 outlet 名称和一个 `renderer` 函数，当 outlet 匹配时，该函数返回要渲染的 `DNode`。

```ts
render() {
    return v('div', [
        w(Outlet, { id: 'my-outlet', renderer: () => {
            return w(MyWidget, {});
        }})
    ])
}
```

可为 `renderer` 函数传入 `MatchDetails` 参数，该参数提供路由专有信息，用于确定要渲染的内容和计算传入部件的属性值。

```ts
interface MatchDetails {
    /**
     * Query params from the matching route for the outlet
     */
    queryParams: Params;

    /**
     * Params from the matching route for the outlet
     */
    params: Params;

    /**
     * Match type of the route for the outlet, either `index`, `partial` or `error`
     */
    type: MatchType;

    /**
     * The router instance
     */
    router: RouterInterface;

    /**
     * Function returns true if the outlet match was an `error` type
     */
    isError(): boolean;

    /**
     * Function returns true if the outlet match was an `index` type
     */
    isExact(): boolean;
}
```

```ts
render() {
    return v('div', [
        w(Outlet, { id: 'my-outlet', renderer: (matchDetails: MatchDetails) => {
            if (matchDetails.isError()) {
                return w(ErrorWidget, {});
            }
            if (matchDetails.isExact()) {
                return w(IndexWidget, { id: matchDetails.params.id });
            }
            return w(OtherWidget, { id: matchDetails.params.id });
        }})
    ])
}
```

#### Global Error Outlet

只要注册了一个 `error` 匹配类型，就会自动为匹配的 outlet 添加一个全局 outlet，名为 `errorOutlet`。这个 outlet 用于为任何未知的路由渲染一个部件。

```ts
render() {
    return w(Outlet, {
        id: 'errorOutlet',
        renderer: (matchDetails: MatchDetails) => {
            return w(ErrorWidget, properties);
        }
    });
}
```

### Link

`Link` 组件是对 DOM 节点 `a` 的封装，允许用户为创建的链接指定一个 `outlet`。也可以通过将 `isOutlet` 属性值设置为 `false` 来使用静态路由。

如果生成的链接需要指定在 route 中不存在的路径或查询参数，可使用 `params` 属性传入。

```ts
import { Link } from '@dojo/framework/routing/Link';

render() {
    return v('div', [
        w(Link, { to: 'foo', params: { foo: 'bar' }}, [ 'Link Text' ]),
        w(Link, { to: '#/static-route', isOutlet: false, [ 'Other Link Text' ])
    ]);
}
```

所有标准的 `VNodeProperties` 都可用于 `Link` 组件，因为最终是使用 `@dojo/framework/widget-core` 中的 `v()` 来创建一个 `a` 元素。

### ActiveLink

`ActiveLink` 组件是对 `Link` 组件的封装，如果链接处于激活状态，则可以为 `a` 节点设置样式类。

```ts
import { ActiveLink } from '@dojo/framework/routing/ActiveLink';

render() {
    return v('div', [
        w(ActiveLink, { to: 'foo', params: { foo: 'bar' }, activeClasses: [ 'link-active' ]}, [ 'Link Text' ])
    ]);
}
```
