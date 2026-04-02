# Fclock

基于 [Plasmo](https://docs.plasmo.com/) 与 **Vue 3** 的浏览器扩展：在新标签页展示**翻页风格**的数字时钟（时、分、秒）与当前日期。

## 功能说明

- 打开新标签页时显示大号翻页数字时钟（24 小时制）
- 数字块带上下分区渐变、中间接缝与阴影，贴近实体翻页钟观感
- 翻页动画由 `FlipDigit` 组件在数字变化时触发

## 技术栈

- Plasmo（Chrome Manifest V3）
- Vue 3 + TypeScript

## 目录结构（主要文件）

| 路径 | 说明 |
|------|------|
| `tabs/newtab.vue` | 新标签页页面：布局、时间/日期逻辑 |
| `tabs/newtab.css` | 新标签页全局补充样式（翻页数字样式在组件内） |
| `components/FlipDigit.vue` | 单个翻页数字：样式与翻页动画 |

## 本地开发

```bash
pnpm install
pnpm dev
```

在 Chrome 中打开 **扩展程序 → 开发者模式 → 加载已解压的扩展程序**，选择项目下的开发构建目录，例如：

`build/chrome-mv3-dev`

修改源码后，在扩展管理页点击「重新加载」或使用 Plasmo 提供的自动刷新流程。

## 生产构建

```bash
pnpm build
```

产物位于 `build/chrome-mv3-prod`，可用于上架或自行打包分发。

打包为 zip（若已配置 Plasmo 的 package 流程）：

```bash
pnpm package
```

## 上架与自动化

仓库中的 `.github/workflows/submit.yml` 可与 Plasmo 文档中的 [提交流程](https://docs.plasmo.com/framework/workflows/submit) 配合使用；首次上架仍需在应用商店后台完成资质与首版信息配置。

## 参考文档

- [Plasmo 文档](https://docs.plasmo.com/)
