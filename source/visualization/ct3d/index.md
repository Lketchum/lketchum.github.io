---
title: CT3D 医疗体数据可视化
date: 2026-09-19 14:00:00
type: page
comments: false
toc: true
---

## 项目要解决什么

CT3D 是一个医疗体数据可视化学习工程。目标不是堆叠渲染效果，而是把 **DICOM 输入如何变成可信的空间视图** 讲清楚、做出来并验证：像素值先转换为 HU，体素按几何信息组织，再经过窗宽窗位、重采样与投影生成可观察结果。

项目保留两套互补实现：

- **MATLAB**：快速验证数学过程，承载教学演示与高层可视化集成。
- **C# .NET 8 WPF**：把读取、领域模型、重采样、显示和测试拆开，验证桌面工程化路径。

FBP/DBT 是仓库中的独立重建学习线；它解决“如何从投影重建图像”，不等同于 MPR 解决的“如何从既有体数据重采样视图”。

## 数据流

`DICOM 文件 → 元数据与像素读取 → 序列排序 → HU 转换 → 三维体数据 → MPR/Oblique/Slab 采样 → 窗宽窗位 → 2D 显示`

<figure class="project-media">
  <img src="/images/ct3d-synthetic-mpr.svg" alt="CT3D 合成体的轴位、冠状位和矢状位 MPR 示意">
  <figcaption>合成体方向示意：不含患者数据，也不是 WPF 运行截图；用于解释三平面方向与联动索引。</figcaption>
</figure>

关键约束贯穿整条链路：

1. 排序不能只依赖文件名，应使用可用的空间/序列信息建立切片顺序。
2. `PixelSpacing` 与层间距决定体素的物理尺度，不能默认三个方向等距。
3. `RescaleSlope` 与 `RescaleIntercept` 决定存储值到 HU 的映射。
4. 窗宽窗位是显示映射，不应破坏原始 HU 体数据。
5. Oblique 与 Slab 会访问非整数坐标或多个采样位置，必须定义插值、边界和聚合规则。

## MATLAB 与 C# 能力对照

<div class="metric-grid">
  <div class="metric-block">
    <span class="metric-block__label">MATLAB 原型</span>
    <strong>DICOM → HU、窗宽窗位、三平面 MPR</strong>
    <p>包含斜切教学，并集成等值面和 <code>volshow</code>，用于观察算法与高层可视化效果。</p>
  </div>
  <div class="metric-block">
    <span class="metric-block__label">C# / WPF 工程</span>
    <strong>序列读取、HU、正交 MPR、Slab 与 Oblique</strong>
    <p>支持正交 Slab mean/MIP、绕 X 轴斜切和三线性采样；现有 29 个自动化测试。</p>
  </div>
</div>

两套实现用于交叉理解和结果对照，但不把 MATLAB 的等值面或 `volshow` 集成包装成 C# 已有能力。

## 架构拆分

### 输入与领域层

负责 DICOM 单帧文件读取、必要标签解析、序列组织、像素解码与 HU 转换。当前输入范围明确限定为 **单帧/文件、8/16-bit 单色、非压缩 DICOM**。

### 体数据与几何层

用体素数组及 spacing 描述数据，在索引坐标与物理尺度之间建立明确关系。正交 MPR 是固定轴切片；Oblique 则从输出平面的坐标反推到体数据坐标。

### 采样与投影层

非整数位置采用三线性插值。Slab 沿法向获取多个样本，再按 mean 或 MIP 聚合。越界策略、采样步长和厚度语义都应可说明、可测试。

### 展示层

WPF 承载视图与交互，窗宽窗位把 HU 映射为显示灰度。显示层不负责 DICOM 解码，也不修改底层 HU 数据。

## 验证策略

<div class="evidence-grid">
  <div class="evidence-block">
    <span class="evidence-block__label">确定性测试</span>
    <strong>从小体数据验证坐标与聚合</strong>
    <p>使用值可预测的体素，检查轴向切片、插值点、mean/MIP 和越界行为。</p>
  </div>
  <div class="evidence-block">
    <span class="evidence-block__label">跨实现验证</span>
    <strong>共享 Golden Fixture，容差 1e-4</strong>
    <p>固定 3 × 3 × 3 合成体、三种正交切面与 30° 斜切；MATLAB 与 C# 使用同一输入和期望输出。</p>
  </div>
  <div class="evidence-block">
    <span class="evidence-block__label">真实数据检查</span>
    <strong>检查元数据与解剖连续性</strong>
    <p>在合法可用数据上检查排序、方向、层间连续性和窗宽窗位；不以“看起来像”替代数值验证。</p>
  </div>
  <div class="evidence-block">
    <span class="evidence-block__label">性能基线</span>
    <strong>已提供可重复 Benchmark</strong>
    <p>固定合成体和迭代参数，输出 axial、oblique 与 slab 的总耗时和平均耗时；正式对外数字仍需同时记录硬件环境。</p>
  </div>
</div>

## 当前边界

- C# 端当前没有原生 VTK、Marching Cubes、3D 网格或体绘制。
- DICOM 支持单帧/文件、8/16-bit 单色、非压缩；多帧、彩色与压缩传输语法不在当前范围。
- C# Oblique 当前围绕 X 轴，不表示任意姿态重切已经完成。
- Slab 当前提供 mean 与 MIP，不将其描述为完整的临床后处理工作站。
- MATLAB 中的等值面与 `volshow` 是集成能力，不等于自研渲染内核。
- 仓库用于学习和工程验证，不用于临床诊断。

## 仓库与运行入口

- 公开仓库：[Lketchum/CT3D](https://github.com/Lketchum/CT3D)
- 获取代码：`git clone https://github.com/Lketchum/CT3D.git`
- C# 测试：在仓库根目录执行 `dotnet test csharp/CT3D.sln`。
- 性能基准：执行 `dotnet run --project csharp/CT3D.Benchmarks --configuration Release -- --iterations 20`。
- C# WPF：在仓库根目录执行 `dotnet run --project csharp/CT3D.App`。
- MATLAB 工作站：在 MATLAB 桌面中进入 `matlab/`，执行 `p05_ct_slice_viewer`；数据准备和工具箱依赖以仓库 `matlab/README.md` 为准。

页面使用明确标注的合成体示意避免患者数据风险；可复现证据仍以代码、共享 Golden Fixture、测试和上述运行入口为准。真实程序录屏只会在数据许可和去标识确认后补充。

## 配套文章

1. [从 DICOM 到 HU：排序、Spacing 与窗宽窗位](/2026/09/19/CT3D-DICOM到HU-排序Spacing与窗宽窗位/)
2. [MPR 几何：从 MATLAB 原型到 C# 三线性采样](/2026/09/19/CT3D-MPR几何与三线性采样/)
3. [C# 迁移：分层、测试与跨实现验证](/2026/09/19/CT3D-CSharp迁移的分层测试与跨实现验证/)
4. [斜切与 Slab/MIP：正确性、性能与边界](/2026/09/19/CT3D-斜切与Slab-MIP/)

## Roadmap

1. 在固定硬件环境记录并解释性能基线，持续扩大 Golden Fixture 的边界覆盖。
2. 扩展 Oblique 的姿态表达，并加强 spacing、边界与交互联动测试。
3. 评估更广的 DICOM 传输语法与多帧支持，先定义测试数据和兼容边界。
4. 在独立实验中研究 Marching Cubes、网格与 GPU 体绘制；完成前不写入 C# 当前能力。
5. 保持 FBP/DBT 重建学习线独立，分别说明输入、目标和验证标准。
