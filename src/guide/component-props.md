# Props(属性)

> This page assumes you've already read the [Components Basics](component-basics.md). Read that first if you are new to components.

## Prop Types

So far, we've only seen props listed as an array of strings:
到目前为止，我们只看到props以字符串数组的形式列出:

```js
props: ['title', 'likes', 'isPublished', 'commentIds', 'author']
```

Usually though, you'll want every prop to be a specific type of value.
通常情况下，你会希望每个props都有特定的价值类型。
In these cases, you can list props as an object, where the properties' names and values contain the prop names and types, respectively:
在这些情况下，你可以将props作为一个对象列出，其中属性的名称和值分别包含了props的名称和类型:

```js
props: {
  title: String,
  likes: Number,
  isPublished: Boolean,
  commentIds: Array,
  author: Object,
  callback: Function,
  contactsPromise: Promise // or any other constructor
}
```

This not only documents your component, but will also warn users in the browser's JavaScript console if they pass the wrong type.
这不仅记录了您的组件，而且还会在浏览器的JavaScript控制台警告用户，如果他们传递了错误的类型。
You'll learn much more about [type checks and other prop validations](#prop-validation) further down this page.
在本页的后面，您将了解更多关于[类型检查和其他prop验证](#prop-validation)的信息。
## Passing Static or Dynamic Props
##传递静态或动态props
So far, you've seen props passed a static value, like in:
到目前为止，您已经看到了props传递静态值，如:

```html
<blog-post title="My journey with Vue"></blog-post>
```

You've also seen props assigned dynamically with `v-bind` or its shortcut, the `:` character, such as in:
你也看到了使用“v-bind”或它的快捷方式“:”字符来动态分配props，例如:

```html
<!-- Dynamically assign the value of a variable -->
<blog-post :title="post.title"></blog-post>

<!-- Dynamically assign the value of a complex expression -->
<blog-post :title="post.title + ' by ' + post.author.name"></blog-post>
```

In the two examples above, we happen to pass string values, but _any_ type of value can actually be passed to a prop.
在上面的两个示例中，我们碰巧传递了字符串值，但是_any_类型的值实际上可以传递给prop。

### Passing a Number

```html
<!-- Even though `42` is static, we need v-bind to tell Vue that -->
<!-- this is a JavaScript expression rather than a string.       -->
<blog-post :likes="42"></blog-post>

<!-- Dynamically assign to the value of a variable. -->
<blog-post :likes="post.likes"></blog-post>
```

### Passing a Boolean

```html
<!-- Including the prop with no value will imply `true`. -->
<blog-post is-published></blog-post>

<!-- Even though `false` is static, we need v-bind to tell Vue that -->
<!-- this is a JavaScript expression rather than a string.          -->
<blog-post :is-published="false"></blog-post>

<!-- Dynamically assign to the value of a variable. -->
<blog-post :is-published="post.isPublished"></blog-post>
```

### Passing an Array

```html
<!-- Even though the array is static, we need v-bind to tell Vue that -->
<!-- this is a JavaScript expression rather than a string.            -->
<blog-post :comment-ids="[234, 266, 273]"></blog-post>

<!-- Dynamically assign to the value of a variable. -->
<blog-post :comment-ids="post.commentIds"></blog-post>
```

### Passing an Object

```html
<!-- Even though the object is static, we need v-bind to tell Vue that -->
<!-- this is a JavaScript expression rather than a string.             -->
<blog-post
  :author="{
    name: 'Veronica',
    company: 'Veridian Dynamics'
  }"
></blog-post>

<!-- Dynamically assign to the value of a variable. -->
<blog-post :author="post.author"></blog-post>
```

### Passing the Properties of an Object
传递对象的属性
If you want to pass all the properties of an object as props, you can use `v-bind` without an argument (`v-bind` instead of `:prop-name`).
如果您想将一个对象的所有属性作为props传递，您可以使用“v-bind”而不带参数(“v-bind”而不是“:prop-name”)。
For example, given a `post` object:
例如，给定一个“post”对象:

```js
post: {
  id: 1,
  title: 'My Journey with Vue'
}
```

The following template:

```html
<blog-post v-bind="post"></blog-post>
```

Will be equivalent to:

```html
<blog-post v-bind:id="post.id" v-bind:title="post.title"></blog-post>
```

## One-Way Data Flow
单向数据流
All props form a **one-way-down binding** between the child property and the parent one: when the parent property updates, it will flow down to the child, but not the other way around.
所有的props都在子属性和父属性之间形成了一个**单向绑定**:当父属性更新时，它将流向子属性，而不是反过来。
This prevents child components from accidentally mutating the parent's state, which can make your app's data flow harder to understand.
这可以防止子组件意外地改变父组件的状态，这会使应用程序的数据流更难理解。
In addition, every time the parent component is updated, all props in the child component will be refreshed with the latest value.
此外，每次更新父组件时，子组件中的所有props都将使用最新值刷新。
This means you should **not** attempt to mutate a prop inside a child component.
这意味着您不应该尝试在子组件内修改prop。
If you do, Vue will warn you in the console.
如果您这样做，Vue将在控制台中警告您。
There are usually two cases where it's tempting to mutate a prop:
通常有两种情况下，它是诱人的变异一种props:
1.
1.
**The prop is used to pass in an initial value;
**使用prop传递一个初始值;
the child component wants to use it as a local data property afterwards.
子组件希望之后将其用作本地数据属性。
** In this case, it's best to define a local data property that uses the prop as its initial value:
**在这种情况下，最好定义一个使用prop作为初始值的本地数据属性:

```js
props: ['initialCounter'],
data() {
  return {
    counter: this.initialCounter
  }
}
```

**The prop is passed in as a raw value that needs to be transformed.
** prop作为需要转换的原始值传入。
** In this case, it's best to define a computed property using the prop's value:
**在这种情况下，最好使用props的值定义一个计算属性:

```js
props: ['size'],
computed: {
  normalizedSize: function () {
    return this.size.trim().toLowerCase()
  }
}
```

::: tip Note
:::提示注意
Note that objects and arrays in JavaScript are passed by reference, so if the prop is an array or object, mutating the object or array itself inside the child component **will** affect parent state.
注意，JavaScript中的对象和数组是通过引用传递的，所以如果props是数组或对象，在子组件中改变对象或数组本身将会影响父组件的状态。
:::

## Prop Validation
# #支持验证
Components can specify requirements for their props, such as the types you've already seen.
组件可以为它们的props指定需求，比如您已经看到的类型。
If a requirement isn't met, Vue will warn you in the browser's JavaScript console.
如果没有满足某个要求，Vue将在浏览器的JavaScript控制台警告您。
This is especially useful when developing a component that's intended to be used by others.
这在开发供他人使用的组件时特别有用。
To specify prop validations, you can provide an object with validation requirements to the value of `props`, instead of an array of strings.
要指定props验证，您可以为“props”的值提供一个验证要求的对象，而不是一个字符串数组。
For example:
例如:
```js
app.component('my-component', {
  props: {
    // Basic type check (`null` and `undefined` values will pass any type validation)
    propA: Number,
    // Multiple possible types
    propB: [String, Number],
    // Required string
    propC: {
      type: String,
      required: true
    },
    // Number with a default value
    propD: {
      type: Number,
      default: 100
    },
    // Object with a default value
    propE: {
      type: Object,
      // Object or array defaults must be returned from
      // a factory function
      default: function() {
        return { message: 'hello' }
      }
    },
    // Custom validator function
    propF: {
      validator: function(value) {
        // The value must match one of these strings
        return ['success', 'warning', 'danger'].indexOf(value) !== -1
      }
    },
    // Function with a default value
    propG: {
      type: Function,
      // Unlike object or array default, this is not a factory function - this is a function to serve as a default value
      default: function() {
        return 'Default function'
      }
    }
  }
})
```
' ' '
When prop validation fails, Vue will produce a console warning (if using the development build).
当props验证失败时，Vue将生成一个控制台警告(如果使用的是开发版本)。
::: tip Note
:::提示注意
Note that props are validated **before** a component instance is created, so instance properties (e.g.
注意，props是在创建组件实例之前**验证的，所以实例属性(例如。
`data`, `computed`, etc) will not be available inside `default` or `validator` functions.
' data '， ' computed '等)在' default '或' validator '函数中不可用。
:::
:::

### Type Checks

The `type` can be one of the following native constructors:

- String
- Number
- Boolean
- Array
- Object
- Date
- Function
- Symbol

In addition, `type` can also be a custom constructor function and the assertion will be made with an `instanceof` check.
此外，‘type’也可以是一个自定义构造函数，断言将通过‘instanceof’检查完成。
For example, given the following constructor function exists:
例如，给定以下构造函数存在:
```js
function Person(firstName, lastName) {
  this.firstName = firstName
  this.lastName = lastName
}
```

You could use:

```js
app.component('blog-post', {
  props: {
    author: Person
  }
})
```

to validate that the value of the `author` prop was created with `new Person`.

## Prop Casing (camelCase vs kebab-case)

HTML attribute names are case-insensitive, so browsers will interpret any uppercase characters as lowercase.
HTML属性名不区分大小写，因此浏览器将任何大写字符解释为小写字符。
That means when you're using in-DOM templates, camelCased prop names need to use their kebab-cased (hyphen-delimited) equivalents:
这意味着，当你使用in-DOM模板时，驼峰格式的props名称需要使用类似的串串格式(连字符分隔):

```js
const app = Vue.createApp({})

app.component('blog-post', {
  // camelCase in JavaScript
  props: ['postTitle'],
  template: '<h3>{{ postTitle }}</h3>'
})
```

```html
<!-- kebab-case in HTML -->
<blog-post post-title="hello!"></blog-post>
```

Again, if you're using string templates, this limitation does not apply.
