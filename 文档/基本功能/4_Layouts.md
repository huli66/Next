# 布局

我们在 Next.js 中引入了改进的布局支持，阅读 [Layouts RFC](https://nextjs.org/blog/layouts-rfc) 了解更多

React 模式允许我没解构一个页面为一系列的组件。很多组件都可以在不同页面中复用，例如，每个页面都应该有同样的导航栏和 footer

```js
// components/layout.js
import Navbar from './navbar'
import Footer from './footer'

export default function Layout({ children }) {
  return (
    <>
      <Navbar />
      <main>{children}</main>
      <Footer />
    </main>
  )
}

```

## 例子

### 自定义应用的单一共享布局

如果你整个应用只有一个 layout，你可以创建一个 Custom App，并且用这个布局包装你的应用，因为 `<Layout />` 组件在更改页面时被复用，所以它可以保持状态

```js
// pages/_app.js
import Layout from "../components/layout";

export default function MyApp({ Component, pageProps }) {
  return (
    <Layout>
      <Component {...pageProps} />
    </Layout>
  );
}
```

### 每页布局

如果你需要多个布局，你可以给你的页面添加一个 `getLayout` 属性，允许你为这个布局返回一个 React 组件。这样允许您在每个页面的基础上定义布局，因为我们返回的是一个函数，所以我们可以在需要的时候使用复杂嵌套布局

```js
// page/index.js
import Layout from "../components/layout";
import NestedLayout from "../components/nested-layout";

export default function Page() {
  return {
    /** content */
  };
}

Page.getLayout = function getLayout(page) {
  return (
    <Layout>
      <NestedLayout>{page}</NestedLayout>
    </Layout>
  );
};
```

```js
// pages/_app.js
export default function MyApp({ Component, pageProps }) {
  // 如果可以，请使用在页面级定义的布局
  const getLayout = Component.getLayout || ((page) => page)

  return getLayout(<Component { ...pageProps }>);
}
```

当在页面间导航时，我们想保持页面数据（输入框值，滚动条位置等），实现一个 SPA 体验

这个布局模式让状态可以持久化，因为 React 组件树在页面导航时会保持，借助组件树，React 可以了解哪些元素发生了改变来保持状态

_注意：React 了解哪些元素发生了改变的这个过程被称为 和解 （[reconciliation](https://reactjs.org/docs/reconciliation.html)）_

### 使用 TypeScript

使用 TypeScript 时，你必须先为你的页面创建一个新的包含一个 `getLayout` 函数的类型。然后你必须创建一个新的类型，该类型重写 Component 属性以使用以前创建的类型（看代码）

```js
// pages/index.tsx
import type { ReactElement } from 'react'
import Layout from '../components/layout'
import NestedLayout from '../components/nested-layout'
import type { NextPageWithLayout } from './_app'

const Page: NextPageWithLayout = () => {
  return <p>hello world</p>
}

Page.getLayout = functio getLayout(page: ReactElement) {
  return (
    <Layout>
      <NestedLayout>{page}</NestedLayout>
    </Layout>
  )
}

export default Page
```

```js
// pages/_app.tsx
import type { ReactElement, ReactNode } from "react";
import type { NextPage } from "next";
import type { AppProps } from "next/app";

export type NextPageWithLayout = NextPage & {
  getLayout?: (page: ReactElement) => ReactNode,
};

type AppPropsWithLayout = AppProps & {
  Component: NextPageWithLayout,
};

export default function MyApp({ Component, pageProps }: AppPropsWithLayout) {
  // 尽可能使用页面级定义的布局
  const getLayout = Component.getLayout ?? ((page) => page);

  return getLayout(<Component {...pageProps} />);
}
```

### 数据获取

在你的布局中，你可以在客户端用 `useEffect` 或者 SWR 之类的库获取数据。因为这个文件不是一个页面，此时你不可以使用 `getStaticProps` 或者 `getServerSideProps`

```js
// components/layout.js
import useSWR from "swr";
import Navbar from "./navbar";
import Footer from "./footer";

export default function Layout({ childre }) {
  const { data, error } = useSWR("/api/navigation", fetcher);

  if (error) return <div>Failed to load</div>;
  if (!data) return <div>Loading...</div>;

  return (
    <>
      <Navbar links={data.links} />
      <main>{children}</main>
      <Footer />
    </>
  );
}
```

更多请看 Pages 和 Custom App 部分
