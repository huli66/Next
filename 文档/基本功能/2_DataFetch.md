### 二、DataFetching 数据获取

#### 1.getServerSideProps

如果在一个页面 `export` 了一个 `getServerSideProps` 函数，Next.js 会在每次请求时用 `getServerSideProps` 返回的数据预渲染这个页面

_注意不论渲染类型，任何 props 都会被传递到页面组件（page component）并可以在客户端到初始 HTML 文件中查看，这样可以给页面进行正确到水合处理，确保您不会传递任何不应该在 `props` 中提供给客户端的敏感信息_

水合：每个生成的 HTML 文件都与该页面所需的最少的 JavaScript 代码相关联，当浏览器加载一个页面时，其 JavaScript 代码运行并使页面完全具有交互性，此过程称为水合

##### getServerSideProps 在什么时候执行

`getServerSideProps` 只在服务器端运行不在客户端运行，如果页面使用了它，那么：

- 当你直接请求页面，`getSerVerSideProps` 在请求时执行，然后页面将被使用返回的 props 预渲染
- 当你在客户端通过 `next/link` or `next/routes` 请求页面转换时，Next.js 向服务器发送一个 API 请求执行 `getServerSideProps`

`getServerSideProps` 返回用于渲染页面的 JSON，这些所用工作都由 Next.js 自动处理，所以你只需要定义了 `getServerSideProps` 不需要做任何额外工作

可以用 [next-code-elimination tool](https://next-code-elimination.vercel.app/) 验证 Next.js 从客户端包中消除了什么
（这个页面里比较写的基于 Next.js 代码和客户端收到的 HTML 文件的代码）

`getServerSideProps` 只能从一个页面 `export`(导出)，不可以从非页面文件导出

注意必须将 `getServerSideProps` 作为一个独立的函数导出，如果你将之作为页面组件(page component)的一个属性是不会生效的

##### 在什么时候使用 getServerSideProps

应该在需要渲染一个页面数据需要在请求时获取时使用（高时效性），这可能是由于数据的性质或请求的属性（比如授权 headers 或者地理位置），页面使用 `getServerSideProps` 会在请求是进行服务端渲染，只有在`cache-control headers`配置了才会缓存

如果不需要在请求时渲染数据，就应该考虑在客户端获取数据或者用 `getStaticProps`

- getServerSideProps or API Routes
  It can be tempting to reach for an API Route when you want to fetch data from the server, then call that API route from getServerSideProps，这是一种不必要且低效对方法，因为它会同时在服务器端运行 `getServerSideProps` 和 API Routes 造成额外的请求
  下面的例子，一个 API 路由被用来从 CMS 获取一些数据，然后直接从 `getServerSideProps` 调用这个 API 路由，这产生了一个额外的调用，降低性能。相反，直接将 API Route 中使用的逻辑导入到 `getServerSideProps` 中，这意味着直接从 `getServerSideProps` 内部调用 CMS、数据库或其他 API

##### 客户端获取数据

如果你的页面频繁更新数据，而你不需要预渲染数据，你可以在客户端获取数据，这方面到一个例子是特定于用户到数据

- 首先，立即显示没有数据到页面，可以用静态生成预先呈现页面到某些部分，缺失数据显示 loading
- 然后，在客户端获取数据，准备好后展示

这种方法用于用户仪表板页面效果很好，比如，因为仪表盘是一个私人订制的用户特定的页面，SEO 没有意义，并且页面不需要被预渲染，数据频繁更新，要求发送请求时获取数据

##### 请求时用 getServerSideProps 拉取数据

```js
function Page({ data }) {
  // Render data ...
}

export async function getServerSideProps(ctx) {
  // 获取数据
  const res = await fetch(`https://.../data`);
  const data = await res.json();

  // 通过 props 传递数据给页面
  return {
    props: {
      data,
    },
  };
}

export default Page;
```

##### 服务端渲染缓存

可以缓存动态响应，比如使用 stale-while-revalidate

```js
// s-maxage=10 这个值在 10s 内被认为是新的
// 如果接下来 10s 内重复请求，前面缓存的值依然新鲜（有效）
// stale-while-revalidate=59 如果在 59s 前重复请求，缓存变旧但是依然渲染

// 在后台，将发出重新验证请求来用新的值填充缓存，如果你刷新页面，就可以看到新值
export async function getServerSideProps({ req, res }) {
  res.setHeader(
    "Cache-Control",
    "public, s-maxage=10, stale-while-revalidate=59"
  );

  return {
    props: {},
  };
}
```

##### getServerSideProps 渲染一个错误页面吗

如果 `getServerSideProps` 抛出一个错误，将会展示 `pages/500.js` file。查看 500 page 文档去学习如何创建，在开发期间，这个文件不会被使用，而是显示开发覆盖图(报错)

[500 page](https://nextjs.org/docs/advanced-features/custom-error-page#500-page)

#### getStaticPaths

如果一个页面有动态路由并且需要使用 `getStaticPros`，那它需要定义一个 path 列表用于静态生成
当你从一个用了动态路由的页面 export 一个 `getStaticPaths` 方法时，Next.js 会静态地预渲染所有 `getStaticPaths` 列出的路径

```js
export async function getStaticPaths () {
	return {
		paths: [
			{ params: { ... }}
		],
		fallback: true // false or 'blocking'
	}
}
```

`getStaticPaths` 必须和 `getStaticProps` 一起使用，不能和 `getServerSideProps` 一起使用

[getStaticPaths API reference](https://nextjs.org/docs/api-reference/data-fetching/get-static-paths)里包含了 `getStaticPaths` 可以使用的所有参数和属性

##### 什么时候使用 getStaticPaths TODO

如果静态生成预渲染页面使用了动态路由并且：

- 数据来自无头 CMS
- 数据来自数据库
- 数据来自文件
- 数据可以公开缓存（不是用于特定用户的）
- 页面必须被预渲染（用于 SEO）并且非常快 —— `getStaticProps` 生成 HTML 和 JSON 文件，这两个文件都可以通过 CDN 缓存以提升性能

##### getStaticPaths 在什么时候运行

`getStaticPaths` 只会在生产包？构建时运行，不会在运行时调用，您可以使用这个 [工具](https://next-code-elimination.vercel.app/)验证从客户端包中删除的 getStaticPath 内写的代码

`getStaticProps` 如何和 `getStaticPaths` 一起运行的：

-

##### 在什么地方可以使用 getStaticPaths

##### 开发环境中每个请求都会运行
