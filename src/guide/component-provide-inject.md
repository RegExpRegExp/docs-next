# Provide / inject(提供/注入)
> This page assumes you've already read the [Components Basics](component-basics.md).
>本页假设您已经阅读了[Components Basics](component-basic .md)。
Read that first if you are new to components.
如果您对组件不熟悉，请先阅读它。
Usually, when we need to pass data from the parent to child component, we use [props](component-props.md).
通常，当我们需要将数据从父组件传递给子组件时，我们使用[props](component-props.md)。
Imagine the structure where you have some deeply nested components and you only need something from the parent component in the deep nested child.
想象一下这样的结构:您有一些深嵌套的组件，而在深嵌套的子组件中只需要父组件中的一些内容。
In this case, you still need to pass the prop down the whole component chain which might be annoying.
在这种情况下，您仍然需要将支撑传递到整个组件链，这可能很烦人。
For such cases, we can use the `provide` and `inject` pair.
对于这种情况，我们可以使用' provide '和' inject '对。
Parent components can serve as dependency provider for all its children, regardless how deep the component hierarchy is.
无论组件层次结构有多深，父组件都可以作为其所有子组件的依赖提供程序。
This feature works on two parts: parent component has a `provide` option to provide data and child component has an `inject` option to start using this data.
这个特性在两个方面起作用:父组件有一个'提供'选项来提供数据，子组件有一个'注入'选项来开始使用这些数据。
![Provide/inject scheme](/images/components_provide.png)
![提供/注入方案](/图片/ components_provide.png)
For example, if we have a hierarchy like this:
例如，如果我们有一个像这样的层次结构:

```
Root
└─ TodoList
   ├─ TodoItem
   └─ TodoListFooter
      ├─ ClearTodosButton
      └─ TodoListStatistics
```

If we want to pass the length of todo-items directly to `TodoListStatistics`, we would pass the prop down the hierarchy: `TodoList` -> `TodoListFooter` -> `TodoListStatistics`.
如果我们想将todo-item的长度直接传递给“TodoListStatistics”，我们将向下传递这个支柱:“TodoList”->“TodoListFooter”->“TodoListStatistics”。
With provide/inject approach, we can do this directly:
通过提供/注入方法，我们可以直接做到这一点:


```js
const app = Vue.createApp({})

app.component('todo-list', {
  data() {
    return {
      todos: ['Feed a cat', 'Buy tickets']
    }
  },
  provide: {
    user: 'John Doe'
  },
  template: `
    <div>
      {{ todos.length }}
      <!-- rest of the template -->
    </div>
  `
})

app.component('todo-list-statistics', {
  inject: ['user'],
  created() {
    console.log(`Injected property: ${this.user}`) // > Injected property: John Doe
  }
})
```

However, this won't work if we try to provide some component instance property here:
然而，如果我们尝试在这里提供一些组件实例属性，这将不会工作:
```js
app.component('todo-list', {
  data() {
    return {
      todos: ['Feed a cat', 'Buy tickets']
    }
  },
  provide: {
    todoLength: this.todos.length // this will result in error 'Cannot read property 'length' of undefined`
  },
  template: `
    ...
  `
})
```


length // this will result in error 'Cannot read property 'length' of undefined`
length //这会导致“无法读取未定义的属性”length“

```js
app.component('todo-list', {
  data() {
    return {
      todos: ['Feed a cat', 'Buy tickets']
    }
  },
  provide() {
    return {
      todoLength: this.todos.length
    }
  },
  template: `
    ...
  `
})
```

This allows us to more safely keep developing that component, without fear that we might change/remove something that a child component is relying on.
这允许我们更安全地继续开发该组件，而不用担心我们可能会更改/删除子组件所依赖的东西。
The interface between these components remains clearly defined, just as with props.
这些组件之间的接口仍然被清晰地定义，就像道具一样。
In fact, you can think of dependency injection as sort of “long-range props”, except:
事实上，你可以把依赖注入看作是一种“远程道具”，除了:
- parent components don’t need to know which descendants use the properties it provides
-父组件不需要知道哪个后代使用它提供的属性
- child components don’t need to know where injected properties are coming from
-子组件不需要知道注入的属性来自哪里
## Working with reactivity
##处理反应性
In the example above, if we change the list of `todos`, this change won't be reflected in the injected `todoLength` property.
在上面的例子中，如果我们改变了“todos”列表，这个改变不会反映在注入的“todoLength”属性中。
This is because `provide/inject` bindings are _not_ reactive by default.
这是因为‘提供/注入’绑定在默认情况下是_not_反应的。
We can change this behavior by passing a `ref` property or `reactive` object to `provide`.
我们可以通过将“ref”属性或“reactive”对象传递给“provide”来改变这种行为。
In our case, if we wanted to react to changes in the ancestor component, we would need to assign a Composition API `computed` property to our provided `todoLength`:
在我们的例子中，如果我们想对祖先组件的变化做出反应，我们需要为我们提供的“todoLength”分配一个Composition API ' computed '属性:


```js
app.component('todo-list', {
  // ...
  provide() {
    return {
      todoLength: Vue.computed(() => this.todos.length)
    }
  }
})
```
In this, any change to `todos.
在这里，任何对“待办事项”的改变。

length` will be reflected correctly in the components, where `todoLength` is injected.
长度'将正确地反映在组件中，其中' todoLength '被注入。

Read more about `reactive` provide/inject in the [Composition API section](composition-api-provide-inject.html#reactivity)
在[合成API部分]阅读更多关于“反应性”提供/注入的内容。
