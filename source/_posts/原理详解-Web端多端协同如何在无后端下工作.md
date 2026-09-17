---
title: 原理详解：Web 端多端协同如何在无后端下工作
date: 2026-03-27 08:00:00
categories: 三维可视化开发
tags:
  - Yjs
  - CRDT
  - WebRTC
  - 协同编辑
  - Three.js
---

本文对应演示页：[Web 3D 协同同步编辑器](/visualization/editor/)。  
目标不是“再写一遍用法”，而是把 **多端协同在纯静态站点上为什么可行、数据怎么走、冲突怎么解、为什么不会死循环** 讲清楚，方便面试时按链路口述。

<!-- more -->

## 如何验证多端协同

可以，而且很直接：

1. 两边都打开**线上同一地址**（推荐 GitHub Pages，不要一边 localhost、一边另一台机器却进了不同页面）
2. 确认房间名完全一致（例如都用 `?room=demo-room`），可点「复制链接」发给另一台
3. 等待状态显示 **「中继已连接」**，且 **「在线」≥ 2**
4. 一侧拖拽坐标轴 / 改颜色 / 切模型，另一侧应跟上
5. 两侧分别转视角：物体一致，相机互不影响

### 为什么「本机两个标签行，换 Mac 就不行」

旧版用 `y-webrtc` 时：

- **同浏览器多标签** 往往靠 `BroadcastChannel` 就能通（不依赖公网信令）
- **换一台 Mac** 必须走 WebRTC，还要公共 **signaling** 帮两边互相发现  
  而 Yjs 默认公共信令经常不稳定/不可用，于是跨设备会表现为“完全不同步”

当前版本已改为 **`y-websocket` + 公共中继 `wss://demos.yjs.dev`**：  
状态经中继转发，跨电脑/跨网络更稳；CRDT 合并与 `isRemoteUpdate` 防环逻辑不变。

若仍不同步，按下面排查：

1. 两边是否都显示「中继已连接」
2. 房间名是否一字不差
3. 是否都打开了**已部署的最新页面**（旧缓存可能还是 webrtc 版）
4. 网络是否拦截 WebSocket（部分公司网）——可点「重新连接」或换手机热点再试

## 一句话架构

> **每个客户端各自渲染 3D；协同层只同步一份可合并的状态文档。**

拆开就是三层：

| 层 | 技术 | 职责 |
| --- | --- | --- |
| 表现层 | Three.js + OrbitControls + TransformControls | 本地渲染、拾取、编辑、独立相机 |
| 状态层 | Yjs `Y.Map`（CRDT） | 保存可合并的共享状态 |
| 传输层 | y-websocket（公共中继） | 把文档增量送达同房间客户端 |

GitHub Pages 只托管 `index.html` 和 `.glb`，**不托管房间服务器、不托管数据库**。

## 为什么“无后端”也能协同

常见协同有两条路：

1. **中心化**：客户端 → 你的服务器 → 广播给其他人  
2. **去中心化**：客户端彼此同步，服务器最多只做“介绍认识”（信令）

本项目走第 2 条：

- **Yjs** 让每端都有完整文档副本，并用 CRDT 保证最终一致
- **y-websocket** 把增量发到中继，再转发给同房间其他人  
  （早期 demo 也曾用 y-webrtc；但公共 WebRTC 信令不稳定，跨设备易失败，因此改为中继）
- 我们仍然**不自建业务数据库**：中继只转发 update，不理解 3D 业务含义

因此静态托管足够：缺的不是“网页服务器”，而是“有状态的业务后端”；而业务状态被 CRDT 文档本身承担了。

## 同步什么，不同步什么

### 同步（进入 `meshState`）

```text
meshState (Y.Map)
├─ position: [x, y, z]
├─ rotation: [x, y, z]
├─ scale:    [x, y, z]
├─ materialColor: "#rrggbb"
└─ modelId: "helmet" | "duck"
```

设计原则：**禁止在协同信道里传 Mesh 二进制**。  
模型文件各端本地加载（`./models/*.glb`）；信道只传“选了哪个模型 + 变换/颜色”。

### 不同步

- 相机 pose（位置、朝向、OrbitControls target）
- 帧率、渲染参数、UI 面板开合
- glTF 几何与贴图本体

这样多人可以围着同一物体讨论，而不会出现“你一转视角，我的屏幕也被拽走”。

## 数据流：本地修改如何到达远端

```text
用户拖拽 TransformControls
        │
        ▼
objectChange / dragging-changed
        │
        ▼
读取 modelRoot.position/rotation/scale
写入 yMeshState（Y.Map）
        │
        ▼
Yjs 生成 document update（增量）
        │
        ▼
y-websocket 发到中继，再转发给同房间 peers
        │
        ▼
对端 yMeshState.observe()
        │
        ▼
更新对端 modelRoot（不改对端相机）
```

关键点：

1. **写入的是状态，不是操作录像**  
   远端不需要重放“拖了 120 帧”，只要最终（以及中间采样）状态一致。

2. **拖拽过程中持续写入**  
   用 `objectChange` 流式更新，观感接近实时；松手时再保证一次提交。

3. 房间名 = 文档隔离键  
   `new WebsocketProvider(server, 'lketchum-viz/' + roomName, ydoc)` 中，同名房间才会互通。

## 反向流：远端更新如何应用到本地

```js
yMeshState.observe(() => {
  // 1) 如 modelId 变化，先切换本地模型
  // 2) 再应用 position/rotation/scale/color
});
```

这里有两个实现细节：

### 1. 模型切换与变换解耦

- `modelId` 变化 → `GLTFLoader` 加载对应 glb  
- 变换字段变化 → 只改 `modelRoot` 的 TRS  

同步根节点是 `Group`（`modelRoot`），真正 glTF 场景是它的子节点。这样换模型不会把“协同坐标系”搞乱。

### 2. 初始化完成后才接收远端

若一进页就 `observe`，又同时 `await loadModel()`，容易出现 **并发加载互相取消**（token 抢占）。  
因此用 `syncReady`：本地首模加载完成后再处理远端 observe。

## 冲突怎么解决：CRDT 在这里的角色

多人同时拖同一物体时，传统做法常是：

- 加锁（谁持有编辑权）  
- 或服务器最后写入覆盖  

Yjs 的 `Y.Map` 属于 CRDT 家族：每个更新都带因果/并发信息，副本之间交换后能收敛到同一结果，**不需要中心锁**。

对本项目的直观理解：

- 你改 `position`，我几乎同时改 `rotation` → 合并后两者都在  
- 两人同时改 `position` → 按 CRDT 规则收敛到一致值（可能不是“算术平均”，而是可确定的合并结果）

面试时可以说：

> 协同的一致性不靠后端锁，而靠状态副本的可交换合并；传输层只负责把 update 送达。

## 防死循环：`isRemoteUpdate`

最容易踩的坑：

```text
本地拖拽 → 写 Yjs → 自己又 observe 到 → 再写 Yjs → 环路/抖动
```

或：

```text
远端更新 → 改 Mesh → 触发 controls change → 再写回 Yjs → 来回拉锯
```

解法是单方向闸门：

```text
本地用户操作：
  isRemoteUpdate === false 时，才允许 writeMeshStateToYjs()

远端 observe：
  isRemoteUpdate = true
  应用 TRS / 颜色
  isRemoteUpdate = false
```

这不是分布式算法，而是 **UI 事件与同步事件的隔离**。几乎所有“可视化 + 文档同步”系统都会有等价机制。

## 相机为何必须解耦

OrbitControls 管的是“我如何看”，TransformControls 管的是“世界里物体如何被改”。

若同步相机：

- 体验上变成强制观影  
- 网络抖动会直接造成眩晕感  
- 冲突语义也不清晰（视角不是业务对象）

所以共享态只覆盖 **scene object state**，不覆盖 **view state**。

## 传输层说明：为什么改用 WebSocket 中继

`y-webrtc` 的理想路径是 P2P：信令只负责“介绍”，数据直连。  
但对个人博客演示很不友好：

1. 公共 signaling 经常挂  
2. 跨 NAT / 运营商网络还可能需要 TURN  
3. 结果就是：**同电脑标签页能同步，换 Mac 不能**

因此演示改为：

```text
浏览器 A ──WSS──▶ demos.yjs.dev ──WSS──▶ 浏览器 B
```

代价是依赖公共中继可用性；收益是跨设备成功率明显高于纯公共 WebRTC。  
若以后要完全自主，可自建 `y-websocket` 服务或私有 signaling，业务层代码几乎不用改。

## 和“模型加载”的关系

协同并不负责“把模型文件发过去”：

- 静态站提供 `DamagedHelmet.glb` / `Duck.glb`  
- 各端按 `modelId` 自己加载  
- 加载完成后套上共享 TRS  

收益：

- 带宽小（状态是字节级/百字节级更新）  
- 可缓存  
- 信道更稳  

代价：

- 各端必须能访问同一套模型资源  
- 新增模型要同时发资源与 `modelId` 约定  

## 面试时可怎么讲（1 分钟版）

1. 我在静态站上做了个 Web 3D 协同编辑器  
2. Three.js 负责本地渲染与编辑；Yjs 存共享物体状态；y-websocket 经中继同步到各端  
3. 只同步 TRS/颜色/模型 ID，不同步相机和 Mesh 二进制  
4. 用 `isRemoteUpdate` 防止本地与远端事件环路  
5. 一致性交给 CRDT；跨设备稳定性依赖可达的中继，而不是自建业务库  

## 相关演示与代码入口

- 演示：[Web 3D 协同同步编辑器](/visualization/editor/)  
- 实作速览：[实作：GitHub Pages 上的 Web 3D 协同编辑器](/2026/03/26/实作-GitHub-Pages-Web3D协同编辑器/)  
- 源码单文件：`source/visualization/editor/index.html`

后续若继续深化，可以往这些方向扩展：多人光标/选中态、操作撤销栈（Yjs UndoManager）、权威端校验、自建信令与 TURN。
