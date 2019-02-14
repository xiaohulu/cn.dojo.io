# 测试

> [dojo/framework/src/testing/README.md](https://github.com/dojo/framework/blob/master/src/testing/README.md)
>
> commit 4de8ee423a58be4454adb9274b55740904442b1d

用于测试和断言 Dojo 部件期望的虚拟 DOM 和行为的简单 API。

- [测试](#%E6%B5%8B%E8%AF%95)
  - [Features](#features)
  - [harness](#harness)
    - [API](#api)
    - [Custom Comparators](#custom-comparators)
  - [selectors](#selectors)
  - [`harness.expect`](#harnessexpect)
    - [`harness.expectPartial`](#harnessexpectpartial)
      - [`harness.trigger`](#harnesstrigger)
      - [`harness.getRender`](#harnessgetrender)
  - [Assertion Templates](#assertion-templates)

## Features

- 简单、熟悉和少量的 API
- 重点测试 Dojo 虚拟 DOM 结构
- 默认不需要 DOM
- 全面支持函数式编程和 tsx 声明

## harness

当使用 `@dojo/framework/testing` 时，`harness` 是最重要的 API，主要用于设置每一个测试并提供一个执行虚拟 DOM 断言和交互的上下文。目的在于当更新 `properties` 或 `children` 和失效部件时，镜像部件的核心行为，并且不需要任何特殊或自定义逻辑。

### API

```ts
harness(renderFunction: () => WNode, customComparators?: CustomComparator[]): Harness;
```

- `renderFunction`: 返回被测部件 WNode 的函数
- [`customComparators`](custom-comparators): 一组自定义比较器的描述符。每个描述符提供一个比较器函数，用于比较通过 `selector` 和 `property` 定位的 `properties`

harness 函数返回的 `Harness` 对象提供了与被测部件交互的精简 API：

`Harness`

- [`expect`](#harnessexpect): 对被测部件完整的渲染结果执行断言
- [`expectPartial`](#harnessexpectpartial): 对被测部件的一部分渲染结果执行断言
- [`trigger`](#harnesstrigger): 用于在被测部件的节点上触发函数
- [`getRender`](#harnessgetRender): 根据提供的索引返回对应的渲染器

使用 `@dojo/framework/widget-core` 中的 `w()` 函数设置一个待测试的部件简单且眼熟：

```ts
class MyWidget extends WidgetBase<{ foo: string }> {
    protected render() {
        const { foo } = this.properties;
        return v('div', { foo }, this.children);
    }
}

const h = harness(() => w(MyWidget, { foo: 'bar' }, ['child']));
```

如下所示，harness 函数也支持 `tsx` 语法。README 文档中其余示例使用编程式的 `w()` API，在 [unit tests](./blob/master/tests/unit/harnessWithTsx.tsx) 中可查看更多 `tsx` 示例。

```ts
const h = harness(() => <MyWidget foo="bar">child</MyWidget>);
```

`renderFunction` 是延迟执行的，所以可在断言之间包含额外的逻辑来操作部件的 `properties` 和 `children`。

```ts
let foo = 'bar';

const h = harness(() => {
    return w(MyWidget, { foo }, [ 'child' ]));
};

h.expect(/** assertion that includes bar **/);
// update the property that is passed to the widget
foo = 'foo';
h.expect(/** assertion that includes foo **/)
```

### Custom Comparators

在某些情况下，我们在测试期间无法得知属性的确切值，所以需要使用自定义比较描述符。

描述符中有一个定位要检查的虚拟节点的 [`selector`](./path/to/selector)，一个应用自定义比较的属性名和一个接收实际值并返回一个 boolean 类型断言结果的比较器函数。

```ts
const compareId = {
    selector: '*', // all nodes
    property: 'id',
    comparator: (value: any) => typeof value === 'string' // checks the property value is a string
};

const h = harness(() => w(MyWidget, {}), [compareId]);
```

对于所有的断言，返回的 `harness` API 将只对 `id` 属性使用 `comparator` 进行测试，而不是标准的相等测试。

## selectors

`harness` API 支持 CSS style 选择器概念，来定位要断言和操作的虚拟 DOM 中的节点。查看 [支持的选择器的完整列表](https://github.com/fb55/css-select#supported-selectors) 以了解更多信息。

除了标准 API 之外还提供：

- 支持将定位节点 `key` 属性简写为 `@` 符号
- 当使用标准的 `.` 来定位样式类时，使用 `classes` 属性而不是 `class` 属性

## `harness.expect`

测试中最常见的需求是断言部件的 `render` 函数的输出结构。`expect` 接收一个返回被测部件期望的渲染结果的函数作为参数。

API

```ts
expect(expectedRenderFunction: () => DNode | DNode[], actualRenderFunction?: () => DNode | DNode[]);
```

- `expectedRenderFunction`: 返回查询节点期望的 `DNode` 结构的函数
- `actualRenderFunction`: 一个可选函数，返回被断言的实际 `DNode` 结构

```ts
h.expect(() =>
    v('div', { key: 'foo' }, [w(Widget, { key: 'child-widget' }), 'text node', v('span', { classes: ['class'] })])
);
```

`expect` 也可以接收第二个可选参数，返回要断言的渲染结果的函数。

```ts
h.expect(() => v('div', { key: 'foo' }), () => v('div', { key: 'foo' }));
```

如果实际的渲染输出和期望的渲染输出不同，就会抛出一个异常，并使用结构化的可视方法，用 `(A)` （实际值）和 `(E)` （期望值）指出所有不同点。

断言的错误输出示例:

```ts
v("div", {
    "classes": [
        "root",
(A)     "other"
(E)     "another"
    ],
    "onclick": "function"
}, [
    v("span", {
        "classes": "span",
        "id": "random-id",
        "key": "label",
        "onclick": "function",
        "style": "width: 100px"
    }, [
        "hello 0"
    ])
    w(ChildWidget, {
        "id": "random-id",
        "key": "widget"
    })
    w("registry-item", {
        "id": true,
        "key": "registry"
    })
])
```

### `harness.expectPartial`

`expectPartial` 根据 [`selector`](#selectors) 断言部件的部分渲染输出。

API

```ts
expectPartial(selector: string, expectedRenderFunction: () => DNode | DNode[]);
```

- `selector`: 用于查找目标节点的选择器
- `expectedRenderFunction`: 返回查询节点期望的 `DNode` 结构的函数
- `actualRenderFunction`: 一个可选函数，返回被断言的实际 `DNode` 结构

示例:

```ts
h.expectPartial('@child-widget', () => w(Widget, { key: 'child-widget' }));
```

#### `harness.trigger`

`harness.trigger()` 在 `selector` 定位的节点上调用 `name` 指定的函数。

```ts
interface FunctionalSelector {
    (node: VNode | WNode): undefined | Function;
}

trigger(selector: string, functionSelector: string | FunctionalSelector, ...args: any[]): any;
```

- `selector`: 用于查找目标节点的选择器
- `functionSelector`: 要么是从节点的属性中找到的被调用的函数名，或者是从节点的属性中返回一个函数的函数选择器
- `args`: 为定位到的函数传入的参数

如果有返回结果，则返回的是被触发函数的结果。

用法示例:

```ts
// calls the `onclick` function on the first node with a key of `foo`
h.trigger('@foo', 'onclick');
```

```ts
// calls the `customFunction` function on the first node with a key of `bar` with an argument of `100`
// and receives the result of the triggered function
const result = h.trigger('@bar', 'customFunction', 100);
```

`functionalSelector` 返回部件属性中的函数。函数也会被触发，与使用普通字符串 `functionSelector` 的方式相同。

用法示例：

假定有如下 VDOM 树结构，

```ts
v(Toolbar, {
    key: 'toolbar',
    buttons: [
        {
            icon: 'save',
            onClick: () => this._onSave()
        },
        {
            icon: 'cancel',
            onClick: () => this._onCancel()
        }
    ]
});
```

并且你想触发 save 按钮的 `onClick` 函数。

```ts
h.trigger('@buttons', (renderResult: DNode<Toolbar>) => {
    return renderResult.properties.buttons[0].onClick;
});
```

#### `harness.getRender`

`harness.getRender()` 返回索引指定的渲染器，如果没有提供索引则返回最后一个渲染器。

```ts
getRender(index?: number);
```

- `index`: 要返回的渲染器的索引

用法示例:

```ts
// Returns the result of the last render
const render = h.getRender();
```

```ts
// Returns the result of the render for the index provided
h.getRender(1);
```

## Assertion Templates

断言模板（assertion template）允许你构建一个传入 `h.expect()` 的期望渲染函数。断言模板背后的思想来自经常要断言整个渲染输出，并需要修改断言的某些部分。

要使用断言模板，首先需要导入模块：

```ts
import assertionTemplate from '@dojo/framework/testing/assertionTemplate';
```

然后，在你的测试中，你可以编写一个基本断言，它是部件的默认渲染状态：

假定有以下部件：

```ts
class NumberWidget extends WidgetBase<{ num?: number }> {
    protected render() {
        const { num } = this.properties;
        const message = num === undefined ? 'no number passed' : `the number ${num}`;
        return v('div', [v('span', [message])]);
    }
}
```

基本断言如下所示：

```ts
const baseAssertion = assertionTemplate(() => {
    return v('div', [
        v('span', { '~key': 'message' }, [ 'no number passed' ]);
    ]);
});
```

在测试中这样写：

```ts
it('should render no number passed when no number is passed as a property', () => {
    const h = harness(() => w(NumberWidget, {}));
    h.expect(baseAssertion);
});
```

现在我们看看，为 `NumberWidget` 部件传入 `num` 属性后，如何测试数据结果：

```ts
it('should render the number when a number is passed as a property', () => {
    const numberAssertion = baseAssertion.setChildren('~message', ['the number 5']);
    const h = harness(() => w(NumberWidget, { num: 5 }));
    h.expect(numberAssertion);
});
```

这里，我们使用 baseAssertion 的 `setChildren()` api，然后我们使用特殊的 `~` 选择器来定位 key 值为 `~message` 的节点。`~key` 属性（使用 tsx 的模板中是 `assertion-key`）是断言模板的一个特殊属性，在断言时会被删除，因此在匹配渲染结构时不会显示出来。此功能允许你修饰断言模板，以便能简单的选择节点，而不需要扩展实际的部件渲染函数。一旦我们获取到 `message` 节点，我们就可以将其子节点设置为期望的 `the number 5`，然后在 `h.expect` 中使用生成的模板。需要注意的是，断言模板在设置值时总是返回一个新的断言模板，这可以确保你不会意外修改现有模板（可能导致其他测试失败），并允许你基于新模板，增量逐层构建出新的模板。

断言模板具有以下 API：

```ts
insertBefore(selector: string, children: DNode[]): AssertionTemplateResult;
insertAfter(selector: string, children: DNode[]): AssertionTemplateResult;
insertSiblings(selector: string, children: DNode[], type?: 'before' | 'after'): AssertionTemplateResult;
append(selector: string, children: DNode[]): AssertionTemplateResult;
prepend(selector: string, children: DNode[]): AssertionTemplateResult;
replace(selector: string, children: DNode[]): AssertionTemplateResult;
setChildren(selector: string, children: DNode[], type?: 'prepend' | 'replace' | 'append'): AssertionTemplateResult;
setProperty(selector: string, property: string, value: any): AssertionTemplateResult;
getChildren(selector: string): DNode[];
getProperty(selector: string, property: string): any;
```
