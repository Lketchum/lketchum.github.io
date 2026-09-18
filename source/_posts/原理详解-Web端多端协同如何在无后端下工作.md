---
title: 原理详解：Web 端多端协同如何在无后端下工作
date: 2026-03-27 08:00:00
updated: 2026-03-27 10:00:00
categories: 三维可视化开发
tags:
  - Firebase
  - Three.js
  - 协同编辑
  - Realtime Database
  - glTF
---

本文对应演示页：[Web 3D 协同同步编辑器](/visualization/editor/)。  
目标是把 **多端协同在 GitHub Pages 静态站上如何落地** 讲清楚：同步什么、不传什么、数据怎么走、为何不自建服务器，以及如何跨电脑稳定验证。  
（这里的「无后端」指 **无自建 Node 房间服**；实时存储使用 Firebase 托管服务。）

<!-- more -->

## 如何验证多端协同

1. 两边打开**同一链接**（推荐已部署的 GitHub Pages；房间名用 `?room=` 保持一致）
2. 左上角状态显示 **「Firebase 已连接」**
3. **「在线」≥ 2**（presence 人数）
4. 一侧拖拽坐标轴 / 改颜色 / 切模型，另一侧应跟上
5. 两侧分别转视角：物体一致，**相机互不影响**

### 演进说明（面试可讲取舍）

| 版本 | 传输方式 | 问题 |
| --- | --- | --- |
| 初版 | y-webrtc 公共信令 | 同浏览器标签页常能通；换 Mac 经常失败 |
| 过渡 | y-websocket + 公共 demo 中继 | 仍依赖不稳定的公共服务，「在线」常为 1 |
| **当前** | **Firebase Realtime Database** | 不自建服务器，跨设备更稳 |

结论：**表现层与状态结构设计可复用；跨设备稳定性取决于传输层是否可控。**

## 一句话架构

> **GitHub Pages 托管页面与模型；Firebase 只存一份小 JSON 状态；各端各自用 Three.js 渲染。**

```text
浏览器 A（Three.js）                 浏览器 B（Three.js）
      │                                    │
      │  写/听 meshState JSON               │
      └──────────► Firebase RTDB ◄──────────┘
                   rooms/{room}/...
```

| 层 | 技术 | 职责 |
| --- | --- | --- |
| 表现层 | Three.js + OrbitControls + TransformControls | 本地渲染、编辑、独立相机 |
| 状态层 | `meshState` JSON | position / rotation / scale / color / modelId |
| 传输层 | Firebase Realtime Database | 路径监听与多端推送 |
| 托管层 | GitHub Pages | `index.html` + `.glb` 静态资源 |

注意：网站**不必**部署到 Firebase Hosting；Firebase 在这里只当数据库。

## 为什么不必自建 Node 服务器

静态站缺的是「有状态、可推送的共享存储」，不是「再买一台 VPS」。

- **GitHub Pages**：负责 HTML/JS/模型文件  
- **Firebase**：负责 `rooms/{roomId}/meshState` 的读写与实时回调  
- 业务上仍然 **不传 Mesh 二进制**，信道压力很小  

这和「评论系统用 GitHub Issues / Giscus」是同一类思路：静态页 + 托管后端能力。

## 同步什么，不同步什么

### 同步（写入 Firebase）

```text
rooms/{room}/meshState
├─ position: [x, y, z]
├─ rotation: [x, y, z]
├─ scale:    [x, y, z]
├─ materialColor: "#rrggbb"
└─ modelId: "helmet" | "duck"

rooms/{room}/presence/{clientId}
└─ at / ua   # 仅用于在线人数
```

### 不同步

- 相机位置与 OrbitControls target  
- glTF 几何与贴图（各端本地加载 `./models/*.glb`）  
- UI 面板状态  

## 数据流

### 本地 → Firebase

```text
拖拽 TransformControls
    → 读取 modelRoot TRS
    → set(rooms/{room}/meshState, json)   # 拖拽中节流，松手立即写
```

### Firebase → 本地

```text
onValue(meshState)
    → 若 modelId 变化：GLTFLoader 换模
    → 应用 position/rotation/scale/color 到 modelRoot
    → 不改相机
```

### 在线人数

```text
set(presence/{clientId})
onDisconnect(...).remove()
onValue(presence) → 统计 key 数量
```

## 防死循环：`isRemoteUpdate`

```text
本地拖拽 → 写 Firebase → onValue 回调到自己
若回调里再触发写回 → 抖动/环路
```

做法：

- 远端应用状态时：`isRemoteUpdate = true`  
- 此期间禁止 `writeMeshStateToFirebase`  
- 应用结束后复位标志  

这是 UI 事件与同步事件的隔离，和用不用 Yjs 无关。

## 相机为何解耦

OrbitControls =「我怎么看」；TransformControls =「物体怎么被改」。  
只同步物体状态，多人才能各自绕物体观察，而不会被对方视角拖走。

## 模型加载与协同的分工

- 静态站提供 `DamagedHelmet.glb` / `Duck.glb`  
- 协同只传 `modelId` + TRS  
- 换模后仍挂在同一 `modelRoot` 上，坐标系不乱  

## 安全说明（演示项目）

当前可用测试规则（公开读写）方便演示；上线长期站点应改为鉴权或房间级规则，避免被刷写。  
Web 端 `apiKey` 本来就会暴露在前端，防护主要靠 **Database Rules**，不是藏 key。

## 面试口述（1 分钟）

1. 我在 GitHub Pages 上做了 Web 3D 协同编辑器  
2. Three.js 负责渲染与编辑；共享态是一份小 JSON  
3. 用 Firebase Realtime Database 做跨设备实时同步，不自建服务器  
4. 只同步 TRS/颜色/模型 ID，不同步相机和 Mesh  
5. 用 `isRemoteUpdate` 防止回写环路；模型本地加载以控制带宽  

## 相关入口

- 演示：[Web 3D 协同同步编辑器](/visualization/editor/)  
- 实作：[实作：GitHub Pages 上的 Web 3D 协同编辑器](/2026/03/26/实作-GitHub-Pages-Web3D协同编辑器/)  
- 专栏：[三维可视化开发](/visualization/)  
- 代码：`source/visualization/editor/index.html` + `firebase-config.js`
