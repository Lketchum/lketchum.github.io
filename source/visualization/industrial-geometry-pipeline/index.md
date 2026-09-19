---
title: Industrial 3D Geometry Pipeline
date: 2026-09-19 14:50:00
type: page
comments: false
toc: true
---

<div class="project-plan-hero">
  <div>
    <p class="project-feature__eyebrow">Flagship Project Blueprint</p>
    <h2>工业三维几何端到端管线</h2>
    <p>连接 C++ 几何内核、C# 服务与 Three.js 展示，用可验证的自研算法避免项目停留在 Open CASCADE API 调用层。</p>
  </div>
  <span class="project-status project-status--planned">规划中</span>
</div>

> 本页是公开项目蓝图，不代表对应功能已经实现。实际成果只会在代码、测试与 benchmark 完成后更新。

## 项目目标

建立一条可复现的工业模型处理链：

<div class="pipeline-flow">
  <span>STEP 模型</span>
  <b>→</b>
  <span>Open CASCADE 导入</span>
  <b>→</b>
  <span>B-Rep 拓扑</span>
  <b>→</b>
  <span>三角网格</span>
  <b>→</b>
  <span>自研 BVH / 查询</span>
  <b>→</b>
  <span>glTF + JSON</span>
  <b>→</b>
  <span>C# API</span>
  <b>→</b>
  <span>Three.js</span>
</div>

项目重点不是重新实现完整 CAD 内核，而是理解精确几何与离散网格之间的边界，并对空间查询、正确性和性能负责。

## 建议仓库结构

```text
industrial-3d-geometry-pipeline/
├── cpp/
│   ├── geometry-core/       # C++20 domain model and own algorithms
│   └── geometry-cli/        # Reproducible import/query/export commands
├── service/
│   └── Geometry.Api/        # ASP.NET Core jobs and artifact API
├── web/                     # Three.js model, topology and section viewer
├── tests/
│   └── fixtures/            # Licensed STEP and synthetic meshes
├── benchmarks/              # Fixed datasets and benchmark manifests
└── docs/                    # Architecture, decisions and reports
```

首版由 C# 服务以独立进程调用 C++ CLI。这样能隔离 native 崩溃、固定输入输出并保留命令行复现能力；只有性能数据证明进程边界成为瓶颈后，才评估 C ABI/P/Invoke。

## 技术职责边界

### Open CASCADE 负责

- STEP 读取与 B-Rep 几何/拓扑访问；
- 曲线曲面求值和基础三角化；
- 可作为截面、距离等运算的参考实现。

### 必须自己实现

- AABB 数据结构与射线包围盒测试；
- Ray-Triangle Intersection；
- BVH 构建、遍历与查询统计；
- BVH 查询与 brute-force 全量遍历的结果对照。

如果只调用 Open CASCADE 完成导入、剖切和导出，项目只能证明库集成，不能证明几何算法能力。

### 进阶自研候选

- 截面线段到闭合轮廓的连接与容差处理；
- 网格邻接关系和边界边识别；
- 点到三角形、点到网格距离；
- BVH broad phase + triangle-triangle narrow phase；
- CPU/GPU BVH 查询对照。

## 分阶段里程碑

### M0：工程骨架与数据约定

- CMake Presets、依赖说明、格式化、测试和 CI；
- 坐标系、长度单位、容差与 glTF 轴向约定；
- 许可清晰的 STEP fixture 和合成三角网格。

### M1：STEP 与 B-Rep

- 读取 STEP，输出 Solid/Shell/Face/Wire/Edge/Vertex 计数；
- 展示拓扑树和几何类型；
- 检查空模型、非流形、缺失单位和导入失败。

### M2：三角化与 glTF

- 将 B-Rep 面离散为带法线和 face ID 的网格；
- 导出 glTF 和拓扑映射 JSON；
- 在 Three.js 中按 B-Rep Face 选择与高亮。

### M3：自研 BVH

- 实现 AABB、Möller–Trumbore 射线三角形求交；
- 完成 median split 基线，再评估 SAH；
- 与 brute-force 逐射线比较命中三角形和距离；
- 报告构建时间、查询时间、节点数、深度和内存。

### M4：C# 服务与 Web 展示

- ASP.NET Core 提供上传、任务状态、产物和错误信息；
- C++ CLI 生成 glTF、拓扑 JSON 和 benchmark 结果；
- Three.js 展示模型、拓扑树、拾取与剖切结果。

### M5：几何分析扩展

- 截面提取与轮廓连接；
- 点/面或构件间距离；
- 碰撞检测 broad/narrow phase；
- CPU/GPU BVH 仅在 CPU 正确性基线稳定后启动。

## 正确性与性能证据

<div class="evidence-grid">
  <div class="evidence-block">
    <span class="evidence-block__label">Correctness</span>
    <strong>参考实现与可预测几何</strong>
    <p>使用三角形、立方体、退化面和 brute-force 查询验证命中、距离、拓扑映射与坐标转换。</p>
  </div>
  <div class="evidence-block">
    <span class="evidence-block__label">Robustness</span>
    <strong>容差与退化情况</strong>
    <p>覆盖平行射线、共面、零面积三角形、重复顶点、极小/极大模型和非流形输入。</p>
  </div>
  <div class="evidence-block">
    <span class="evidence-block__label">Performance</span>
    <strong>按模型规模报告</strong>
    <p>固定硬件下记录 STEP 导入、三角化、BVH 构建、查询延迟、内存和 glTF 体积。</p>
  </div>
  <div class="evidence-block">
    <span class="evidence-block__label">Reproducibility</span>
    <strong>CLI、fixture、CI</strong>
    <p>README 提供一键命令；公开输入、参数、期望结果和 benchmark manifest。</p>
  </div>
</div>

## 完成标准

- 有一个完整可运行的 C++20 三维项目，而不是算法片段合集；
- 至少一个核心算法由自己实现，并能脱离库解释和重写；
- 单元测试、Golden Fixture、CI 和固定硬件 benchmark 齐全；
- 能解释复杂度、浮点误差、容差和退化几何；
- GitHub README 能让面试官复现实验；
- 能在 15 分钟内讲清数据流、架构边界、失败实验和性能结果；
- 提供 30 秒演示，但演示不替代测试与指标。

## 简历使用条件

在 M3 完成前，只能写“项目规划/学习中”，不能写成已完成的几何算法项目。M4 完成并有可复现证据后，才适合描述为：

> 独立构建工业三维几何处理管线，完成 STEP/B-Rep 解析、三角化、glTF 输出及自研 BVH 拾取，并通过 brute-force 对照、自动化测试与多规模 benchmark 验证正确性和性能。

所有数字必须来自公开报告，不能先写目标值再当作结果。
