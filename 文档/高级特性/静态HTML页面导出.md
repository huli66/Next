### 静态 HTML 页面导出

`next export` 允许您将 Next.js 应用程序导出到可以独立运行不依赖 Node.js 服务器的静态 HTML，如果你不需要使用服务器不支持的功能可以直接使用 `next export`

如果你想建立一个只有一些预渲染静态 HTML 的混合网站，Next.js 已经自动实现了这个功能

package.json

```json
"script": {
	"build": "next build && next export"
}
```

执行 `npm run build` 会自动生成一个 out 目录

`npm export` 为你的 APP 构建了一个 HTML 版本的。`next build` 的过程中，`getStaticProps` 和 `getStaticPaths` 将会为 pages 目录下的每个页面生成一个 HTML 文件。然后 `next export` 会赋值已经导出了的文件到对应目录，`getInitialProps` 会在 `npm export` 期间生成 HMTL 文件，而不是 `next build` 期间
_（个人理解：用了 getStaticProps 或 getStaticPaths 到页面在 next build 过程中生成，用 getInitialProps 的页面在 npm export 过程中生成）_

对于更高级的场景，你可以在 `next.config.js` 文件里定义一个 `exportPathMap` 参数以确切配置将生成哪些页面

#### 支持的功能

Next.js 构建静态站点所需的大多数核心功能都支持，包括：

- 使用 `getStaticPaths` 时的动态路由
- 使用 `next/link` 预抓取页面
- 预加载 JavaScript
- 动态 imports
- 任何样式选项（例如：CSS 模块，styled-jsx）
- 客户端获取数据
- `getStaticProps`
- `getStaticPaths`
- 使用自定义加载对图像进行优化 Image

#### 不支持的功能

依赖 Node.js 服务支持的功能，或者不能在构建进程中计算出的动态逻辑都不被支持

- Image 优化（默认下载）
- 国际化路由
- API 路由
- 重写 Rewrites
- Redirects
- Headers
- Middleware
- Incremental Static Regeneration
- `fallback: true`
- `getServerSideProps`

#### getInitialProps

可以用 `getInitialProps` 替代 `getStaticProps` 但是有一些注意点：

- `getInitialProps` 在任意给定页面不能和 `getStaticProps` 或 `getStaticPaths` 一起使用，而不是用 `getStaticPaths` 你将需要在 `next.config.js` 里配置 `exportPathMap` 参数来告诉 exporter 哪些 HTML 文件应该被导出
- 当 `getInitialProps` 在导出时调用，`context` 参数里的 `res` 和 `req` 字段将会时空对象，因为导出期间没有服务器运行
- `getInitialProps` 每次客户端导航时调用，如果你希望只在构建时获取数据，请换成 `getStaticProps`
- `getInitialProps` 应该从一个 API 获取数据，不能像 `getStaticProps` 那样使用 Node.js 特定的库或文件系统

建议尽可能从 `getInitialProps` 迁移到 `getStaticProps`
