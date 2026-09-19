# Blog Development Backlog

> 面向简历/面试的内容与能力建设清单。  
> 状态：`done` 已完成 · `wip` 进行中 · `todo` 待做 · `later` 以后再说  
> 更新：2026-09-19

---

## 一、已完成总结

### 1. 站点框架装修

| 项 | 说明 | 状态 |
| --- | --- | --- |
| 三专栏导航 | 三维可视化 / 金融高性能 / 开发随笔 | done |
| 专栏入口页 | `/visualization/` `/finance/` `/notes/` | done |
| 首页模块卡 | 首屏三入口，非纯文章列表 | done |
| About 重写 | 能力方向表 + 专栏链接 | done |
| 分类 URL | `category_map` → visualization/finance/notes | done |
| 侧栏专栏导航 | `source/_data/sidebar.njk` | done |
| TOC 乱结构修复 | 去掉重复 H1，统一 `##`，页面 `toc: false` | done |
| 本地构建链路 | clean / build / server；补齐 word-counter | done |

### 2. 三维可视化 · 已落地

| 项 | 路径/文章 | 状态 |
| --- | --- | --- |
| Web 3D 协同编辑器 | `/visualization/editor/` | done |
| glTF 模型加载 | Helmet / Duck（本地 `.glb`） | done |
| Firebase 跨设备同步 | `meshState` + `presence` | done |
| 防环 / 相机解耦 | `isRemoteUpdate`；不同步相机 | done |
| 三维算法实验室 | `/visualization/algorithm-lab/` | done |
| 工业标注 + Firebase | 构件锚定、导入导出、多端同步 | done |
| 剖切 / 体素索引演示 | algorithm-lab 内 | done |
| 实作文 | 《实作：GitHub Pages 上的 Web 3D 协同编辑器》 | done |
| 原理文 | 《原理详解：Web 端多端协同如何在无后端下工作》 | done |
| 专栏导读 | 《专栏导读：三维可视化开发》 | done |
| CT3D 项目理解 | 核对 MATLAB / C# 数据流、现有能力与边界 | done |
| CT3D 项目详情页 | `/visualization/ct3d/`，含架构、验证与 roadmap | done |
| CT3D 系列文章 | DICOM/HU、MPR、迁移验证、Oblique/Slab 共四篇 | done |
| CT3D 站点入口 | 首页、三维专栏、About 作品卡 | done |
| CT3D 可信证据 | 29 个测试、Golden Fixture、Benchmark、.NET CI | done |
| 三维专栏卡片导航 | 核心项目 / Web3D 实验 / 文章能力地图 | done |
| 后续项目规划页 | Industrial Geometry Pipeline / GPU Volume Ray Casting | done |

### 3. 金融高性能 · 已有内容

| 项 | 说明 | 状态 |
| --- | --- | --- |
| 专栏入口 + 导读 | 能力地图框架 | done |
| 《金融量化分析》 | 工具链入门（偏浅，待升级） | done |

### 4. 开发随笔 · 已有内容

| 项 | 说明 | 状态 |
| --- | --- | --- |
| 专栏入口 + 导读 | 写作模板 | done |
| ChatGPT 网站 / 机制分析 / 理财整理 等 | 历史短文，已归入随笔分类 | done |

### 5. 架构取舍记录（已验证）

- ❌ 公共 y-webrtc 信令：同浏览器易通，跨 Mac 不稳  
- ❌ 公共 y-websocket demo 中继：「在线」常为 1  
- ✅ Firebase Realtime Database：不自建服务器，跨设备可用  
- ✅ 页面仍部署 GitHub Pages，不必迁 Firebase Hosting  

---

## 二、Backlog（欲开发）

### A. 三维可视化（优先 · 面试作品）

| ID | 内容 | 产出形式 | 优先级 | 状态 |
| --- | --- | --- | --- | --- |
| V-01 | 协同编辑器：选中态 / 远端光标 / 操作者标识 | demo 增强 | P1 | todo |
| V-02 | 协同编辑器：房间鉴权或收紧 Firebase Rules | 工程 hardening | P1 | todo |
| V-03 | 文章：glTF 加载管线与资源生命周期 | 长文 | P1 | todo |
| V-04 | 文章：TransformControls + 状态同步防环实战 | 长文（可并入原理文补篇） | P2 | todo |
| V-05 | 文章：工业标注数据模型（构件 ID / 法线 / 三角面） | 长文 + 链 lab | P1 | todo |
| V-06 | 文章：剖切/截面在 BIM/工业场景的用途与实现要点 | 长文 | P2 | todo |
| V-07 | 文章：体素/空间索引为何能加速拾取 | 长文 | P2 | todo |
| V-08 | algorithm-lab：性能面板（面数、拾取耗时） | demo 增强 | P2 | todo |
| V-09 | 项目卡片：可公开项目「背景/难点/方案/结果/复盘」挂 About | 页面 | P1 | done |
| V-10 | LOD / 批处理 / 帧预算专题（可先短文后 demo） | 文 ± demo | P2 | later |
| V-11 | 与业务系统集成：REST 拉构件树 + 前端展示 | 文 ± demo | P3 | later |
| V-12 | CT3D：梳理 DICOM→HU→MPR/Oblique/Slab 数据流与能力边界 | 代码理解/验证记录 | P0 | done |
| V-13 | CT3D：建立 MATLAB/C# 跨实现输入、参数与期望输出基准 | 验证资产 | P0 | done |
| V-14 | CT3D：DICOM/HU、MPR、迁移验证、Oblique/Slab 系列文章 | 四篇长文 | P0 | done |
| V-15 | CT3D：旗舰项目详情页与站点入口 | 页面 | P0 | done |
| V-16 | CT3D：固定环境采集加载、重采样与 Slab 性能基线 | 基准记录 | P1 | todo |
| V-17 | CT3D：补充不含患者数据的合成体 MPR 示意 | 项目页素材 | P2 | done |
| V-20 | CT3D：数据许可与去标识确认后补真实程序录屏 | 项目页素材 | P2 | later |
| V-18 | Industrial 3D Geometry Pipeline：架构、里程碑、自研算法与验收蓝图 | 规划详情页 | P1 | done |
| V-19 | GPU Volume Ray Casting：CPU/CUDA 对照、优化与性能证据蓝图 | 规划详情页 | P1 | done |
| V-21 | Industrial 3D Geometry Pipeline：创建独立仓库并完成 MVP | 独立项目（实施队列 1） | P2 | later |
| V-22 | GPU Volume Ray Casting：旗舰 MVP 稳定后创建独立仓库 | 独立项目（实施队列 2） | P2 | later |
| V-23 | 三维专栏改为核心项目 / Web3D 实验 / 文章三组整卡导航 | 页面改造 | P1 | done |

> 两个规划页只展示技术蓝图，不代表项目代码已经存在。实施顺序固定为 V-21 → V-22；CPU/GPU BVH 留在旗舰项目进阶阶段，不另建重复项目。

### B. 金融高性能（优先 · 职业方向）

| ID | 内容 | 产出形式 | 优先级 | 状态 |
| --- | --- | --- | --- | --- |
| F-01 | 文章：如何定义并测量延迟（p50/p99、尾延迟） | 长文 | P0 | todo |
| F-02 | 文章：C#/.NET 并发与异步模型取舍 | 长文 | P0 | todo |
| F-03 | 文章：序列化与内存布局对吞吐的影响 | 长文 | P1 | todo |
| F-04 | 文章：幂等、对账、故障隔离（资金正确性） | 长文 | P1 | todo |
| F-05 | 小实验：BenchmarkDotNet 对比两种实现 | 文 + 代码仓库链接 | P1 | todo |
| F-06 | 升级《金融量化分析》：从工具列表改为工程约束叙事 | 改稿 | P2 | todo |
| F-07 | 行情/回测/执行链路中的工程边界（与量化交界） | 长文 | P2 | later |

### C. 开发随笔（持续 · 侧写）

| ID | 内容 | 产出形式 | 优先级 | 状态 |
| --- | --- | --- | --- | --- |
| N-01 | 随笔模板固化：现象 → 缩小范围 → 结论 → 下次更快 | 规范 | P2 | todo |
| N-02 | Firebase Rules / 静态站接入 BaaS 踩坑短记 | 短文 | P2 | todo |
| N-03 | Hexo NexT TOC/侧栏踩坑与修复 | 短文 | P3 | todo |
| N-04 | 历史短文择优升级或归档（机制分析类） | 整理 | P3 | later |

### D. 站点与作品呈现

| ID | 内容 | 优先级 | 状态 |
| --- | --- | --- | --- |
| S-01 | About「项目与作品」补齐可点击作品卡 | P1 | done |
| S-02 | 首页/专栏页增加「最新更新」三条 | P2 | todo |
| S-03 | Giscus 评论（可选，GitHub Discussions） | P3 | later |
| S-04 | README 补充专栏结构与 Firebase 说明 | P2 | wip |
| S-05 | 推送 GitHub Pages，确保线上与本地一致 | P0 | todo |

---

## 三、建议推进顺序（近 4 周）

1. **V-16** 在固定环境记录并解释 MPR、Oblique 与 Slab 性能基线
2. **V-05 + V-02** 补工业标注文章，并收紧 Firebase Rules
3. **F-01 / F-02** 保留金融主线，完成延迟测量与 .NET 并发两篇硬文章
4. **V-21 → V-22** 后续按顺序实施，不并行开发；创建仓库前保持“规划中”状态

---

## 四、维护约定

- 新想法先加到本文件对应分区，标 `todo`  
- 做完把状态改为 `done`，并在「已完成总结」补一行  
- 文章尽量带：背景 → 方案对比 → 实现 → 结果/坑 → 面试怎么讲  
- Demo 优先可公网访问、可复制链接验证  

相关入口：

- 三维：[visualization](./source/visualization/index.md)  
- 金融：[finance](./source/finance/index.md)  
- 随笔：[notes](./source/notes/index.md)  
- 协同编辑器：`source/visualization/editor/`  
- 算法实验室：`source/visualization/algorithm-lab/`
