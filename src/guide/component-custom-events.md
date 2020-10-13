# Custom Events(自定义事件)
> This page assumes you've already read the [Components Basics](component-basics.md).
>本页假设您已经阅读了[Components Basics](component-basic .md)。
Read that first if you are new to components.
如果您对组件不熟悉，请先阅读它。
## Event Names
# #事件名称
Unlike components and props, event names don't provide any automatic case transformation.
与组件和道具不同，事件名不提供任何自动案例转换。
Instead, the name of an emitted event must exactly match the name used to listen to that event.
相反，所发出事件的名称必须与用于侦听该事件的名称完全匹配。
For example, if emitting a camelCased event name:
例如，如果发送驼峰大小写事件名:

```js
this.$emit('myEvent')
```
Listening to the kebab-cased version will have no effect:
监听(短横线隔开式) 命名的版本将没有任何效果:

```html
<!-- Won't work -->
<my-component @my-event="doSomething"></my-component>
```

Since event names will never be used as variable or property names in JavaScript, there is no reason to use camelCase or PascalCase.
因为在JavaScript中事件名永远不会用作变量名或属性名，所以没有理由使用camelCase或PascalCase。
Additionally, `v-on` event listeners inside DOM templates will be automatically transformed to lowercase (due to HTML's case-insensitivity), so `@myEvent` would become `@myevent` -- making `myEvent` impossible to listen to
此外，DOM模板中的“v-on”事件监听器将自动转换为小写(由于HTML的大小写不敏感)，因此“@myEvent”将变成“@myEvent”——使“myEvent”不可能被监听.

For these reasons, we recommend you **always use kebab-case for event names**.
由于这些原因，我们建议您**始终对事件名称使用(短横线隔开式)**。
## Defining Custom Events

<VideoLesson href="https://vueschool.io/lessons/defining-custom-events-emits?friend=vuejs" title="Learn how to define which events a component can emit with Vue School">Watch a free video about Defining Custom Events on Vue School</VideoLesson>

Emitted events can be defined on the component via the `emits` option.
可以通过“emit”选项在组件上定义所发出的事件。

```js
app.component('custom-form', {
  emits: ['in-focus', 'submit']
})
```

When a native event (e.g., `click`) is defined in the `emits` option, the component event will be used __instead__ of a native event listener.
当原生事件(例如“click”)在“emit”选项中定义时，组件事件将被剩余使用，而不是原生事件侦听器。
::: tip
:::提示
It is recommended to define all emitted events in order to better document how a component should work.
建议定义所有发出的事件，以便更好地记录组件应该如何工作。
:::
:::
### Validate Emitted Events
###验证发出的事件
Similar to prop type validation, an emitted event can be validated if it is defined with the Object syntax instead of the Array syntax.
与prop类型验证类似，如果使用对象语法而不是数组语法定义发出的事件，则可以对其进行验证。
To add validation, the event is assigned a function that receives the arguments passed to the `$emit` call and returns a boolean to indicate whether the event is valid or not.
要添加验证，将为事件分配一个函数，该函数接收传递给' $emit '调用的参数，并返回一个布尔值来指示事件是否有效。

```js
app.component('custom-form', {
  emits: {
    // No validation
    click: null,

    // Validate submit event
    submit: ({ email, password }) => {
      if (email && password) {
        return true
      } else {
        console.warn('Invalid submit event payload!')
        return false
      }
    }
  },
  methods: {
    submitForm() {
      this.$emit('submit', { email, password })
    }
  }
})
```

## `v-model` arguments

By default, `v-model` on a component uses `modelValue` as the prop and `update:modelValue` as the event. We can modify these names passing an argument to `v-model`:

```html
<my-component v-model:title="bookTitle"></my-component>
```

In this case, child component will expect a `title` prop and emits `update:title` event to sync:

```js
const app = Vue.createApp({})

app.component('my-component', {
  props: {
    title: String
  },
  template: `
    <input 
      type="text"
      :value="title"
      @input="$emit('update:title', $event.target.value)">
  `
})
```

```html
<my-component v-model:title="bookTitle"></my-component>
```

## Multiple `v-model` bindings

By leveraging the ability to target a particular prop and event as we learned before with [`v-model` arguments](#v-model-arguments), we can now create multiple v-model bindings on a single component instance.

Each v-model will sync to a different prop, without the need for extra options in the component:

```html
<user-name
  v-model:first-name="firstName"
  v-model:last-name="lastName"
></user-name>
```

```js
const app = Vue.createApp({})

app.component('user-name', {
  props: {
    firstName: String,
    lastName: String
  },
  template: `
    <input 
      type="text"
      :value="firstName"
      @input="$emit('update:firstName', $event.target.value)">

    <input
      type="text"
      :value="lastName"
      @input="$emit('update:lastName', $event.target.value)">
  `
})
```

<p class="codepen" data-height="300" data-theme-id="39028" data-default-tab="html,result" data-user="Vue" data-slug-hash="GRoPPrM" data-preview="true" data-editable="true" style="height: 300px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;" data-pen-title="Multiple v-models">
  <span>See the Pen <a href="https://codepen.io/team/Vue/pen/GRoPPrM">
  Multiple v-models</a> by Vue (<a href="https://codepen.io/Vue">@Vue</a>)
  on <a href="https://codepen.io">CodePen</a>.</span>
</p>
<script async src="https://static.codepen.io/assets/embed/ei.js"></script>

## Handling `v-model` modifiers

When we were learning about form input bindings, we saw that `v-model` has [built-in modifiers](/guide/forms.html#modifiers) - `.trim`, `.number` and `.lazy`. In some cases, however, you might also want to add your own custom modifiers.

Let's create an example custom modifier, `capitalize`, that capitalizes the first letter of the string provided by the `v-model` binding.

Modifiers added to a component `v-model` will be provided to the component via the `modelModifiers` prop. In the below example, we have created a component that contains a `modelModifiers` prop that defaults to an empty object.

Notice that when the component's `created` lifecycle hook triggers, the `modelModifiers` prop contains `capitalize` and its value is `true` - due to it being set on the `v-model` binding `v-model.capitalize="bar"`.

```html
<my-component v-model.capitalize="bar"></my-component>
```

```js
app.component('my-component', {
  props: {
    modelValue: String,
    modelModifiers: {
      default: () => ({})
    }
  },
  template: `
    <input type="text" 
      :value="modelValue"
      @input="$emit('update:modelValue', $event.target.value)">
  `,
  created() {
    console.log(this.modelModifiers) // { capitalize: true }
  }
})
```

Now that we have our prop set up, we can check the `modelModifiers` object keys and write a handler to change the emitted value. In the code below we will capitalize the string whenever the `<input />` element fires an `input` event.

```html
<div id="app">
  <my-component v-model.capitalize="myText"></my-component>
  {{ myText }}
</div>
```

```js
const app = Vue.createApp({
  data() {
    return {
      myText: ''
    }
  }
})

app.component('my-component', {
  props: {
    modelValue: String,
    modelModifiers: {
      default: () => ({})
    }
  },
  methods: {
    emitValue(e) {
      let value = e.target.value
      if (this.modelModifiers.capitalize) {
        value = value.charAt(0).toUpperCase() + value.slice(1)
      }
      this.$emit('update:modelValue', value)
    }
  },
  template: `<input
    type="text"
    :value="modelValue"
    @input="emitValue">`
})

app.mount('#app')
```

For `v-model` bindings with arguments, the generated prop name will be `arg + "Modifiers"`:

```html
<my-component v-model:foo.capitalize="bar"></my-component>
```

```js
app.component('my-component', {
  props: ['foo', 'fooModifiers'],
  template: `
    <input type="text" 
      :value="foo"
      @input="$emit('update:foo', $event.target.value)">
  `,
  created() {
    console.log(this.fooModifiers) // { capitalize: true }
  }
})
```
