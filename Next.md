# Next.js

## 安装

```sh
npx create-next-app@latest
npx create-next-app@latest --typescript
```

### 手动创建 Next.js 项目

```SH
mkdir NextDemo
# 把文件初始化成可管理的项目（根目录里添加一个 package.json 文件）
npm init

# 安装需要的依赖包 react react-dom next
npm install --save react react-dom next
yarn add react react-dom next

# 增加常用快捷命令，把常用命令行工具的配置到 package.json 中
# 创建 pages 文件夹和文件
# 在根目录下创建 pages 文件夹，Next.js 会自动创建对应的路由

```

```JS
// 常用命令配置
"script": {
  "dev": "next",
  "build": "next build",
  "start": "next start"
}
```

**在根目录下创建 pages 文件夹，Next.js 会自动创建对应的路由**

### 路由

- Link 标签跳转
