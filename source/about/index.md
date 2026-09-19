---
title: 关于
date: 2024-07-06 15:44:33
updated: 2026-03-26 21:00:00
type: about
toc: false
---

## 博客定位

当前技术发展方向，沉淀技能记录开发心得：

1. **[三维可视化开发](/visualization/)**：渲染、场景工程、交互与性能
2. **[金融高性能开发](/finance/)**：低延迟、高吞吐与金融系统工程实践
3. **[开发随笔](/notes/)**：问题复盘、工具实践与工程思考

目标不是“写完一篇文章”，而是把项目与学习过程整理成可展示、可追问的能力证据。

## 关于我

热爱技术、愿意把问题拆开做到底。本科与研究生均为土木工程背景，研究生方向涉及基于深度学习的图像识别；转行软件研发后，长期以 **C# / .NET 后端** 为主，并持续向三维可视化与金融高性能方向拓展。

## 个人经历

- 2015.09–2019.07：大连理工大学，土木工程（国际班），大连
- 2019.09–2022.08：同济大学，建筑与土木工程，上海
- 2022.08–2024.03：北京构力科技有限公司（武汉研发中心），Web 后端研发（C#），武汉
- 2024.06–至今：锐珂医疗 Carestream（上海全球研发中心），软件后端研发 SDE（C#），上海

## 能力方向（持续建设）

| 方向 | 关注点 | 对应专栏 |
| --- | --- | --- |
| 三维可视化 | 场景组织、渲染性能、交互工程化 | [三维可视化开发](/visualization/) |
| 金融高性能 | 延迟/吞吐、并发、可靠性 | [金融高性能开发](/finance/) |
| 工程基本功 | 排查路径、工具链、系统设计碎片 | [开发随笔](/notes/) |

## 项目与作品

<div class="project-grid">
  <article class="project-card">
    <div class="project-card__header">
      <h3>CT3D</h3>
      <span class="project-status project-status--active">持续验证</span>
    </div>
    <p class="project-card__summary">医疗体数据可视化学习工程，连接 MATLAB 算法原型与 C# .NET 8 WPF 工程实现。</p>
    <dl class="project-card__facts">
      <dt>背景</dt>
      <dd>建立从 DICOM 像素、HU 到 MPR、斜切和 Slab 的可解释处理链路。</dd>
      <dt>自研核心</dt>
      <dd>DICOM 序列读取、HU、窗宽窗位、正交 MPR、Slab mean/MIP、绕 X oblique 与三线性采样。</dd>
      <dt>验证</dt>
      <dd>共享合成体 Golden Fixture 通过 MATLAB/C# 双端验证；C# 当前有 29 个自动化测试，并提供可重复性能基准。</dd>
      <dt>边界</dt>
      <dd>C# 尚无原生 VTK、Marching Cubes、3D 网格和体绘制；DICOM 限单帧/文件、8/16-bit 单色、非压缩。</dd>
    </dl>
    <div class="project-card__links">
      <a href="/visualization/ct3d/">项目详情</a>
      <a href="https://github.com/Lketchum/CT3D">GitHub</a>
    </div>
  </article>

  <article class="project-card">
    <div class="project-card__header">
      <h3>Algorithm Lab</h3>
      <span class="project-status project-status--stable">在线演示</span>
    </div>
    <p class="project-card__summary">面向工业/BIM 场景的 Three.js 算法实验室。</p>
    <dl class="project-card__facts">
      <dt>背景</dt>
      <dd>把构件级选择、工业标注、剖切和空间索引放入同一可操作场景。</dd>
      <dt>自研核心</dt>
      <dd>构件锚点与法线标注、JSON 导入导出、剖切/截面、体素空间索引及 Firebase 同步。</dd>
      <dt>验证</dt>
      <dd>可在浏览器操作并通过同房间链接检查跨端标注同步。</dd>
      <dt>边界</dt>
      <dd>定位为前端技术验证，不代表完整 BIM 平台或生产级权限系统。</dd>
    </dl>
    <div class="project-card__links">
      <a href="/visualization/algorithm-lab/">打开演示</a>
      <a href="/visualization/">专栏说明</a>
    </div>
  </article>

  <article class="project-card">
    <div class="project-card__header">
      <h3>Web 3D 协同编辑器</h3>
      <span class="project-status project-status--stable">在线演示</span>
    </div>
    <p class="project-card__summary">部署在 GitHub Pages、以 Firebase 为实时通道的轻量三维协同实验。</p>
    <dl class="project-card__facts">
      <dt>背景</dt>
      <dd>验证纯静态托管条件下，多设备编辑同一 Three.js 对象的最小闭环。</dd>
      <dt>自研核心</dt>
      <dd>TRS/颜色/模型状态同步、在线状态、防回写环与本地相机解耦。</dd>
      <dt>验证</dt>
      <dd>两个设备进入同一房间后，可分别验证共享物体状态和独立相机。</dd>
      <dt>边界</dt>
      <dd>只同步有限对象状态，不是通用场景协作协议；权限与冲突处理仍需增强。</dd>
    </dl>
    <div class="project-card__links">
      <a href="/visualization/editor/">打开演示</a>
      <a href="/2026/03/26/实作-GitHub-Pages-Web3D协同编辑器/">实现文章</a>
    </div>
  </article>
</div>
