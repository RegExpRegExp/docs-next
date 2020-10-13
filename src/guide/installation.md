# Installation

Vue.js is built by design to be incrementally adoptable. This means that it can be integrated into a project multiple ways depending on the requirements.

There are three primary ways of adding Vue.js to a project:

1. Import it as a [CDN package](#cdn) on the page
2. Install it using [npm](#npm)
3. Use the official [CLI](#cli) to scaffold a project, which provides batteries-included build setups for a modern frontend workflow (e.g., hot-reload, lint-on-save, and much more)

## Release Notes

Latest version: ![npm](https://img.shields.io/npm/v/vue/next.svg)

Detailed release notes for each version are available on [GitHub](https://github.com/vuejs/vue-next/blob/master/CHANGELOG.md).

## Vue Devtools

> Currently in Beta - Vuex and Router integration is still WIP

When using Vue, we recommend also installing the [Vue Devtools](https://github.com/vuejs/vue-devtools#vue-devtools) in your browser, allowing you to inspect and debug your Vue applications in a more user-friendly interface.

[Get the Chrome Extension](https://chrome.google.com/webstore/detail/vuejs-devtools/ljjemllljcmogpfapbkkighbhhppjdbg)

[Get the Firefox Addon](https://addons.mozilla.org/en-US/firefox/addon/vue-js-devtools/)

[Get the standalone Electron app](https://github.com/vuejs/vue-devtools/blob/dev/packages/shell-electron/README.md)

## CDN

For prototyping or learning purposes, you can use the latest version with:

```html
<script src="https://unpkg.com/vue@next"></script>
```

For production, we recommend linking to a specific version number and build to avoid unexpected breakage from newer versions.

## npm

npm is the recommended installation method when building large scale applications with Vue. It pairs nicely with module bundlers such as [Webpack](https://webpack.js.org/) or [Rollup](https://rollupjs.org/). Vue also provides accompanying tools for authoring [Single File Components](../guide/single-file-component.html).

```bash
# latest stable
$ npm install vue@next
```

## CLI

Vue provides an [official CLI](https://github.com/vuejs/vue-cli) for quickly scaffolding ambitious Single Page Applications. It provides batteries-included build setups for a modern frontend workflow. It takes only a few minutes to get up and running with hot-reload, lint-on-save, and production-ready builds. See [the Vue CLI docs](https://cli.vuejs.org) for more details.

::: tip
The CLI assumes prior knowledge of Node.js and the associated build tools. If you are new to Vue or front-end build tools, we strongly suggest going through [the guide](./introduction.html) without any build tools before using the CLI.
:::

For Vue 3, you should use Vue CLI v4.5 available on `npm` as `@vue/cli`. To upgrade, you need to reinstall the latest version of `@vue/cli` globally:

```bash
yarn global add @vue/cli
# OR
npm install -g @vue/cli
```

Then in the Vue projects, run

```bash
vue upgrade --next
```

## Vite

[Vite](https://github.com/vitejs/vite) is a web development build tool that allows for lightning fast serving of code due its native ES Module import approach.

Vue projects can quickly be set up with Vite by running the following commands in your terminal.

With npm:

```bash
$ npm init vite-app <project-name>
$ cd <project-name>
$ npm install
$ npm run dev
```

Or with Yarn:

```bash
$ yarn create vite-app <project-name>
$ cd <project-name>
$ yarn
$ yarn dev
```

## Explanation of Different Builds 不同打包的解释

In the [`dist/` directory of the npm package](https://cdn.jsdelivr.net/npm/vue@3.0.0-rc.1/dist/) you will find many different builds of Vue.js.
在npm包的[' dist/ '目录](https://cdn.jsdelivr.net/npm/vue@3.0.0-rc.1/dist/)中，你会发现许多不同版本的Vue.js。

Here is an overview of which `dist` file should be used depending on the use-case.
以下是应该根据用例使用哪个“dist”文件的概述。

### From CDN or without a Bundler
从CDN或没有捆绑

#### `vue(.runtime).global(.prod).js`:

- For direct use via `<script src="...">` in the browser, exposes the Vue global.
-直接使用通过' <script src="…“>”在浏览器中，公开Vue全局变量。



- In-browser template compilation:
-浏览器内模板编译:

- `vue.global.js` is the "full" build that includes both the compiler and the runtime so it supports compiling templates on the fly.
——“vue.global.js是“完整的”构建，包括编译器和运行时，所以它支持动态编译模板。

- `vue.runtime.global.js` contains only the runtime and requires templates to be pre-compiled during a build step.
- ' vue.runtime.global.js '只包含运行时，并且要求模板在构建过程中进行预编译。

- Inlines all Vue core internal packages - i.e. it's a single file with no dependencies on other files.
-内联所有Vue核心内部包-即，它是一个独立的文件，不依赖于其他文件。

This means you must import everything from this file and this file only to ensure you are getting the same instance of code.
这意味着您必须从这个文件和这个文件中导入所有内容，以确保获得相同的代码实例。

- Contains hard-coded prod/dev branches, and the prod build is pre-minified.
-包含硬编码的prod/dev分支，并且prod构建是预先缩小的。

Use the `*.prod.
使用‘* .prod。

js` files for production.
js的文件用于生产。

:::tip Note
Global builds are not [UMD](https://github.com/umdjs/umd) builds. They are built as [IIFEs](https://developer.mozilla.org/en-US/docs/Glossary/IIFE) and are only meant for direct use via `<script src="...">`.
:::

#### vue(.runtime).esm-browser(.prod).js:

- For usage via native ES modules imports (in browser via `<script type="module">`.
-通过本地ES模块导入使用(在浏览器中通过`<script type="module"> `

- Shares the same runtime compilation, dependency inlining and hard-coded prod/dev behavior with the global build.
-与全局构建共享相同的运行时编译、依赖内联和硬编码的prod/dev行为。

### With a Bundler

#### vue(.runtime).esm-bundler.js:

- For use with bundlers like `webpack`, `rollup` and `parcel`.
-与“webpack”、“rollup”及“parcel”等捆扎程序一起使用。

- Leaves prod/dev branches with `process.env.
-将prod/dev分支设置为“process.env”。

NODE_ENV guards` (must be replaced by bundler)
NODE_ENV guard '(必须被bundler替换)

- Does not ship minified builds (to be done together with the rest of the code after bundling)
-不提供缩小的构建(在绑定后与其余代码一起完成)

- Imports dependencies (e.g.
-导入依赖项(例如:

`@vue/runtime-core`, `@vue/runtime-compiler`)
“@vue / runtime-core”、“@vue /运行时编译)

- Imported dependencies are also esm-bundler builds and will in turn import their dependencies (e.g. @vue/runtime-core imports @vue/reactivity)
导入的依赖项也是esm-bundler构建，并将依次导入它们的依赖项(例如@vue/运行时-core导入@vue/reactivity)

- This means you **can** install/import these deps individually without ending up with different instances of these dependencies, but you must make sure they all resolve to the same version.
-这意味着您可以** *单独安装/导入这些dep，而不会导致这些依赖项的不同实例，但您必须确保它们都解析为相同的版本。

- In-browser template compilation:
-浏览器内模板编译:

- `vue.runtime.esm-bundler.js` **(default)** is runtime only, and requires all templates to be pre-compiled.
- ' vue.runtime.esm-bundler.js '的**(默认)**是只运行时的，并且要求所有的模板都是预先编译的。

This is the default entry for bundlers (via module field in `package.json`) because when using a bundler templates are typically pre-compiled (e.g. in `*.
这是绑定器的默认条目(通过‘package.json’中的模块字段)，因为使用绑定器模板通常是预编译的(例如在‘*中)。
vue` files).
vue的文件)。

- `vue.esm-bundler.js`: includes the runtime compiler.
- ' vue.esm-bundler.js ':包含运行时编译器。

Use this if you are using a bundler but still want runtime template compilation (e.g. in-DOM templates or templates via inline JavaScript strings).
如果你正在使用一个bundler但仍然需要运行时模板编译(例如dom内模板或通过内联JavaScript字符串的模板)，可以使用这个。

You will need to configure your bundler to alias vue to this file.
您将需要配置您的bundler以将vue别名配置到这个文件。
### For Server-Side Rendering

#### `vue.cjs(.prod).js`:

- For use in Node.js server-side rendering via `require()`.
-通过`require()`用于Node.js服务器端渲染。

- If you bundle your app with webpack with `target: 'node'` and properly externalize `vue`, this is the build that will be loaded.
-如果你把你的应用和webpack捆绑在一起，并正确地具体化“vue”，这就是将要加载的版本。

- The dev/prod files are pre-built, but the appropriate file is automatically required based on `process.env.NODE_ENV`.
- dev/prod文件是预构建的，但是根据' process.env.NODE_ENV '自动需要相应的文件。

## Runtime + Compiler vs. Runtime-only

If you need to compile templates on the client (e.g. passing a string to the template option, or mounting to an element using its in-DOM HTML as the template), you will need the compiler and thus the full build:
如果你需要在客户端编译模板(例如，传递一个字符串到模板选项，或挂载到一个元素使用它的in-DOM HTML作为模板)，你将需要编译器和完整的构建:
```js
// this requires the compiler
Vue.createApp({
  template: '<div>{{ hi }}</div>'
})

// this does not
Vue.createApp({
  render() {
    return Vue.h('div', {}, this.hi)
  }
})
```
When using `vue-loader`, templates inside `*.
当使用' vue-loader '时，模板在' *中。

vue` files are pre-compiled into JavaScript at build time.
vue的文件在构建时被预编译成JavaScript。

You don’t really need the compiler in the final bundle, and can therefore use the runtime-only build.
您实际上并不需要最终包中的编译器，因此可以使用只运行时的构建。
