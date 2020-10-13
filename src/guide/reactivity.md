# Reactivity in Depth  深度响应

Now it’s time to take a deep dive!
现在是时候深挖了!
One of Vue’s most distinct features is the unobtrusive reactivity system.
Vue最显著的特点之一是其不引人注目的反应系统。
Models are proxied JavaScript objects.
模型是代理的JavaScript对象。
When you modify them, the view updates.
当您修改它们时，视图会更新。
It makes state management simple and intuitive, but it’s also important to understand how it works to avoid some common gotchas.
它使状态管理简单而直观，但是理解它如何工作以避免一些常见的陷阱也很重要。
In this section, we are going to dig into some of the lower-level details of Vue’s reactivity system.
在本节中，我们将深入研究Vue的反应系统的一些底层细节。

<VideoLesson href="https://www.vuemastery.com/courses/vue-3-reactivity/vue3-reactivity" title="Learn how how reactivity works in depth with Vue Mastery">Watch a free video on Reactivity in Depth on Vue Mastery</VideoLesson>

## What is Reactivity? 什么事引用
This term comes up in programming quite a bit these days, but what do people mean when they say it?
这个术语最近在编程中经常出现，但是人们说它的时候是什么意思呢?
Reactivity is a programming paradigm that allows us to adjust to changes in a declarative manner.
反应性是一种编程范式，它允许我们以声明的方式调整更改。
The canonical example that people usually show, because it’s a great one, is an excel spreadsheet.
人们通常展示的典型例子，因为它很好，是一个excel电子表格。

<video width="550" height="400" controls>
  <source src="/images/reactivity-spreadsheet.mp4" type="video/mp4">
  Your browser does not support the video tag.
</video>

If you put the number two in the first cell, and the number 3 in the second and asked for the SUM, the spreadsheet would give it to you.
如果你把数字2放在第一个单元格，把数字3放在第二个单元格，然后要求和，电子表格会给你。
No surprises there.
没有惊喜。
But if you update that first number, the SUM automagically updates too.
但如果你更新第一个数字，和也会自动更新。
JavaScript doesn’t usually work like this -- If we were to write something comparable in JavaScript:
JavaScript通常不是这样工作的——如果我们用JavaScript写一些类似的东西:

```js
var val1 = 2
var val2 = 3
var sum = val1 + val2

// sum
// 5

val1 = 3

// sum
// 5
```

If we update the first value, the sum is not adjusted.
如果我们更新第一个值，则不会调整总和。
So how would we do this in JavaScript?
JavaScript怎么做呢?
- Detect when there’s a change in one of the values
-检测其中一个值是否发生变化
- Track the function that changes it
-跟踪改变它的功能
- Trigger the function so it can update the final value
-触发函数，使其可以更新最终值

## How Vue Tracks These Changes
## Vue如何跟踪这些变化
When you pass a plain JavaScript object to an application or component instance as its `data` option, Vue will walk through all of its properties and convert them to [Proxies](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy) using a handler with getters and setters.
当你纯JavaScript对象传递给一个应用程序或组件实例作为它的“数据”选项,Vue将遍历所有的属性和将其转换成(代理)(https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy)使用getter和setter方法的处理程序。
This is an ES6-only feature, but we offer a version of Vue 3 that uses the older `Object.defineProperty` to support IE browsers.
这是es6特有的特性，但是我们提供了一个Vue 3版本，使用旧的Object.defineProperty来支持IE浏览器。
Both have the same surface API, but the Proxy version is slimmer and offers improved performance.
两者都有相同的surface API，但代理版本更简洁，并提供了改进的性能。

<div class="reactivecontent">
  <iframe height="500" style="width: 100%;" scrolling="no" title="Proxies and Vue's Reactivity Explained Visually" src="https://codepen.io/sdras/embed/zYYzjBg?height=500&theme-id=light&default-tab=result" frameborder="no" allowtransparency="true" allowfullscreen="true">
    See the Pen <a href='https://codepen.io/sdras/pen/zYYzjBg'>Proxies and Vue's Reactivity Explained Visually</a> by Sarah Drasner
    (<a href='https://codepen.io/sdras'>@sdras</a>) on <a href='https://codepen.io'>CodePen</a>.
  </iframe>
</div>

That was rather quick and requires some knowledge of [Proxies](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy) to understand!
这相当快，需要了解一些[代理]的知识(https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy) !
So let’s dive in a bit.
让我们深入一点。
There’s a lot of literature on Proxies, but what you really need to know is that a **Proxy is an object that encases another object or function and allows you to intercept it.**
有很多关于代理的文献，但是您真正需要知道的是**代理是一个对象，它封装了另一个对象或函数，并允许您拦截它
We use it like this: `new Proxy(target, handler)`
我们这样使用它:' new Proxy(target, handler) '

```js
const dinner = {
  meal: 'tacos'
}

const handler = {
  get(target, prop) {
    return target[prop]
  }
}

const proxy = new Proxy(dinner, handler)
console.log(proxy.meal)

// tacos
```

Ok, so far, we’re just wrapping that object and returning it.
到目前为止，我们只是包装那个对象并返回它。
Cool, but not that useful yet.
很酷，但还不是很有用。
But watch this, we can also intercept this object while we wrap it in the Proxy.
但是看这里，我们也可以拦截这个对象，当我们把它包装在代理中。
This interception is called a trap.
这种拦截称为陷阱。

```js
const dinner = {
  meal: 'tacos'
}

const handler = {
  get(target, prop) {
    console.log(‘intercepted!’)
    return target[prop]
  }
}

const proxy = new Proxy(dinner, handler)
console.log(proxy.meal)

// intercepted!
// tacos
```

Beyond a console log, we could do anything here we wish.
除了控制台日志之外，我们可以在这里做任何我们想做的事情。
We could even _not_ return the real value if we wanted to.
如果我们想的话，我们甚至可以不返回实际值。
This is what makes Proxies so powerful for creating APIs.
这就是代理对于创建api如此强大的原因。
Furthermore, there’s another feature Proxies offer us.
此外，代理还提供了另一个特性。
Rather than just returning the value like this: `target[prop]`, we could take this a step further and use a feature called `Reflect`, which allows us to do proper `this` binding.
与其像这样返回值‘target[prop]’，我们还可以更进一步，使用一个叫做‘Reflect’的特性，它允许我们做正确的‘this’绑定。
It looks like this:
它是这样的:

```js{7}
const dinner = {
  meal: 'tacos'
}

const handler = {
  get(target, prop, receiver) {
    return Reflect.get(...arguments)
  }
}

const proxy = new Proxy(dinner, handler)
console.log(proxy.meal)

// intercepted!
// tacos
```

We mentioned before that in order to have an API that updates a final value when something changes, we’re going to have to set new values when something changes.
我们之前提到过，为了让API在某些东西发生变化时更新最终值，我们需要在某些东西发生变化时设置新值。
We do this in the handler, in a function called `track`, where we pass in the `target` and `key`.
我们在处理程序中做这个，在一个叫做' track '的函数中，我们传入' target '和' key '。

```js{7}
const dinner = {
  meal: 'tacos'
}

const handler = {
  get(target, prop, receiver) {
    track(target, prop)
    return Reflect.get(...arguments)
  }
}

const proxy = new Proxy(dinner, handler)
console.log(proxy.meal)

// intercepted!
// tacos
```

Finally, we also set new values when something changes.
最后，当某些东西发生变化时，我们还设置了新的价值观。
For this, we’re going to set the changes on our new proxy, by triggering those changes:
为此，我们将在我们的新代理上设置更改，通过触发这些更改:

```js
const dinner = {
  meal: 'tacos'
}

const handler = {
  get(target, prop, receiver) {
    track(target, prop)
    return Reflect.get(...arguments)
  },
  set(target, key, value, receiver) {
    trigger(target, key)
    return Reflect.set(...arguments)
  }
}

const proxy = new Proxy(dinner, handler)
console.log(proxy.meal)

// intercepted!
// tacos
```

Remember this list from a few paragraphs ago?
还记得前几段的列表吗?

Now we have some answers to how Vue handles these changes:
现在我们有了一些关于Vue如何应对这些变化的答案:

- <strike>Detect when there’s a change in one of the values</strike>: we no longer have to do this, as Proxies allow us to intercept it
- <strike>检测</strike>值的变化:我们不再需要这样做，因为代理允许我们拦截它

- **Track the function that changes it**: We do this in a getter within the proxy, called `effect`
- **跟踪改变它的函数**:我们在代理中的getter中做这个，称为“effect”

- **Trigger the function so it can update the final value**: We do in a setter within the proxy, called `trigger`
- **触发函数，这样它就可以更新最终的值**:我们在代理中的setter中执行，称为' Trigger '

The proxied object is invisible to the user, but under the hood they enable Vue to perform dependency-tracking and change-notification when properties are accessed or modified.
代理对象对用户是不可见的，但在底层，它们使Vue能够在访问或修改属性时执行依赖跟踪和更改通知
。
As of Vue 3, our reactivity is now available in a [separate package](https://github.com/vuejs/vue-next/tree/master/packages/reactivity).
从Vue 3开始，我们的反应现在在一个[单独的包]中可用(https://github.com/vuejs/vue-next/tree/master/packages/reactivity)。

One caveat is that browser consoles format differently when converted data objects are logged, so you may want to install [vue-devtools](https://github.com/vuejs/vue-devtools) for a more inspection-friendly interface.
需要注意的是，在记录转换后的数据对象时，浏览器控制台的格式会有所不同，因此您可能需要安装[vue-devtools](https://github.com/vuejs/vue-devtools)以获得更友好的检查界面。

### Proxied Objects

Vue internally tracks all objects that have been made reactive, so it always returns the same proxy for the same object.
Vue在内部跟踪所有被激活的对象，因此它总是为相同的对象返回相同的代理。
When a nested object is accessed from a reactive proxy, that object is _also_ converted into a proxy before being returned:
当一个嵌套对象被一个反应性代理访问时，该对象在返回之前被_also转换为代理:

```js
const handler = {
  get(target, prop, receiver) {
    track(target, prop)
    const value = Reflect.get(...arguments)
    if (isObject(value)) {
      return reactive(value)
    } else {
      return value
    }
  }
  // ...
}
```

### Proxy vs. original identity

The use of Proxy does introduce a new caveat to be aware with: the proxied object is not equal to the original object in terms of identity comparison (`===`).
使用代理确实引入了一个需要注意的新警告:就身份比较而言，代理对象与原始对象不相等(' === ')。
For example:
例如:

```js
const obj = {}
const wrapped = new Proxy(obj, handlers)

console.log(obj === wrapped) // false
```

The original and the wrapped version will behave the same in most cases, but be aware that they will fail
在大多数情况下，原始版本和包装版本的行为是相同的，但是要注意它们会失败
operations that rely on strong identity comparisons, such as `.filter()` or `.map()`.
依赖于强标识比较的操作，如' .filter() '或' .map() '。
This caveat is unlikely to come up when using the options API, because all reactive state is accessed from `this` and guaranteed to already be proxies.
在使用options API时，不太可能出现这个警告，因为所有的反应状态都是从“This”访问的，并且保证已经是代理。
However, when using the composition API to explicitly create reactive objects, the best practice is to never hold a reference to the original raw object and only work with the reactive version:
然而，当使用composition API显式创建反应对象时，最佳实践是永远不要持有原始对象的引用，而只使用反应版本:

```js
const obj = reactive({
  count: 0
}) // no reference to original
```

## Watchers

Every component instance has a corresponding watcher instance, which records any properties "touched" during the component’s render as dependencies.
每个组件实例都有一个相应的观察者实例，它记录在组件呈现期间作为依赖项“接触”的任何属性。
Later on when a dependency’s setter is triggered, it notifies the watcher, which in turn causes the component to re-render.
稍后，当一个依赖项的setter被触发时，它会通知观察者，这反过来导致组件重新呈现。

<div class="reactivecontent">
  <iframe height="500" style="width: 100%;" scrolling="no" title="Second Reactivity with Proxies in Vue 3 Explainer" src="https://codepen.io/sdras/embed/GRJZddR?height=500&theme-id=light&default-tab=result" frameborder="no" allowtransparency="true" allowfullscreen="true">
    See the Pen <a href='https://codepen.io/sdras/pen/GRJZddR'>Second Reactivity with Proxies in Vue 3 Explainer</a> by Sarah Drasner
    (<a href='https://codepen.io/sdras'>@sdras</a>) on <a href='https://codepen.io'>CodePen</a>.
  </iframe>
</div>

When you pass an object to a component instance as data, Vue converts it to a proxy.
当您将对象作为数据传递给组件实例时，Vue将其转换为代理。
This proxy enables Vue to perform dependency-tracking and change-notification when properties are accessed or modified.
这个代理使Vue能够在访问或修改属性时执行依赖跟踪和更改通知。
Each property is considered a dependency.
每个属性都被视为一个依赖项。
After the first render, a component would have tracked a list of dependencies &mdash;
在第一次渲染之后，组件将跟踪依赖项列表&mdash;
the properties it accessed during the render.
它在呈现期间访问的属性。
Conversely, the component becomes a subscriber to each of these properties.
相反，组件成为这些属性的订阅服务器。
When a proxy intercepts a set operation, the property will notify all of its subscribed components to re-render.
当代理截获了set操作时，该属性将通知其所有订阅的组件重新呈现。

[//]: # 'TODO: Insert diagram'

> If you are using Vue 2.x and below, you may be interested in some of the change detection caveats that exist for those versions, [explored in more detail here](change-detection.md).
