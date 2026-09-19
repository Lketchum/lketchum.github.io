---
title: GPU Volume Ray Casting
date: 2026-09-19 15:00:00
type: page
comments: false
toc: true
---

<div class="project-plan-hero">
  <div>
    <p class="project-feature__eyebrow">Deep Project Blueprint</p>
    <h2>CPU Reference 与 CUDA 体绘制</h2>
    <p>以同一相机、传递函数和采样规则对照 CPU/GPU 输出，用图像误差与性能数据证明 GPU 优化，而不是只展示渲染效果。</p>
  </div>
  <span class="project-status project-status--planned">规划中</span>
</div>

> 本页是公开项目蓝图，不代表 CUDA renderer 已经实现。CT3D 提供体数据与采样知识基础，但本项目将保持独立仓库和独立证据链。

## 为什么选择这个项目

Industrial 3D Geometry Pipeline 已经覆盖 BVH 与网格空间查询。如果再单独做 CPU/GPU BVH，能力证明会重复。GPU Volume Ray Casting 可以补齐另一组能力：

- 从 CT3D 复用体素、spacing、窗宽窗位和三线性采样知识；
- 新增 CUDA kernel、GPU 内存与 profiling；
- 同时覆盖医学可视化、科学计算和图形性能；
- 能建立清晰的 CPU 正确性基线和 GPU 加速证据。

## 数据流

<div class="pipeline-flow">
  <span>合成体 / 公开体数据</span>
  <b>→</b>
  <span>CPU Reference</span>
  <b>↔</b>
  <span>CUDA Renderer</span>
  <b>→</b>
  <span>PNG 输出</span>
  <b>→</b>
  <span>RMSE / SSIM / 像素差</span>
  <b>+</b>
  <span>Render Time / VRAM / Speedup</span>
</div>

首版采用 headless renderer，固定输入和相机直接输出 PNG 与 JSON 报告。交互 Viewer 放在数值对照和 benchmark 稳定之后，避免 UI 掩盖算法问题。

## 建议仓库结构

```text
gpu-volume-ray-casting/
├── src/
│   ├── core/               # Volume, camera and transfer-function model
│   ├── cpu/                # Reference ray marcher
│   ├── cuda/               # CUDA kernels and device resources
│   └── cli/                # Reproducible rendering commands
├── tests/
│   ├── fixtures/           # Synthetic and licensed public volumes
│   └── golden/             # Reference images and metadata
├── benchmarks/
│   ├── manifests/          # Volume, image, step and camera parameters
│   └── reports/            # Machine-readable results
├── viewer/                 # Optional, after headless baseline
└── docs/
```

## MVP 算法

### CPU Reference

- 射线与体包围盒求交；
- 固定步长 ray marching；
- 三线性体素采样；
- 标量到 RGBA 的传递函数；
- front-to-back alpha compositing；
- 固定相机和背景，输出确定性 PNG。

CPU 版本优先保证易读、可测和确定性，不以最快实现为目标。

### CUDA Renderer

- 与 CPU 使用相同相机、采样步长和传递函数；
- 明确 host/device 数据布局与拷贝时机；
- 先完成直接 global-memory 基线，再评估 3D texture；
- 内核计时与数据传输计时分开记录；
- 不用仅有 FPS 的交互窗口代替正确性验证。

## 优化顺序

1. **Baseline**：global memory + 固定步长，不做提前终止；
2. **Early ray termination**：累计不透明度达到阈值后停止；
3. **Texture sampling**：评估 3D texture 插值、缓存与数值差异；
4. **Sampling study**：比较不同步长的图像误差和耗时；
5. **Empty-space skipping**：建立空间占用结构后跳过空区域；
6. **交互 Viewer**：只有 headless 输出和 benchmark 稳定后再增加。

每个优化必须保留优化前版本，报告图像误差、耗时变化和适用条件。

## 测试数据

- 数学定义的 sphere、多个高斯体和分层 ramp；
- 与 CT3D Golden Fixture 相似的可预测小体；
- 许可清晰的公开体数据；
- 不提交真实病人数据，不把数据去标识假设当作许可证明。

建议至少覆盖 `128³`、`256³` 和显存允许时的 `512³`；输出分辨率和采样步长必须写入 benchmark manifest。

## 正确性验证

<div class="evidence-grid">
  <div class="evidence-block">
    <span class="evidence-block__label">Pixel Agreement</span>
    <strong>CPU/GPU 同参数输出</strong>
    <p>报告最大绝对误差、RMSE、SSIM 和差值图；明确浮点精度、插值和纹理采样造成的允许差异。</p>
  </div>
  <div class="evidence-block">
    <span class="evidence-block__label">Analytic Cases</span>
    <strong>可预测的合成体</strong>
    <p>常量体、空体、单点、球体和边界射线用于验证包围盒、采样、合成与背景行为。</p>
  </div>
  <div class="evidence-block">
    <span class="evidence-block__label">Regression</span>
    <strong>Golden Image + 参数清单</strong>
    <p>固定相机、传递函数、步长和输出尺寸；CI 检查图像误差，不比较肉眼截图。</p>
  </div>
  <div class="evidence-block">
    <span class="evidence-block__label">Profiling</span>
    <strong>分离传输和 kernel</strong>
    <p>使用 CUDA Event 与 profiler 区分 H2D、kernel、D2H，避免把一次性数据传输隐藏在 FPS 中。</p>
  </div>
</div>

## 性能报告要求

每份结果同时记录：

- CPU、GPU、内存和操作系统；
- GPU 驱动、CUDA Toolkit、编译器与 Release 配置；
- 体数据尺寸、输出分辨率、采样步长和传递函数；
- warm-up、迭代次数与统计方法；
- CPU render time、GPU kernel time、端到端时间、显存和加速比；
- 对应图像的 RMSE/SSIM，避免以降低质量换取未说明的加速。

FPS 只用于持续交互渲染；headless 基准以每帧毫秒和吞吐量为主。

## 分阶段里程碑

### G0：CPU 可复现基线

- 完成 Volume/Camera/TransferFunction 数据模型；
- 合成体、包围盒求交、三线性采样和 compositing 测试；
- 固定输入输出 Golden Image。

### G1：CUDA 正确性版本

- 完成最小 kernel 与 host/device 管线；
- 在多个合成体上达到约定误差；
- 建立 Release benchmark 命令和环境报告。

### G2：性能优化

- 依次加入 early termination、texture sampling 和采样步长实验；
- 每次优化保留 before/after 报告；
- 使用 profiler 说明瓶颈，而不是凭经验猜测。

### G3：空间跳跃与交互

- 研究 empty-space skipping；
- 增加可选交互 Viewer；
- 与 CT3D 或静态站只共享演示产物，不合并仓库职责。

## 完成标准

- CPU reference 与 CUDA renderer 都能通过一条命令运行；
- 至少一个 CUDA 优化有正确性不退化的量化证据；
- 多数据规模下报告图像误差、render time、显存和加速比；
- 能解释 ray-box、三线性采样、alpha compositing 和 early termination；
- 能解释 warp、访存、分支和 CPU/GPU 同步成本；
- README、fixture、Golden Image、测试、CI 和性能报告完整；
- 能在 15 分钟内讲清误差来源、性能瓶颈和优化取舍。

## 简历使用条件

在 G1 完成前，只能描述为 CUDA 学习计划。G2 完成并公开 CPU/GPU 对照后，才适合写：

> 独立实现 CPU reference 与 CUDA Volume Ray Casting，在相同相机、采样和传递函数下进行 RMSE/SSIM 对照，并通过多体数据规模 benchmark 分析 kernel、传输和端到端性能。

具体加速比、FPS 和误差只能引用实际报告，不能使用目标值。
