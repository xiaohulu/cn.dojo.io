---
layout: tutorial
title: Form validation
icon: exclamation-circle
overview: This tutorial covers patterns for form validation, building on both the form widget tutorial and the state management tutorial.
paginate: true
topic: forms
---

> [tutorials/1015_form_validation/index.md](https://github.com/dojo/dojo.io/blob/master/site/source/tutorials/1015_form_validation/index.md)
>
> commit 3e0f3ff1ed392163bc65e9cd015c4705cb9c586e

{% section 'first' %}

# 表单校验

## Overview

本教程将介绍如何在示例应用程序的上下文中处理基本的表单校验。在 [注入状态](../1010_containers_and_injecting_state) 教程中，我们已经介绍了处理表单数据；我们将在这些概念的基础上，在现有表单上添加校验状态和错误信息。本教程中，我们将构建一个支持动态的客户端校验和模拟的服务器端校验示例。

## 前提

你可以打开 [codesandbox.io 上的教程](https://codesandbox.io/s/github/dojo/dojo.io/tree/master/site/source/tutorials/1015_form_validation/demo/initial/biz-e-corp) 或者 [下载](../assets/1015_form_validation-initial.zip) 示例项目，然后运行 `npm install`。

本教程假设你已经学习了 [表单部件教程](../005_form_widgets) 和 [状态管理教程](../1010_containers_and_injecting_state)。

{% section %}

## 创建存储表单错误的对象

{% task '在应用程序上下文中添加表单错误。' %}

现在，错误对象应该对应存在于 `WorkerForm.ts` 和 `ApplicationContext.ts` 文件中的 `WorkerFormData`。这种错误配置有多种处理方式，一种情况是为单个 input 的多个校验步骤分别设置错误信息。现在我们将从最简单的情况开始，即为每个 input 添加布尔类型的 valid 和 invalid 状态。

{% instruction '为 `WorkerForm.ts` 文件中创建一个 `WorkerFormErrors` 接口' %}

{% include_codefile 'demo/finished/biz-e-corp/src/widgets/WorkerForm.ts' lines:15-19 %}

```ts
export interface WorkerFormErrors {
    firstName?: boolean;
    lastName?: boolean;
    email?: boolean;
}
```

将 `WorkerFormErrors` 中的属性定义为可选，这样我们就可以为 form 中的字段创建三种状态：未校验的、有效的和无效的。

{% instruction '接下来将 `formErrors` 方法添加到 `ApplicationContext` 类中' %}

在练习中，完成以下三步：

1. 在 ApplicationContext 类中创建一个私有字段 `_formErrors`
2. 在 `ApplicationContext` 中为 `_formErrors` 创建一个 public 访问器
3. 更新 `WorkerFormContainer.ts` 文件中的 `getProperties` 函数，支持传入新的错误对象

提示：查看 `ApplicationContext` 类中已有的 `_formData` 私有字段是如何使用的。可按照相同的流程添加 `_formErrors` 变量。

确保 `ApplicationContext.ts` 中存在以下代码：

```ts
// modify import to include WorkerFormErrors
import { WorkerFormData, WorkerFormErrors } from './widgets/WorkerForm';

// private field
private _formErrors: WorkerFormErrors = {};

// public getter
get formErrors(): WorkerFormErrors {
    return this._formErrors;
}
```

`WorkerFormContainer.ts` 中修改后的 `getProperties` 函数：

```ts
function getProperties(inject: ApplicationContext, properties: any) {
    const {
        formData,
        formErrors,
        formInput: onFormInput,
        submitForm: onFormSave
    } = inject;

    return {
        formData,
        formErrors,
        onFormInput: onFormInput.bind(inject),
        onFormSave: onFormSave.bind(inject)
    };
}
```

{% instruction '最后，修改 `WorkerForm.ts` 中的 `WorkerFormProperties` 来接收应用程序上下文传入的 `formErrors` 对象：' %}

```ts
export interface WorkerFormProperties {
    formData: WorkerFormData;
    formErrors: WorkerFormErrors;
    onFormInput: (data: Partial<WorkerFormData>) => void;
    onFormSave: () => void;
}
```

{% section %}

## 为 form 表单输入框绑定校验

{% task '在 `onInput` 中执行校验' %}

现在，我们已经可以在应用程序状态中存储表单错误，并将这些错误传给 form 表单部件。但 form 表单依然缺少真正的用户输入校验；为此，我们需要温习正则表达式并写一个基本的校验函数。

{% instruction '在 `ApplicationContext.ts` 中创建一个私有方法 `_validateInput`' %}

跟已存在的 `formInput` 函数相似，应该为 `_validateInput` 传入 Partial 类型的 `WorkerFormData` 输入对象。校验函数应该返回一个 `WorkerFormErrors` 对象。示例应用程序中只展示了最基本的校验检查——示例中邮箱地址的正则表达式模式匹配简洁但有不够完备。你可以用更健壮的邮箱测试来代替，或者做其它修改，如检查第一个名字和最后一个名字的最小字符数。

{% include_codefile 'demo/finished/biz-e-corp/src/ApplicationContext.ts' lines:32-50 %}

```ts
private _validateInput(input: Partial<WorkerFormData>): WorkerFormErrors {
    const errors: WorkerFormErrors = {};

    // validate input
    for (let key in input) {
        switch (key) {
            case 'firstName':
                errors.firstName = !input.firstName;
                break;
            case 'lastName':
                errors.lastName = !input.lastName;
                break;
            case 'email':
                errors.email = !input.email || !input.email.match(/^[a-zA-Z0-9.!#$%&’*+/=?^_`{|}~-]+@[a-zA-Z0-9-]+(?:\.[a-zA-Z0-9-]+)*$/);
        }
    }

    return errors;
}
```

现在，我们将在每一个 `onInput` 事件中直接调用校验函数来测试它。将下面一行代码添加到 `ApplicationContext.ts` 中的 `formInput` 中：

```ts
this._formErrors = deepAssign({}, this._formErrors, this._validateInput(input));
```

{% instruction '更新 `WorkerForm` 的渲染方法来显示校验状态' %}

至此，`WorkerForm` 部件的 `formErrors` 属性中存着每个 form 字段的校验状态，每次调用 `onInput` 事件时都会更新校验状态。剩下的就是将 valid/invalid 属性传给所有输入部件。幸运的是，Dojo 的 `TextInput` 部件包含一个 `invalid` 属性，可用于在 DOM 节点上设置 `aria-invalid` 属性，并切换可视化样式类。

`WorkerForm.ts` 中更新后的渲染方法，应该是将每个 form 字段部件的上 `invalid` 属性与 `formErrors` 对应上。我们也为 form 元素添加了一个 `novalidate` 属性来禁用原生浏览器校验。

```ts
protected render() {
    const {
        formData: { firstName, lastName, email },
        formErrors
    } = this.properties;

    return v('form', {
        classes: this.theme(css.workerForm),
        novalidate: 'true',
        onsubmit: this._onSubmit
    }, [
        v('fieldset', { classes: this.theme(css.nameField) }, [
            v('legend', { classes: this.theme(css.nameLabel) }, [ 'Name' ]),
            w(TextInput, {
                key: 'firstNameInput',
                label:'First Name',
                labelHidden: true,
                placeholder: 'Given name',
                value: firstName,
                required: true,
                invalid: this.properties.formErrors.firstName,
                onInput: this.onFirstNameInput
            }),
            w(TextInput, {
                key: 'lastNameInput',
                label: 'Last Name',
                labelHidden: true,
                placeholder: 'Surname name',
                value: lastName,
                required: true,
                invalid: this.properties.formErrors.lastName,
                onInput: this.onLastNameInput
            })
        ]),
        w(TextInput, {
            label: 'Email address',
            type: 'email',
            value: email,
            required: true,
            invalid: this.properties.formErrors.email,
            onInput: this.onEmailInput
        }),
        w(Button, {}, [ 'Save' ])
    ]);
}
```

现在，当你在浏览器中查看应用程序时，每个表单字段的边框颜色会随着你键入的内容而变化。接下来我们将添加错误信息，并更新 `onInput` ，让检验只在第一次失去焦点（blur）事件后发生。

{% section %}

## 扩展 TextInput

{% task '创建一个错误消息' %}

简单的将 form 字段的边框颜色设置为红色或绿色并不能告知用户更多信息——我们需要为无效状态添加一些错误消息文本。最基本要求，我们的错误文本必须与 form 中的 input 关联，可设置样式和可访问。一个包含错误信息的 form 表单字段看起来应该是这样的：

```ts
v('div', { classes: this.theme(css.inputWrapper) }, [
    w(TextInput, {
        ...
        aria: {
            describedBy: this._errorId
        },
        onInput: this._onInput
    }),
    invalid === true ? v('span', {
        id: this._errorId,
        classes: this.theme(css.error),
        'aria-live': 'polite'
    }, [ 'Please enter valid text for this field' ]) : null
])
```

通过 `aria-describeby` 属性将错误消息与文本输入框关联，并使用 `aria-live` 属性来确保当它添加到 DOM 或发生变化后能被读取到。将输入框和错误信息包裹在一个 `<div>` 中，则在需要时可相对输入框来获取到错误信息的位置。

{% instruction '扩展 `TextInput`，创建一个包含错误信息和 `onValidate` 方法的 `ValidatedTextInput` 部件' %}

为多个文本输入框重复创建相同的错误消息样板明显是十分啰嗦的，所以我们将扩展 `TextInput`。这还将让我们能够更好的控制何时校验，例如，也可以添加给 blur 事件。现在，只是创建一个 `ValidatedTextInput` 部件，它接收与 `TextInput` 相同的属性接口，但多了一个 `errorMessage` 字符串和 `onValidate` 方法。它应该返回与上面相同的节点结构。

你也需要创建包含 `error` 和 `inputWrapper` 样式类的 `validatedTextInput.m.css` 文件，尽管我们会弃用本教程中添加的特定样式：

```css
.inputWrapper {}

.error {}
```

```ts
import { WidgetBase } from '@dojo/framework/widget-core/WidgetBase';
import { TypedTargetEvent } from '@dojo/framework/widget-core/interfaces';
import { v, w } from '@dojo/framework/widget-core/d';
import uuid from '@dojo/framework/core/uuid';
import { ThemedMixin, theme } from '@dojo/framework/widget-core/mixins/Themed';
import TextInput, { TextInputProperties } from '@dojo/widgets/text-input';
import * as css from '../styles/validatedTextInput.m.css';

export interface ValidatedTextInputProperties extends TextInputProperties {
    errorMessage?: string;
    onValidate?: (value: string) => void;
}

export const ValidatedTextInputBase = ThemedMixin(WidgetBase);

@theme(css)
export default class ValidatedTextInput extends ValidatedTextInputBase<ValidatedTextInputProperties> {
    private _errorId = uuid();

    protected render() {
        const {
            disabled,
            label,
            maxLength,
            minLength,
            name,
            placeholder,
            readOnly,
            required,
            type = 'text',
            value,
            invalid,
            errorMessage,
            onBlur,
            onInput
        } = this.properties;

        return v('div', { classes: this.theme(css.inputWrapper) }, [
            w(TextInput, {
                aria: {
                    describedBy: this._errorId
                },
                disabled,
                invalid,
                label,
                maxLength,
                minLength,
                name,
                placeholder,
                readOnly,
                required,
                type,
                value,
                onBlur,
                onInput
            }),
            invalid === true ? v('span', {
                id: this._errorId,
                classes: this.theme(css.error),
                'aria-live': 'polite'
            }, [ errorMessage ]) : null
        ]);
    }
}
```

你可能已注意到，我们创建的 `ValidatedTextInput` 包含一个 `onValidate` 属性，但我们还没有用到它。在接下来的几步中，这将变得非常重要，因为我们可以对何时校验做更多的控制。现在，只是把它当做一个占位符。

{% instruction '在 `WorkerForm` 中使用 `ValidatedTextInput`' %}

现在 `ValidatedTextInput` 已存在，让我们在 `WorkerForm` 中导入它并替换掉 `TextInput`，并在其中写一些错误消息文本：

**Import 语句块**

{% include_codefile 'demo/finished/biz-e-corp/src/widgets/WorkerForm.ts' lines:1-7 %}

```ts
import { WidgetBase } from '@dojo/framework/widget-core/WidgetBase';
import { TypedTargetEvent } from '@dojo/framework/widget-core/interfaces';
import { v, w } from '@dojo/framework/widget-core/d';
import { ThemedMixin, theme } from '@dojo/framework/widget-core/mixins/Themed';
import Button from '@dojo/widgets/button';
import ValidatedTextInput from './ValidatedTextInput';
import * as css from '../styles/workerForm.m.css';
```

**render() 方法内部**

{% include_codefile 'demo/finished/biz-e-corp/src/widgets/WorkerForm.ts' lines:72-108 %}

```ts
v('fieldset', { classes: this.theme(css.nameField) }, [
    v('legend', { classes: this.theme(css.nameLabel) }, [ 'Name' ]),
    w(ValidatedTextInput, {
        key: 'firstNameInput',
        label: 'First Name',
        labelHidden: true,
        placeholder: 'Given name',
        value: firstName,
        required: true,
        onInput: this.onFirstNameInput,
        onValidate: this.onFirstNameValidate,
        invalid: formErrors.firstName,
        errorMessage: 'First name is required'
    }),
    w(ValidatedTextInput, {
        key: 'lastNameInput',
        label: 'Last Name',
        labelHidden: true,
        placeholder: 'Surname name',
        value: lastName,
        required: true,
        onInput: this.onLastNameInput,
        onValidate: this.onLastNameValidate,
        invalid: formErrors.lastName,
        errorMessage: 'Last name is required'
    })
]),
w(ValidatedTextInput, {
    label: 'Email address',
    type: 'email',
    value: email,
    required: true,
    onInput: this.onEmailInput,
    onValidate: this.onEmailValidate,
    invalid: formErrors.email,
    errorMessage: 'Please enter a valid email address'
}),
```

{% task '创建从 `onFormInput` 中提取出来的  `onFormValidate` 方法' %}

{% instruction '传入 `onFormValidate` 方法来更新上下文' %}

现在校验逻辑毫不客气的躺在 `ApplicationContext.ts` 中的 `formInput` 中。现在我们将它抬到自己的 `formValidate` 函数中，并参考 `onFormInput` 模式，将 `onFormValidate` 传给 `WorkerForm`。这里有三个步骤：

1. 在 `ApplicationContext.ts` 中添加 `formValidate` 方法，并将 `formInput` 中更新 `_formErrors` 代码放到 `formValidate` 中：

    ```ts
    public formValidate(input: Partial<WorkerFormData>): void {
        this._formErrors = deepAssign({}, this._formErrors, this._validateInput(input));
        this._invalidator();
    }

    public formInput(input: Partial<WorkerFormData>): void {
        this._formData = deepAssign({}, this._formData, input);
        this._invalidator();
    }
    ```

2. 更新 `WorkerFormContainer`，将 `formValidate` 传给 `onFormValidate`：

    ```ts
    function getProperties(inject: ApplicationContext, properties: any) {
        const {
            formData,
            formErrors,
            formInput: onFormInput,
            formValidate: onFormValidate,
            submitForm: onFormSave
        } = inject;

        return {
            formData,
            formErrors,
            onFormInput: onFormInput.bind(inject),
            onFormValidate: onFormValidate.bind(inject),
            onFormSave: onFormSave.bind(inject)
        };
    }
    ```

3. 在 `WorkerForm` 中先在 `WorkerFormProperties` 接口中添加 `onFormValidate`:

    ```ts
    export interface WorkerFormProperties {
        formData: WorkerFormData;
        formErrors: WorkerFormErrors;
        onFormInput: (data: Partial<WorkerFormData>) => void;
        onFormValidate: (data: Partial<WorkerFormData>) => void;
        onFormSave: () => void;
    }
    ```

    然后为每个 form 字段的校验创建内部方法，并将这些方法（如 `onFirstNameValidate`）传给每个 `ValidatedTextInput` 部件。这将使用与 `onFormInput`、`onFirstNameInput`、`onLastNameInput` 和 `onEmailInput` 相同的模式：

    ```ts
    protected onFirstNameValidate(firstName: string) {
        this.properties.onFormValidate({ firstName });
    }

    protected onLastNameValidate(lastName: string) {
        this.properties.onFormValidate({ lastName });
    }

    protected onEmailValidate(email: string) {
        this.properties.onFormValidate({ email });
    }
    ```

{% instruction '在 `ValidatedTextInput` 中调用 `onValidate`' %}

你可能已注意到，当用户输入事件发生后，form 表单不再校验。这是因为我们已不在 `ApplicationContext.ts` 的 `formInput` 中处理校验，但我们还没有将校验添加到其它地方。要做到这一点，我们在 `ValidateTextInput` 中添加以下私有方法：

```ts
private _onInput(value: string) {
    const { onInput, onValidate } = this.properties;
    onInput && onInput(value);
    onValidate && onValidate(value);
}
```

现在将它传给 `TextInput`，替换掉 `this.properties.onInput`：

```ts
w(TextInput, {
    aria: {
        describedBy: this._errorId
    },
    disabled,
    invalid,
    label,
    maxLength,
    minLength,
    name,
    placeholder,
    readOnly,
    required,
    type,
    value,
    onBlur,
    onInput: this._onInput
})
```

表单错误功能已恢复，并为无效字段添加了错误消息。

{% section %}

## 使用 blur 事件

{% task '仅在第一次 blur 事件后开始校验' %}

现在只要用户开始在字段中输入就会显示校验信息，这是一种不友好的用户体验。在用户开始输入邮箱地址时就看到 “invalid email address” 是没有必要的，也容易分散注意力。更好的模式是将校验推迟到第一次 blur 事件之后，然后在 input 事件中开始更新校验信息。

{% aside 'Blur 事件' %}
当元素失去焦点后会触发 [blur](https://developer.mozilla.org/en-US/docs/Web/Events/blur) 事件。
{% endaside %}

现在已在 `ValidatedTextInput` 部件中调用了 `onValidate`，这是可以实现的。

{% instruction '创建一个私有的 `_onBlur` 函数，它会调用 `onValidate`' %}

在 `ValidatedTextInput.ts` 文件中:

```ts
private _onBlur(value: string) {
    const { onBlur, onValidate } = this.properties;
    onValidate && onValidate(value);
    onBlur && onBlur();
}
```

我们仅需在第一次 blur 事件之后使用这个函数，因为随后的校验交由 `onInput` 处理。下面的代码将根据输入框之前是否已校验过，来使用 `this._onBlur` 或 `this.properties.onBlur`：

{% include_codefile 'demo/finished/biz-e-corp/src/widgets/ValidatedTextInput.ts' lines:50-67 %}

```ts
w(TextInput, {
    aria: {
        describedBy: this._errorId
    },
    disabled,
    invalid,
    label,
    maxLength,
    minLength,
    name,
    placeholder,
    readOnly,
    required,
    type,
    value,
    onBlur: typeof invalid === 'undefined' ? this._onBlur : onBlur,
    onInput: this._onInput
}),
```

现在只剩下修改 `_onInput`，如果字段已经有一个校验状态，则调用 `onValidate`:

{% include_codefile 'demo/finished/biz-e-corp/src/widgets/ValidatedTextInput.ts' lines:24-31 %}

```ts
private _onInput(value: string) {
    const { invalid, onInput, onValidate } = this.properties;
    onInput && onInput(value);

    if (typeof invalid !== 'undefined') {
        onValidate && onValidate(value);
    }
}
```

尝试输入一个邮箱地址来演示这些变化；它应该只在第一次离开 form 字段之后显示错误信息（或绿色边框），而在接下来的编辑中将立即触发校验。

{% section %}

## 在提交时校验

{% task '创建一个模拟的服务器端校验，以处理提交的 form 表单' %}

到目前为止，我们的代码给用户提供了友好提示，但并不能防止我们将无效数据提交到我们的 worker 数组中。我们需要在 `submitForm` 操作中添加两个独立的检查：

1. 如果已存在的校验函数捕获到任何错误，则立即提交失败。
2. 执行额外检查（本示例中我们将检查邮箱唯一性）。这是我们在真正的应用程序中加入服务器端校验的地方。

{% instruction '在 `ApplicationContext.ts` 中创建一个私有方法 `_validateOnSubmit`' %}

新增的 `_validateOnSubmit` 方法应该从对所有 `_formData` 运行已存在的输入校验开始，然后在存在任一错误后返回 false：

```ts
private _validateOnSubmit(): boolean {
    const errors = this._validateInput(this._formData);
    this._formErrors = deepAssign({ firstName: true, lastName: true, email: true }, errors);

    if (this._formErrors.firstName || this._formErrors.lastName || this._formErrors.email) {
        console.error('Form contains errors');
        return false;
    }

    return true;
}
```

接下来我们添加一个检查：假设每个工人的邮箱必须是唯一的，所以我们将在 `_workerData` 数组中测试输入的邮箱地址是否已存在。在现实中安全起见，这个检查运行在服务器端：

{% include_codefile 'demo/finished/biz-e-corp/src/ApplicationContext.ts' lines:53-70 %}

```ts
private _validateOnSubmit(): boolean {
    const errors = this._validateInput(this._formData);
    this._formErrors = deepAssign({ firstName: true, lastName: true, email: true }, errors);

    if (this._formErrors.firstName || this._formErrors.lastName || this._formErrors.email) {
        console.error('Form contains errors');
        return false;
    }

    for (let worker of this._workerData) {
        if (worker.email === this._formData.email) {
            console.error('Email must be unique');
            return false;
        }
    }

    return true;
}

```

修改完 `ApplicationContext.ts` 中的 `submitForm` 函数后，只有有效的工人数据才能提交成功。我们也需要在成功提交后清空 `_formErrors` 和 `_formData`：

{% include_codefile 'demo/finished/biz-e-corp/src/ApplicationContext.ts' lines:82-92 %}

```ts
public submitForm(): void {
    if (!this._validateOnSubmit()) {
        this._invalidator();
        return;
    }

    this._workerData = [ ...this._workerData, this._formData ];
    this._formData = {};
    this._formErrors = {};
    this._invalidator();
}
```

{% section %}

## 总结

本教程不可能涵盖所有可能用例，但是存储、注入和显示校验状态的基本模式，为创建复杂的表单校验提供了坚实的基础。接下来将包含以下步骤：

- 为传递给 `WorkerForm` 的对象配置错误信息
- 创建一个 toast 来显示提交时的错误
- 为一个 form 字段添加多个校验步骤

你可以在 [codesandbox.io](https://codesandbox.io/s/github/dojo/dojo.io/tree/master/site/source/tutorials/1015_form_validation/demo/finished/biz-e-corp) 中打开完整示例或[下载](../assets/1015_form_validation-finished.zip)项目。

{% section 'last' %}
