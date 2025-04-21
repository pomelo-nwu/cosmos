# WebGL 基础知识

## 什么是 WebGL？

WebGL（Web Graphics Library）是一种 JavaScript API，允许在 HTML `<canvas>` 元素中渲染 2D 和 3D 图形，无需安装任何插件。它是基于 OpenGL ES 2.0 标准开发的，提供了在浏览器中进行高性能图形渲染的能力。

WebGL 最大的特点是可以直接访问 GPU（图形处理单元），这使得它能够处理大规模并行计算任务，非常适合图形渲染和数据可视化。

## 为什么图可视化库使用 WebGL？

### 传统图可视化方法的局限

传统的 Web 图可视化通常使用以下技术：

- **SVG**：基于 XML 的矢量图形格式
- **Canvas 2D**：提供 2D 上下文的绘图 API
- **DOM 元素**：直接操作 HTML 元素

这些方法在处理小规模数据时表现良好，但面对大规模数据（如包含数千或数万个节点的图）时，性能会急剧下降。原因在于：

1. 每个元素都需要在 DOM 或对象模型中单独表示
2. 更新需要频繁重绘大量元素
3. 计算都在 CPU 上进行，而 CPU 不擅长并行处理大量数据

### WebGL 的优势

WebGL 克服了这些限制：

1. **GPU 加速**：运算在 GPU 上进行，GPU 专为并行处理设计
2. **批量处理**：可以一次处理成千上万个顶点
3. **无 DOM 依赖**：不需要为每个元素创建 DOM 节点
4. **直接内存操作**：使用类型化数组（如 `Float32Array`）进行高效的内存操作

在 Cosmos 这样的图可视化库中，WebGL 允许我们同时渲染和模拟数十万个节点和边，实现实时交互，这是传统方法无法实现的。

## WebGL 核心概念

### 1. 上下文和初始化

在 Cosmos 中，通过 `regl` 库（一个 WebGL 封装库）初始化 WebGL 上下文：

```javascript
// 从 src/index.ts
this.reglInstance = regl({
  canvas: this.canvas,
  attributes: {
    antialias: false,
    preserveDrawingBuffer: true,
  },
  extensions: ["OES_texture_float", "ANGLE_instanced_arrays"],
});
```

这段代码：

- 创建 WebGL 上下文
- 禁用抗锯齿（提高性能）
- 启用绘图缓冲区保留（用于屏幕截图等功能）
- 启用两个扩展：`OES_texture_float`（允许使用浮点纹理）和 `ANGLE_instanced_arrays`（用于实例化渲染）

### 2. 着色器（Shader）

着色器是运行在 GPU 上的小程序，分为两种类型：

#### 顶点着色器（Vertex Shader）

顶点着色器处理顶点位置和属性。在 Cosmos 中，顶点着色器负责确定节点的位置和大小：

```glsl
// 简化自 src/modules/Points/draw-points.vert
void main() {
  // 从纹理获取点位置
  vec4 pointPosition = texture2D(positionsTexture, (textureCoords + 0.5) / pointsTextureSize);
  vec2 point = pointPosition.rg;

  // 转换到标准化设备坐标
  vec2 normalizedPosition = 2.0 * point / spaceSize - 1.0;
  // 应用变换矩阵（用于缩放、平移）
  vec3 finalPosition = transformationMatrix * vec3(normalizedPosition, 1);
  // 设置顶点位置
  gl_Position = vec4(finalPosition.rg, 0, 1);

  // 设置点大小（会影响片段着色器中的范围）
  gl_PointSize = calculatePointSize(size * sizeScale);
}
```

#### 片段着色器（Fragment Shader）

片段着色器处理像素颜色和透明度。在 Cosmos 中，它用于绘制平滑的圆形节点：

```glsl
// 简化自 src/modules/Points/draw-points.frag
void main() {
    // 丢弃完全透明的像素
    if (alpha == 0.0) { discard; }

    // 计算到点中心的距离
    vec2 pointCoord = 2.0 * gl_PointCoord - 1.0;
    float pointCenterDistance = dot(pointCoord, pointCoord);

    // 基于距离计算透明度，创建平滑边缘
    float opacity = alpha * (1.0 - smoothstep(smoothing, 1.0, pointCenterDistance));

    // 设置最终颜色（RGB + 透明度）
    gl_FragColor = vec4(rgbColor, opacity);
}
```

### 3. 缓冲区（Buffer）

缓冲区是在 GPU 内存中存储数据的对象。在 Cosmos 中，用于存储点的位置、颜色、大小等：

```javascript
// 从 src/modules/Points/index.ts
// 存储点的颜色
this.colorBuffer = reglInstance.buffer(colors);

// 存储点的大小
this.sizeBuffer = reglInstance.buffer(sizes);
```

### 4. 纹理（Texture）

纹理通常用于存储图像，但在 WebGL 计算中，它们也可以用作通用数据存储。Cosmos 使用纹理存储节点位置、速度和其他属性：

```javascript
// 从 src/modules/Points/index.ts
this.currentPositionFbo({
  color: reglInstance.texture({
    data: initialState,
    shape: [pointsTextureSize, pointsTextureSize, 4],
    type: "float",
  }),
  depth: false,
  stencil: false,
});
```

这里创建了一个浮点纹理，存储节点的位置信息。每个节点在纹理中占用 4 个浮点数（RGBA 通道）。

### 5. 帧缓冲对象（Framebuffer）

帧缓冲对象允许渲染到纹理而不是屏幕。在 Cosmos 中，这用于实现力模拟计算：

```javascript
// 从 src/modules/ForceManyBody/index.ts
// 创建用于力计算的帧缓冲区
this.forceCommand = reglInstance({
  frag: forceFrag,
  vert: updateVert,
  framebuffer: () => points?.velocityFbo as regl.Framebuffer2D,
  // ...
});
```

这使得我们可以将力的计算结果直接写入纹理，然后在下一步中使用这些结果更新节点位置。

### 6. 绘制命令

WebGL 绘制命令定义了如何渲染对象。在 Cosmos 中，这些命令被封装在 `regl` 命令中：

```javascript
// 从 src/modules/Points/index.ts
this.drawCommand = reglInstance({
  frag: drawPointsFrag,
  vert: drawPointsVert,
  primitive: "points",
  count: () => data.pointsNumber ?? 0,
  attributes: {
    // 定义顶点属性
    pointIndices: {
      buffer: this.drawPointIndices,
      size: 2,
    },
    size: {
      buffer: () => this.sizeBuffer,
      size: 1,
    },
    color: {
      buffer: () => this.colorBuffer,
      size: 4,
    },
  },
  uniforms: {
    // 定义着色器统一变量
    positionsTexture: () => this.currentPositionFbo,
    // ...其他参数
  },
  // ...混合模式设置
});
```

### 7. 渲染循环

WebGL 应用通常使用 `requestAnimationFrame` 建立渲染循环。在 Cosmos 中，渲染循环负责运行力模拟并更新画面：

```javascript
// 简化自 src/index.ts 中的 frame 方法
private frame(): void {
  // 运行力模拟
  if (this.store.isSimulationRunning) {
    if (this.forceManyBody) this.forceManyBody.run()
    if (this.forceLinkIncoming) this.forceLinkIncoming.run(LinkDirection.INCOMING)
    if (this.forceLinkOutgoing) this.forceLinkOutgoing.run(LinkDirection.OUTGOING)
    // ...其他力的计算

    // 更新位置
    this.points.updatePosition()
  }

  // 清除画布
  this.reglInstance.clear({
    color: this.store.backgroundColor,
    depth: 1,
  })

  // 绘制线和点
  this.lines.draw()
  this.points.draw()

  // 请求下一帧
  this.requestAnimationFrameId = requestAnimationFrame(this.frame.bind(this))
}
```

## WebGL 在图可视化中的应用

Cosmos 利用 WebGL 实现了几个核心功能：

### 1. 高效渲染大量节点和边

通过将节点表示为 WebGL 点精灵（point sprites），边表示为线段或曲线，Cosmos 可以高效渲染数十万个图元素：

```javascript
// 来自 src/modules/Points/index.ts 和 src/modules/Lines/index.ts
// 渲染点
this.drawCommand();

// 渲染线
this.drawLinesCommand();
```

### 2. GPU 加速的力模拟

Cosmos 最创新的部分是在 GPU 上完成力导向布局计算：

```javascript
// 从 src/modules/ForceManyBody/index.ts
public run(): void {
  // 清除力计算层
  for (let i = 0; i < this.quadtreeLevels; i += 1) {
    this.clearLevelsCommand?.({ levelFbo: this.levelsFbos.get(`level[${i}]`) });
    // ...
  }

  // 在 GPU 上计算排斥力
  this.clearVelocityCommand?.();
  for (let i = 0; i < this.quadtreeLevels; i += 1) {
    this.forceCommand?.({
      levelFbo: this.levelsFbos.get(`level[${i}]`),
      levelTextureSize,
      level: i,
    });
    // ...
  }
}
```

### 3. 快速选择和交互

WebGL 允许高效地实现节点选择和交互功能：

```javascript
// 从 src/modules/Points/index.ts
// 实现选择区域内的点
this.findPointsOnAreaSelectionCommand?.();

// 实现悬停在点上的交互
this.findHoveredPointCommand?.();
```

## 总结

WebGL 为图可视化带来了革命性的性能提升，使得处理大规模数据成为可能。Cosmos 库通过精心设计的着色器和数据结构，充分利用 GPU 的并行计算能力，实现了高性能的图渲染和力模拟。

在后续章节中，我们会深入探讨 Cosmos 如何具体实现力导向布局算法、节点渲染和用户交互等功能，将这些 WebGL 概念应用到实际的图可视化中。
