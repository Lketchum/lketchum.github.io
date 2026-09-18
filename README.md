# Personal Blog / Site

这是一个基于 Hexo 的个人博客项目，使用 npm 管理依赖，并通过 Hexo 生成/预览静态站点。

## 1. 安装依赖

在项目根目录执行：

```bash
npm install
```

如果是全新环境，第一次启动前必须先执行此命令。它会根据 `package.json` 中的依赖安装 Hexo 和主题相关模块。

## 2. 清理缓存（可选）

```bash
npm run clean
```

用于清理 Hexo 生成缓存和 public 目录，适合在更新文章、修改主题配置后使用。

## 3. 构建站点

```bash
npm run build
```

该命令会执行：

```bash
hexo generate
```

生成静态页面，输出到 `public/` 目录。

## 4. 启动本地预览

```bash
npm run server
```

该命令会执行：

```bash
hexo server
```

启动本地开发服务器，默认地址为：

```text
http://localhost:4000
```

如果端口被占用，可以使用：

```bash
npx hexo server -p 8080
```

然后访问：

```text
http://localhost:8080
```

## 5. 常用操作顺序

首次使用时推荐按顺序执行：

```bash
npm install
npm run clean
npm run build
npm run server
```

## 6. 说明

- 该项目的 `package.json` 中定义了常用脚本：
  - `build`: 生成静态站点
  - `clean`: 清理缓存
  - `deploy`: 部署站点
  - `server`: 本地启动服务
- 若本地依赖未安装，直接运行 `npm run server` 或 `npm run build` 可能会失败。

## 7. Windows 终端示例

在 PowerShell 或 CMD 中，进入项目目录后执行：

```powershell
cd E:\SelfProject\lketchum.github.io
npm install
npm run build
npm run server
```

如果需要更多帮助，可以继续扩展此文档。
