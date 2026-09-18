---
title: 三维可视化开发
date: 2026-03-26 21:00:00
updated: 2026-03-27 10:00:00
type: page
comments: false
toc: false
---

本专栏聚焦 **三维可视化工程能力**：从渲染管线、场景组织到交互与性能优化，沉淀可复述、可展示的项目经验。

## 在线演示

**[Web 3D 协同同步编辑器](/visualization/editor/)**（Three.js + Firebase）

- 页面与 glTF 模型托管在 GitHub Pages（纯静态）
- 物体状态经 Firebase Realtime Database 跨设备同步
- 仅同步 `position / rotation / scale / color / modelId`，不同步相机
- 两边打开同一 `?room=` 链接，看到「Firebase 已连接」且在线 ≥ 2 即可验证

**[三维算法实验室](/visualization/algorithm-lab/)**（工业标注 / 剖切 / 空间索引）

- 建筑结构由梁、柱、楼板和核心筒等独立构件组成，支持构件选择与高亮
- 工业标注绑定构件 ID、局部锚点、表面法线和三角面索引
- 支持标注编辑、删除、本地持久化及 JSON 导入/导出
- 标注按 `rooms/{room}/industrialAnnotations/{annotationId}` 存储，并通过 Firebase 多端实时同步
- 同时展示剖切/截面和体素空间索引
- 适合用于项目演示、面试陈述和技术验证
- 不依赖后端，使用纯前端 Three.js 方案，便于演示和复现

相关文章：

- [实作：GitHub Pages 上的 Web 3D 协同编辑器](/2026/03/26/实作-GitHub-Pages-Web3D协同编辑器/)
- [原理详解：Web 端多端协同如何在无后端下工作](/2026/03/27/原理详解-Web端多端协同如何在无后端下工作/)

## 写作目标

- 把可视化项目拆成面试可讲的技术点（架构、性能、取舍）
- 覆盖 Web 三维与相关工程实践，形成可持续更新的能力地图
- 用真实问题驱动内容，而不是堆砌概念

## 计划主题

1. 渲染基础：相机、坐标系、材质与光照
2. 场景工程：模型加载、层级管理、资源与内存
3. 交互与体验：拾取、编辑、漫游与状态同步
4. 性能优化：Draw Call、LOD、批处理、帧预算
5. 工程落地：与业务系统集成、调试与可观测性

## 相关文章

请前往分类页查看本专栏全部文章：

[三维可视化开发 · 文章列表](/categories/visualization/)
