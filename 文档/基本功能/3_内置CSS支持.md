# 内建支持 CSS

Next.js 允许您从一个 JS 文件导入一个 CSS 文件，这是 Next.js 通过扩展 JS 的 `import` 实现的

## 添加全局样式表

在 `pages/_app.js` 里引入 CSS 文件可以给应用程序添加全局样式

创建一个 `pages/_app.js` 文件（如果没有），然后引入 CSS 文件

```js
import '../styles.css'

export default function MyApp({ Component, pageProps }) {
  return <Component { ...pageProps }>
}
```

这些样式回应用到应用里的所有页面和组件。由于样式表带全局特性，并且为了避免冲突，宁应该只在 `pages/_app.js` 里引用它

在开发环境中，以这种方式表达样式表可以在编辑时重新加载样式，这意味着可以保持程序状态

在生产环境中，所有 CSS 文件会被自动拼接到一个单独的压缩 CSS 文件中

### 从 node_modules 引入样式

从 Next.js 9.5.4 版本后，在程序中的任何地方从 `node_modules` 引入 CSS 文件都是被允许的。
对于全局样式表，比如 `bootstrap` `nprogress` 你可以在 `pages/_app.js` 中引入

```js
import "bootstrap/dist/css/bootstrap.css";

export default function MyApp({ Component, pageProps }) {
  return <Component {...pageProps} />;
}
```

从第三方库组件引入 CSS ，你可以在自己的组件中引入

## 添加组件级样式

Next.js 使用 `[name].module.css` 命名约定支持 CSS 模块
通过自动创建唯一的类名，CSS 模块可以在本地范围内作用于 CSS，这允许您在不同的文件中使用相同的 CSS 类名而不用担心冲突

这种行为使得 CSS 模块成为包含组件级 CSS 的理想方式，可以在应用的任意地方引入 CSS 模块文件

例如，`components/` 目录下的可复用 `Button` 组件：
第一步，创建 `components/Button.module.css` 文件

```css
/*
不用担心 .error {} 会和其他 .css 或者 .module.css 文件冲突
*/
.error {
  color: white;
}
```

然后创建 `components/Button.js`，引入并使用上面的 CSS 文件

```js
import styles from "./Button.module.css";

export function Button() {
  return (
    <button
      type="button"
      // 注意：此时 error 类是作为引入的 styles 对象的一个属性被使用
      className={styles.error}
    >
      Destroy
    </button>
  );
}
```

**CSS 模块**是一个可选的特性，并且只对扩展名为 _.module.css_ 的文件启用。常规的 `<link>` 样式表和全局样式表依旧支持

在生产环境中，所有 CSS 模块会自动拼接到许多缩小的和代码分裂的 `.css` 文件中，这些 CSS 文件代表应用程序中的热执行路径，确保应用程序绘制（paint）时加载最少量的 CSS

## Sass 支持

Next.js 允许使用 `.scss` 和 `.sass` 扩展名导入 **Sass**，你可以通过 CSS 模块和 `.module.scss` 和 `.module.sass` 使用组件级的 Sass

使用 Next.js 内置的 Sass 支持前，请确保安装了 _sass_

```sh
npm install --save-dev sass
```

Sass 支持和上面详细说的 CSS 内建支持具有相同的优点和限制

**注意**（我没用过 Sass）：Sass supports two different syntaxes, each with their own extension. The .scss extension requires you use the SCSS syntax, while the .sass extension requires you use the Indented Syntax ("Sass").

If you're not sure which to choose, start with the .scss extension which is a superset of CSS, and doesn't require you learn the Indented Syntax ("Sass")

### 自定义 Sass 选项

如果您想配置 Sass 编译，你可以在 `next.config.js` 中用 `sassOptions`实现

例如添加 `includePaths`:

```js
const path = require("path");

module.exports = {
  sassOptions: {
    includePaths: [path.join(__dirname, "styles")],
  },
};
```

### Sass 变量

Next.js 支持从 CSS 模块导出 Sass 变量

例如，使用导出的 `primaryColor` Sass 变量：

```css
/* variables.module.scss */
$primary-color: #64ff00;

:export {
  primaryColor: $primary-color;
}
```

```js
// pages/_app.js
import variables from "../styles/variables.module.scss";

export default function MyApp({ Component, pageProps }) {
  return (
    <Layout color={variables.primaryColor}>
      <Component {...pageProps} />
    </Layout>
  );
}
```

## CSS-in-JS

可以使用任意现有的 CSS-in-JS 解决方案，最简单的是行内样式：

```js
export default function HiThere() {
  return <p style={{ color: "red" }}>hi there</p>;
}
```

我们绑定 styled-jsx 来提供独立作用域的 CSS 支持，其目的是支持 `shadow CSS` 类似于 Web Components，不幸的是（WebComponent）不支持服务器渲染，只支持 JS

参阅上面的例子，了解其他流行的 CSS-in-JS 解决方案（例如 styled-components）

使用 `styled-jsx` 的组件如下所示：

```jsx
export default function HelloWorld() {
  return (
    <div>
      Hello world
      <p>scoped!</p>
      <style jsx>{`
        p {
          color: blue;
        }
        div {
          background: red;
        }
        @media (max-width: 600px) {
          div {
            background: blue;
          }
        }
      `}</style>
      <style global jsx>{`
        body {
          background: black;
        }
      `}</style>
    </div>
  );
}
```

## FAQ

- 在禁用 JavaScript 时可以生效吗？
  可以的，如果禁用了 JavaScript ，CSS 仍然会在生产版本中加载（`next build` 过程中），在开发环境中，我们需要启用 JavaScript，以便使用 [Fast Refresh]() 提供最佳的开发人员体验
