# 一.Pages

在 Next.js 中，一个 **page（页面）** 就是一个从 `.js`,`.jsx`,`.ts`,`.tsx` 文件导出的 **React 组件**，这些文件放在 `pages` 目录下，每个页面都使用其文件名作为路由
例如：你可以创建一个 `pages/about.js` 导出一个 React 组件，它可以通过 `/about` 到达

```jsx
function About() {
  return <div>About</div>;
}
export default About;
```

### 动态路由页面

Next.js 支持动态页面页面，例如，如果您创建一个名为 `pages/posts/[id].js` 的文件，可以通过 `/post/1`, `/post/2` 等访问它
更多内容可以查看 [动态路由文档](https://nextjs.org/docs/routing/dynamic-routes)

## 1.预渲染

默认情况下，Next.js 将预渲染每个页面，这意味着 Next.js 会预先为每个页面生成 HTML 文件，而不是让客户端 JavaScript 完成所有工作，预渲染可以带来更好的性能和 SEO 效果

每个生成的 HTML 页面与该页面所需的最小 JavaScript 代码相关联。当一个页面被浏览器下载后，它的 JavaScript 代码执行并使页面具有完全的交互性（这个过程被称为 **水合** hydration)

### 两种预渲染方式

Next.js 具有两种形式的预渲染：**静态生成（Static Generation）**和**服务器渲染（Serverside Rendering）**，两种方式的不同之处在于生成 HTML 页面的时机

- 静态生成：HTML 文件在构建时生成，每次 request 时复用
- 服务端渲染：每次页面请求（request）时重新生成 HTML 文件

Next.js 允许为每个页面选择预渲染方式，可以创建一个混合渲染的 Next.js 程序，对大多数页面使用静态生成，对其他页面使用服务端渲染

出于性能考虑，更推荐使用静态生成，CDN 可以在没有额外配置对情况下缓存静态生成对页面以提高性能，然而有些情况下我们只能选择服务端渲染的方式

也可以同时使用客户端渲染和静态生成或者服务端渲染，这意味着页面的部分可以完全由客户端 JavaScript 渲染，更多可以看 [数据获取]()

## 2.静态生成(推荐使用)

如果一个页面使用了静态生成，那个**构建时**将生成此页面对应等 HTML 文件，这意味着生产环境中，运行 `next build` 命令时将生成该页面对应的 HTML 文件，然后在每个页面请求时被重用，还可以被 CDN 缓存

带不带数据都可以静态生成

### 不带数据静态生成

默认情况下，Next.js 使用静态生成来预渲染页面，但不涉及获取数据

```jsx
function About() {
  return <div>About</div>;
}
export default About;
```

_注意：页面在渲染时不需要获取任何外部数据的情况下，Next.js 只需要在构建时为页面生成一个 HTML 文件即可_

### 带数据静态生成

- 1.页面**内容**取决于外部数据：`getStaticProps`
- 2.页面**path**取决于外部数据：`getStaticPaths`(通常还要同时使用`getStaticProps`)

场景一：页面内容依赖外部数据
**要在预渲染时获取外部数据，Next.js 允许从同一个文件夹 `exports` 一个名为 `getStaticProps` 的 `async` 函数，该函数在构建时被调用，并允许在预渲染时将获取的数据作为 `props` 参数传递给页面**

```js
// TODO: 需要获取 posts（通过调用后端 API）
//		在此页面被预渲染之前
function Blog({ posts }) {
  return (
    <ul>
      {posts.map((post) => (
        <li>{post.title}</li>
      ))}
    </ul>
  );
}

export async function getStaticProps() {
  // 调用外部 API 获取数据列表
  const res = await fetch("https:// .../posts");
  const posts = await res.json();

  // 通过返回 { props: { posts }} 对象
  // Blog 组件在构建时将接收到 posts 参数
  return {
    props: {
      posts,
    },
  };
}

export default Blog;
```

了解更多见 [数据获取文档]()

场景二：页面路径取决于外部数据

Next.js 允许您用动态路由创建页面，例如，您可以创建一个名叫 `pages/posts/[id].js` 的文件，来展示基于 `id` 区分的单篇博客，当你进入 `/post/1` 时你可以展示一篇博客文章

然而，您想在构建时哪个 `id`

构建博客
