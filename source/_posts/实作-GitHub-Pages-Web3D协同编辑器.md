---
title: 实作：GitHub Pages 上的 Web 3D 协同编辑器
date: 2026-03-26 22:55:00
updated: 2026-03-27 10:00:00
categories: 三维可视化开发
tags:
  - Three.js
  - Firebase
  - 协同编辑
  - glTF
---

在纯静态 GitHub Pages 上实现可交互的 3D 物体协同编辑：Three.js 负责渲染与操作轴，Firebase Realtime Database 负责跨设备状态同步。

<!-- more -->

## 在线体验

打开演示页：[Web 3D 协同同步编辑器](/visualization/editor/)

建议步骤：

1. 打开链接并确认 **「Firebase 已连接」**
2. 点「复制链接」，在另一浏览器 / Mac / 手机打开同一 `?room=`
3. 看到 **「在线」≥ 2** 后，拖拽坐标轴或切换模型验证同步
4. 两侧分别旋转视角：物体一致，相机独立

## 架构要点

1. **页面与模型**：GitHub Pages 静态托管（含本地 `.glb`）
2. **共享状态**：只同步 `meshState` JSON（TRS / 颜色 / modelId），不传 Mesh 二进制
3. **传输**：Firebase Realtime Database 路径 `rooms/{room}/meshState`
4. **在线人数**：`rooms/{room}/presence/{clientId}` + `onDisconnect`
5. **防环**：`isRemoteUpdate` 避免远端回调再次写回
6. **相机解耦**：OrbitControls 仅本地，不进共享态

## 和「部署到 Firebase」的关系

不需要把网站部署到 Firebase Hosting。  
GitHub Pages 继续托管前端；Firebase 只提供数据库。

## 为什么适合面试表达

一条完整故事线：

- 可视化：glTF 加载、TransformControls、PBR 环境光  
- 工程：状态与视图分离、防环、节流写入  
- 系统：静态托管 + BaaS 实时通道，而不是自建 Node 房间服  

更完整的原理说明见：

[原理详解：Web 端多端协同如何在无后端下工作](/2026/03/27/原理详解-Web端多端协同如何在无后端下工作/)
