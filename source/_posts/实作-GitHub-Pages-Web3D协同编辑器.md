---
title: 实作：GitHub Pages 上的 Web 3D 协同编辑器
date: 2026-03-26 22:55:00
categories: 三维可视化开发
tags:
  - Three.js
  - Yjs
  - WebRTC
  - 协同编辑
---

在纯静态 GitHub Pages 上实现可交互的 3D 物体协同编辑：Three.js 负责渲染与操作轴，Yjs CRDT + y-webrtc 负责状态同步。

<!-- more -->

## 在线体验

打开演示页：[Web 3D 协同同步编辑器](/visualization/editor/)

建议用两个浏览器标签页进入同一房间（默认 `lketchum-viz-demo`，也可自定义 `?room=`）。

## 架构要点

1. **不传 Mesh 二进制**：只同步 `meshState`（position / rotation / scale / materialColor）
2. **防环**：`isRemoteUpdate` 标志避免远端更新再次写入 Yjs
3. **相机解耦**：OrbitControls 仅本地使用，不进入共享状态
4. **纯前端**：ES Modules + importmap 从 CDN 加载依赖

## 为什么适合面试表达

它把「可视化交互」和「分布式状态一致性」放在同一条故事线里：你改的是物体，不是视角；冲突由 CRDT 合并，而不是锁。
