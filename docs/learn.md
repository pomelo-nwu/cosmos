# Cosmos 项目学习笔记

## 项目概述

Cosmos 是一个基于 WebGL 的力导向图布局算法和渲染引擎。它的核心特点是将计算和绘制都在 GPU 的片段和顶点着色器中进行，避免了昂贵的内存操作，从而能够实时模拟包含数十万节点和连接的网络图。

### 项目特点

- **GPU 加速**：使用 WebGL 在 GPU 上进行计算和渲染
- **高性能**：能够处理大规模图形数据
- **力导向布局**：使用物理模拟来排列图形节点
- **可定制性**：提供丰富的配置选项

## 项目结构

### 核心文件和目录

- `src/index.ts` - 主要入口文件，定义了 `Graph` 类
- `src/config.ts` - 配置选项
- `src/modules/` - 各种功能模块

  - `Points/` - 处理节点
  - `Lines/` - 处理连接
  - `ForceManyBody/` - 处理力导向算法
  - `ForceLink/` - 处理连接力
  - `Zoom/` - 处理缩放功能
  - `Drag/` - 处理拖拽功能
  - `Clusters/` - 处理节点聚类
  - `GraphData/` - 处理图数据
  - `Store/` - 状态管理
  - 其他功能模块...

- `src/stories/` - 包含各种使用示例
  - `beginners/` - 基础示例
  - `clusters/` - 集群示例
  - `geospatial/` - 地理空间示例
  - `experiments/` - 实验性功能示例

## 核心 API

### 创建图形

```typescript
import { Graph } from "@cosmograph/cosmos";

const div = document.querySelector("div");
const config = {
  // 配置选项
};

const graph = new Graph(div, config);
```

### 设置数据

```typescript
// 设置节点位置 [x1, y1, x2, y2, ...]
graph.setPointPositions(pointPositions);

// 设置连接 [sourceIndex1, targetIndex1, sourceIndex2, targetIndex2, ...]
graph.setLinks(links);

// 设置节点颜色 [r1, g1, b1, a1, r2, g2, b2, a2, ...]
graph.setPointColors(pointColors);

// 设置节点大小 [size1, size2, ...]
graph.setPointSizes(pointSizes);

// 设置连接颜色
graph.setLinkColors(linkColors);

// 设置连接宽度
graph.setLinkWidths(linkWidths);

// 设置连接箭头
graph.setLinkArrows(linkArrows);

// 设置连接强度
graph.setLinkStrength(linkStrength);
```

### 集群功能

```typescript
// 设置节点集群
graph.setPointClusters(pointClusters);

// 设置集群位置
graph.setClusterPositions(clusterPositions);

// 设置集群强度
graph.setPointClusterStrength(clusterStrength);
```

### 操作和渲染

```typescript
// 渲染图形
graph.render();

// 缩放
graph.zoom(0.9);

// 适应视图
graph.fitView();

// 开始/暂停/重启模拟
graph.start();
graph.pause();
graph.restart();

// 单步执行模拟
graph.step();
```

### 选择和交互

```typescript
// 通过索引选择点
graph.selectPointByIndex(index);

// 选择多个点
graph.selectPointsByIndices(indices);

// 取消选择所有点
graph.unselectPoints();

// 获取选择的点索引
graph.getSelectedIndices();

// 获取相邻点索引
graph.getAdjacentIndices(index);

// 设置焦点点
graph.setFocusedPointByIndex(index);
```

### 坐标和位置

```typescript
// 空间坐标转屏幕坐标
graph.spaceToScreenPosition(spacePosition);

// 屏幕坐标转空间坐标
graph.screenToSpacePosition(screenPosition);

// 获取点位置
graph.getPointPositions();

// 获取集群位置
graph.getClusterPositions();
```

## 配置选项

主要配置选项包括：

- `spaceSize` - 空间大小
- `backgroundColor` - 背景颜色
- `pointSize` - 点大小
- `pointColor` - 点颜色
- `linkWidth` - 连接宽度
- `linkColor` - 连接颜色
- `curvedLinks` - 是否使用曲线连接
- `enableDrag` - 是否启用拖拽
- `enableZoom` - 是否启用缩放
- 模拟相关配置：
  - `simulationLinkDistance` - 连接距离
  - `simulationLinkSpring` - 连接弹性
  - `simulationRepulsion` - 排斥力
  - `simulationGravity` - 重力
  - `simulationDecay` - 衰减

## 使用场景

1. 网络分析和可视化
2. 社交网络分析
3. 复杂系统可视化
4. 大规模数据关系展示
5. 科学数据可视化

## 已知问题

- iOS 15.4 及以上版本不支持关键 WebGL 扩展 `EXT_float_blend`，这影响了 Cosmos 的多体力实现。

## 参考链接

- 官方网站：[cosmograph.app](https://cosmograph.app)
- 项目文档：[Cosmos Documentation](https://cosmograph-org.github.io/cosmos/?path=/docs/welcome-to-cosmos--docs)
- 示例演示：[StackBlitz Example](https://stackblitz.com/edit/how-to-use-cosmos?file=src%2Fmain.ts)

## WebGL 图可视化学习指南

本章节将从零开始，为前端工程师讲解如何理解和使用基于 WebGL 的图可视化库 Cosmos。我们将逐步分析源码，了解其设计理念和实现方式。

### 第一步：理解 WebGL 基础概念

**WebGL 是什么？**

WebGL（Web Graphics Library）是一种基于 JavaScript 的 API，允许在兼容的网页浏览器中渲染 3D 和 2D 图形，无需使用插件。它利用 GPU 进行计算和渲染，能够处理大量数据。

**为什么图可视化库使用 WebGL？**

传统的基于 DOM 或 SVG 的图可视化在处理大规模数据（如数千或数万个节点）时，性能会急剧下降。WebGL 直接利用 GPU 并行处理能力，可以高效地渲染和计算大量图形元素。

**关键概念：**

1. **着色器（Shader）**：在 GPU 上运行的小程序，分为顶点着色器（处理位置计算）和片段着色器（处理颜色计算）
2. **缓冲区（Buffer）**：存储图形数据（如点的位置、颜色等）
3. **纹理（Texture）**：在 WebGL 中不仅用于图像，还可以用于存储和传递数据
4. **帧缓冲对象（Framebuffer）**：允许渲染到纹理而非屏幕

### 第二步：理解力导向图的原理

**力导向图是什么？**

力导向图是一种用物理模拟来布局图的方法。它将图的节点视为带电粒子，连接视为弹簧，通过模拟物理力让图形自动排列为视觉上美观的布局。

**核心力模型：**

1. **排斥力（Repulsion）**：节点之间相互排斥，防止重叠
2. **吸引力（Attraction）**：连接的节点之间相互吸引
3. **重力（Gravity）**：将所有节点吸引到中心，防止分散
4. **阻尼力（Damping）**：逐渐减少节点的运动，使系统稳定

### 第三步：深入理解 Cosmos 架构

#### 模块化设计

Cosmos 采用高度模块化的设计，将不同功能拆分为独立的模块：

- **Graph 类**：主要入口点，协调各模块工作
- **Store**：管理全局状态，如点位置、大小、颜色等
- **Points**：处理图中的节点渲染和交互
- **Lines**：处理图中的连接线渲染
- **ForceManyBody**：实现节点间的排斥力
- **ForceLink**：实现连接节点间的吸引力
- **Clusters**：处理节点聚类

每个模块都遵循类似的模式：

1. 初始化纹理和缓冲区
2. 创建 WebGL 着色器程序
3. 提供运行和更新方法

#### 关键实现方式

**Graph 类**

Graph 类是用户与库交互的主要入口点。它负责初始化各种模块并在更新时协调它们。下面是 Graph 类的核心部分：

```typescript
export class Graph {
  public config = new GraphConfig();
  public graph = new GraphData(this.config);
  private canvas: HTMLCanvasElement;
  private store = new Store();
  private points: Points;
  private lines: Lines;
  private forceGravity: ForceGravity | undefined;
  private forceCenter: ForceCenter | undefined;
  private forceManyBody: ForceManyBody | ForceManyBodyQuadtree | undefined;
  private forceLinkIncoming: ForceLink | undefined;
  private forceLinkOutgoing: ForceLink | undefined;
  private forceMouse: ForceMouse | undefined;
  private clusters: Clusters;

  // 构造函数初始化各个模块
  public constructor(div: HTMLDivElement, config?: GraphConfigInterface) {
    // 初始化配置
    if (config) this.config.init(config);

    // 创建 canvas 元素
    this.store.div = div;
    const canvas = document.createElement("canvas");
    canvas.style.width = "100%";
    canvas.style.height = "100%";
    this.store.div.appendChild(canvas);

    // 初始化 WebGL 上下文
    this.reglInstance = regl({
      canvas: this.canvas,
      attributes: {
        antialias: false,
        preserveDrawingBuffer: true,
      },
      extensions: ["OES_texture_float", "ANGLE_instanced_arrays"],
    });

    // 初始化各个模块
    this.points = new Points(
      this.reglInstance,
      this.config,
      this.store,
      this.graph
    );
    this.lines = new Lines(
      this.reglInstance,
      this.config,
      this.store,
      this.graph,
      this.points
    );

    // 初始化力模型模块
    if (this.config.enableSimulation) {
      this.forceGravity = new ForceGravity(/*...*/);
      this.forceCenter = new ForceCenter(/*...*/);
      this.forceManyBody = this.config.useClassicQuadtree
        ? new ForceManyBodyQuadtree(/*...*/)
        : new ForceManyBody(/*...*/);
      this.forceLinkIncoming = new ForceLink(/*...*/);
      this.forceLinkOutgoing = new ForceLink(/*...*/);
      this.forceMouse = new ForceMouse(/*...*/);
    }
    this.clusters = new Clusters(/*...*/);
  }
}
```

### 第四步：理解 WebGL 在 Cosmos 中的应用

#### 数据表示方式

Cosmos 使用 WebGL 友好的数据结构：

1. **Float32Array**：表示点的位置、颜色、大小和连接

   ```typescript
   // 点位置 [x1, y1, x2, y2, ...]
   const pointPositions = new Float32Array([0, 0, 100, 100, 200, 200]);

   // 连接 [source1, target1, source2, target2, ...]
   const links = new Float32Array([0, 1, 1, 2]);
   ```

2. **纹理和帧缓冲对象**：存储和处理数据
   ```typescript
   // 在 Points 模块中创建位置纹理
   this.currentPositionFbo = reglInstance.framebuffer({
     color: reglInstance.texture({
       data: initialState,
       shape: [pointsTextureSize, pointsTextureSize, 4],
       type: "float",
     }),
     depth: false,
     stencil: false,
   });
   ```

#### 着色器程序

Cosmos 使用 GLSL 着色器程序来渲染点和线，以及执行力模拟计算：

**顶点着色器**：控制点的位置和大小

```glsl
// src/modules/Points/draw-points.vert (简化)
void main() {
  // 从纹理中获取点位置
  vec4 pointPosition = texture2D(positionsTexture, (textureCoords + 0.5) / pointsTextureSize);
  vec2 point = pointPosition.rg;

  // 转换点位置到规范化设备坐标
  vec2 normalizedPosition = 2.0 * point / spaceSize - 1.0;
  normalizedPosition *= spaceSize / screenSize;
  vec3 finalPosition = transformationMatrix * vec3(normalizedPosition, 1);
  gl_Position = vec4(finalPosition.rg, 0, 1);

  // 设置点大小
  gl_PointSize = calculatePointSize(size * sizeScale);

  // 传递颜色到片段着色器
  rgbColor = color.rgb;
  alpha = color.a;
}
```

**片段着色器**：控制点的外观

```glsl
// src/modules/Points/draw-points.frag (简化)
void main() {
    // 如果点完全透明，则丢弃片段
    if (alpha == 0.0) { discard; }

    // 计算点内的坐标
    vec2 pointCoord = 2.0 * gl_PointCoord - 1.0;
    // 计算到中心的平方距离
    float pointCenterDistance = dot(pointCoord, pointCoord);
    // 根据距离和平滑参数计算不透明度
    float opacity = alpha * (1.0 - smoothstep(smoothing, 1.0, pointCenterDistance));

    gl_FragColor = vec4(rgbColor, opacity);
}
```

#### 力模拟的实现

Cosmos 的力模拟算法是其最复杂也是最强大的部分。通过在 GPU 上并行计算力，它能够处理大规模的图。

**Barnes-Hut 四叉树算法**：

在 `ForceManyBody` 模块中，Cosmos 实现了基于 Barnes-Hut 四叉树的排斥力计算，这大大提高了大型图的性能：

```glsl
// src/modules/ForceManyBody/force-level.frag (简化)
void main() {
  vec4 pointPosition = texture2D(positionsTexture, textureCoords);
  float x = pointPosition.x;
  float y = pointPosition.y;

  // 根据层级调整边界
  // [计算代码...]

  vec4 velocity = vec4(vec2(0.0), 1.0, 0.0);

  // 基于邻近单元格计算额外速度
  for (float i = 0.0; i < 12.0; i += 1.0) {
    for (float j = 0.0; j < 4.0; j += 1.0) {
      // [计算代码...]
      velocity.xy += calculateAdditionalVelocity(vec2(n / cellSize, m / cellSize) / levelTextureSize, pointPosition.xy);
    }
  }

  gl_FragColor = velocity;
}
```

### 第五步：使用 Cosmos 创建基本图可视化

现在我们了解了 Cosmos 的基本架构，让我们创建一个简单的图可视化应用：

```typescript
import { Graph } from "@cosmograph/cosmos";

// 1. 创建 DOM 容器
const container = document.getElementById("graph-container");

// 2. 配置图形选项
const config = {
  spaceSize: 4096, // 图空间大小
  backgroundColor: "#2d313a", // 背景色
  pointSize: 4, // 点大小
  pointColor: "#4B5BBF", // 点颜色
  linkWidth: 0.1, // 连接宽度
  linkColor: "#5F74C2", // 连接颜色
  curvedLinks: true, // 曲线连接
  enableDrag: true, // 允许拖拽
  simulationLinkDistance: 1, // 连接距离
  simulationLinkSpring: 2, // 连接弹性
  simulationRepulsion: 0.2, // 排斥力
  simulationGravity: 0.1, // 重力
  simulationDecay: 100000, // 衰减
  onClick: (index) => {
    // 点击事件处理
    if (index !== undefined) {
      graph.selectPointByIndex(index);
      graph.zoomToPointByIndex(index);
    } else {
      graph.unselectPoints();
    }
  },
};

// 3. 创建图实例
const graph = new Graph(container, config);

// 4. 生成或加载数据
// 点位置：[x1, y1, x2, y2, x3, y3, ...]
const pointPositions = new Float32Array([
  100,
  100, // 点 1 位置
  200,
  150, // 点 2 位置
  150,
  250, // 点 3 位置
  300,
  300, // 点 4 位置
]);

// 连接：[source1, target1, source2, target2, ...]
const links = new Float32Array([
  0,
  1, // 连接点 0 -> 点 1
  1,
  2, // 连接点 1 -> 点 2
  2,
  3, // 连接点 2 -> 点 3
  3,
  0, // 连接点 3 -> 点 0
]);

// 5. 设置数据
graph.setPointPositions(pointPositions);
graph.setLinks(links);

// 6. 渲染图形
graph.render();

// 7. 开始模拟
graph.start();

// 其他可选操作：
// 适应视图：graph.fitView();
// 缩放：graph.zoom(0.8);
// 暂停模拟：graph.pause();
```

### 第六步：深入了解高级特性

#### 自定义节点和连接样式

Cosmos 允许为每个节点和连接设置不同的样式：

```typescript
// 自定义点颜色 [r1,g1,b1,a1, r2,g2,b2,a2, ...]
const pointColors = new Float32Array([
  255,
  0,
  0,
  1, // 点1: 红色
  0,
  255,
  0,
  1, // 点2: 绿色
  0,
  0,
  255,
  1, // 点3: 蓝色
  255,
  255,
  0,
  1, // 点4: 黄色
]);
graph.setPointColors(pointColors);

// 自定义点大小 [size1, size2, ...]
const pointSizes = new Float32Array([
  5, // 点1: 大小为5
  10, // 点2: 大小为10
  15, // 点3: 大小为15
  20, // 点4: 大小为20
]);
graph.setPointSizes(pointSizes);

// 自定义连接宽度 [width1, width2, ...]
const linkWidths = new Float32Array([0.5, 1, 1.5, 2]);
graph.setLinkWidths(linkWidths);

// 自定义连接箭头 [arrow1, arrow2, ...]
const linkArrows = [true, false, true, false];
graph.setLinkArrows(linkArrows);
```

#### 点集群

点集群功能允许将节点分组，并设置集群位置：

```typescript
// 设置点集群 [cluster1, cluster2, ...]
// undefined 表示节点不属于任何集群
const pointClusters = [0, 0, 1, 1];
graph.setPointClusters(pointClusters);

// 设置集群位置 [x1, y1, x2, y2, ...]
// undefined 表示使用自动居中位置
const clusterPositions = [100, 100, 300, 300];
graph.setClusterPositions(clusterPositions);

// 设置集群强度 [strength1, strength2, ...]
const clusterStrength = new Float32Array([0.5, 0.5, 0.5, 0.5]);
graph.setPointClusterStrength(clusterStrength);
```

### 第七步：性能优化

Cosmos 已经通过 GPU 加速和算法优化实现了高性能，但在使用时仍需注意以下几点：

1. **适当设置 spaceSize**：`spaceSize` 参数影响图的大小和力的计算范围
2. **调整力参数**：`simulationRepulsion`, `simulationLinkSpring`, `simulationGravity` 等参数会影响模拟效果和性能
3. **使用 quadtree 算法**：`useClassicQuadtree` 配置可控制是否使用四叉树优化大规模图的力计算
4. **控制节点和连接数量**：虽然 Cosmos 能处理大量节点，但仍应避免不必要的元素

### 结论

Cosmos 是一个强大的基于 WebGL 的图可视化库，通过将计算和渲染任务交给 GPU，它能够处理大规模的图数据可视化。本指南探讨了 Cosmos 的基本架构、数据表示方式、着色器实现和使用方法，希望能帮助前端工程师理解和使用这个库。

即使没有 WebGL 和图可视化的背景，也可以利用 Cosmos 创建高性能的图可视化应用。随着对库的深入了解，您还可以探索更多高级功能和自定义选项。
