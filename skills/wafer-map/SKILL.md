---
name: wafer-map
description: Guide for using @guwave/wafermap-v2 to render semiconductor wafer maps with Canvas. Use when creating, configuring, or integrating wafer map visualizations (die map, bin map, reticle map, site map) in React or Vue3 projects. **Crucial: Verify .npmrc configuration for @guwave scope before installation.** Triggers on: "wafer map", "晶圆图", "die map", "bin map", "WaferMap", "@guwave/wafermap", "wafermap", ".npmrc config for wafermap".
---

# @guwave/wafermap-v2 使用指南

基于 Canvas 的高性能半导体晶圆图渲染组件，**框架无关**（React / Vue3 / 原生 JS 均可）。

## 前提条件：.npmrc 配置

由于 `@guwave/wafermap-v2` 托管在私有仓库，在安装前**必须**确保项目根目录下存在 `.npmrc` 文件，并且正确配置了 `@guwave` 的作用域仓库地址。

必须检查或创建 `.npmrc` 文件，内容参考如下：

```ini
registry=https://registry.npmmirror.com/

# 必须配置 @guwave 作用域以指向私有仓库
@guwave:registry=http://192.168.2.210:7001/
```

## 安装

确认 `.npmrc` 配置正确后，执行以下命令：

```bash
pnpm add @guwave/wafermap-v2 lodash
```

`lodash` 为 peerDependency，必须安装。

如果项目执行 `pnpm install` 报错，先删除项目根目录下的 `node_modules`，再重试：

```bash
rm -rf node_modules
pnpm install
```

(如果项目中有 `package-lock.json`, 即用的是 npm, 则把 pnpm 命令换为 npm)

## 导入

```typescript
import WaferMap from '@guwave/wafermap-v2';
import '@guwave/wafermap-v2/style.css';

// 类型（按需）
import type { WaferMapOptions, WaferMapDataItem } from '@guwave/wafermap-v2';

// 颜色工具（可选，单独入口）
import { getColorByPFBinNum, makeColorBlockColors, BinPFTypeEnum } from '@guwave/wafermap-v2/utils';
```

## 核心概念

| 概念 | 说明 |
| ------ | ------ |
| **Wafer** | 圆形晶圆，`wafer.diameter` 设置直径（mm），`wafer.notch` 设置缺口方向 |
| **Die** | 晶圆上最小芯片单元，`die.width/height` 设置物理尺寸（mm） |
| **Reticle** | 光罩区域，包含多个 Die，由 `center.dieCountInReticle` 控制 |
| **坐标系** | `axis.directionX`（`'left'`/`'right'`）+ `axis.directionY`（`'up'`/`'down'`）控制方向 |

### 两种渲染模式

- **`type: 'dashboard'`**（默认）— 数据驱动：提供 `data` 数组，每个 die 的坐标和属性由数据决定
- **`type: 'waferMap-config'`** — 配置驱动：仅提供晶圆参数和可选 `siteData`，组件自动生成圆内 die 数据

## 快速开始

```typescript
const container = document.getElementById('wafer-container')!;
// 容器必须有明确的 CSS 宽高
const waferMap = new WaferMap(container, { zoomEnabled: true });

waferMap.setOptions({
  type: 'dashboard',
  clip: true,
  wafer: { diameter: 300, margin: 3, notch: { direction: 'down' } },
  axis: { show: true, directionX: 'right', directionY: 'up' },
  center: { dieCountInReticle: { x: 1, y: 1 } },
  die: {
    width: 3.5,
    height: 4.2,
    bgColor: (item) => colorMap[item.k] || '#ccc',
    line: { show: true, width: 1, color: 'white' },
  },
  dataMap: { dieX: 'dieX', dieY: 'dieY', reticleX: 'rx', reticleY: 'ry' },
  data: dieDataArray,
});
```

## WaferMapOptions 配置速查

| 属性 | 类型 | 默认值 | 说明 |
| ------ | ------ | -------- | ------ |
| `type` | `'dashboard' \| 'waferMap-config'` | `'dashboard'` | 渲染模式 |
| `padding` | `number` | `10` | 画布内边距（px） |
| `clip` | `boolean` | `true` | 是否裁剪圆外 die |
| `rotate` | `number` | `0` | 旋转角度（度） |
| `isSaveDiePos` | `boolean` | `false` | 保存 die 像素位置到数据项 |
| `resetZoom` | `boolean` | `false` | setOptions 时重置缩放 |
| `data` | `WaferMapDataItem[]` | — | die 数据（dashboard 模式必需） |
| `siteData` | `WaferMapSiteData[]` | — | site 数据（waferMap-config 模式可选） |
| `dataMap` | `Record<string, string>` | 见下方 | 数据字段名映射 |
| `dieRange` | `{ minX, minY, maxX, maxY }` | — | 限定渲染的 die 坐标范围 |
| `isDrawDataItem` | `(item) => boolean` | — | 自定义过滤条件 |
| `tooltip` | `boolean \| { render, debounce }` | `true` | tooltip 配置 |

### wafer 晶圆配置

```typescript
wafer: {
  show: true,
  diameter: 300,        // 晶圆直径（mm）
  margin: 3,            // 边距（mm）
  notch: {
    show: true,
    direction: 'down',  // 'up' | 'down' | 'left' | 'right'
    size: 10,
    bgColor: '#000',
  },
  inner: { show: true, line: { color: '#858585', width: 1 }, bgColor: '#ececec' },
  outer: { show: true, line: { color: '#858585', width: 1 }, bgColor: '#ececec' },
}
```

### axis 坐标轴

```typescript
axis: {
  show: true,
  directionX: 'right', // 'left' | 'right'
  directionY: 'up',    // 'up' | 'down'
  line: { width: 1, color: '#707070' },
  font: { fontSize: 10, color: '#000' },
}
```

### center 坐标系中心

```typescript
center: {
  die: { x: 0, y: 0 },                     // 中心 die 坐标
  dieOffset: { xLength: 0, yLength: 0 },    // die 中心偏移（mm）
  reticle: { x: 0, y: 0 },                  // 中心 reticle 坐标
  reticleOffset: { xLength: 0, yLength: 0 }, // reticle 中心偏移（mm）
  dieInReticle: { x: 0, y: 0 },             // reticle 内 die 起始索引
  dieCountInReticle: { x: 1, y: 1 },        // 每 reticle 包含的 die 数（x × y）
}
```

### die 芯片配置

```typescript
die: {
  show: true,
  width: 4,             // mm
  height: 4,            // mm
  margin: 0,            // die 间距
  bgColor: '#189A1F',   // 字符串 或 (item: WaferMapDataItem) => string
  line: {
    show: true,         // boolean | 'auto'（按缩放比例自动显隐）
    width: 2,
    color: '#3a3a3a',
    showByRatio: 0.5,   // 'auto' 模式触发比例
  },
  label: { show: false, fontSize: 6, color: '#000' },
}
```

### reticle 光罩配置

```typescript
reticle: {
  show: false,
  line: { width: 2, color: '#3a3a3a' },
  bgColor: '#fff',      // 字符串 或 (item: WaferMapDataItem) => string
  label: { show: false, fontSize: 16, color: '#000' },
  centerReticleLine: { show: false, width: 1, color: '#aaa' },
}
```

### dataMap 字段映射

后端数据字段名与标准字段不同时配置：

```typescript
dataMap: {
  dieX: 'dieX',             // 数据中 die X 坐标字段名
  dieY: 'dieY',
  reticleX: 'reticleX',
  reticleY: 'reticleY',
  dieLabel: 'dieLabel',
  reticleLabel: 'reticleLabel',
}
```

### WaferMapDataItem 数据格式

```typescript
interface WaferMapDataItem {
  dieX?: number;
  dieY?: number;
  reticleX?: number;
  reticleY?: number;
  dieLabel?: string | number;
  reticleLabel?: string | number;
  [key: string]: any;        // 携带任意业务数据（binType, color 等）
}
```

## 实例方法

```typescript
waferMap.setOptions(options: WaferMapOptions)       // 设置配置并渲染
waferMap.resize(options?: WaferMapOptions)           // 响应容器尺寸变化
waferMap.resetZoom()                                 // 重置缩放到初始状态
waferMap.enableZoom(enabled: boolean)                // 启用/禁用滚轮缩放
waferMap.setRotate(degree: number)                   // 设置旋转角度
waferMap.getData(isCurrent?: boolean)                // 获取 die/reticle shape 数据
waferMap.highlight({ highlightLayer, brush })        // 更新高亮配置
waferMap.changeBrushEnabled(enabled?: boolean)       // 启用/禁用框选
waferMap.clearBrushedShapes()                        // 清除框选
waferMap.onBrushChange(fn: (shapes?) => void)        // 监听框选，返回 unsubscribe
waferMap.onClickDie(fn, { native?: boolean })        // 监听 die 点击，返回 unsubscribe
waferMap.toDataURL({ type?, quality? })              // 导出 DataURL
waferMap.toBlob({ type?, quality? })                 // 导出 Blob（Promise）
waferMap.getFinalCanvas()                            // 获取合成 Canvas 元素
waferMap.use(plugin: Plugin, options?)               // 注册插件（链式调用）
waferMap.destroy()                                   // 销毁实例，释放资源
```

## 框架集成

### React

```tsx
import { useRef, useEffect } from 'react';
import WaferMap from '@guwave/wafermap-v2';
import '@guwave/wafermap-v2/style.css';
import type { WaferMapOptions } from '@guwave/wafermap-v2';

function WaferMapView({ options }: { options: WaferMapOptions }) {
  const containerRef = useRef<HTMLDivElement>(null);
  const instanceRef = useRef<WaferMap | null>(null);

  useEffect(() => {
    if (!containerRef.current) return;
    instanceRef.current = new WaferMap(containerRef.current, { zoomEnabled: true });
    return () => { instanceRef.current?.destroy(); instanceRef.current = null; };
  }, []);

  useEffect(() => {
    if (options) instanceRef.current?.setOptions(options);
  }, [options]);

  return <div ref={containerRef} style={{ width: '100%', height: 600 }} />;
}
```

### Vue 3

```vue
<template>
  <div ref="containerRef" style="width: 100%; height: 600px" />
</template>
<script setup lang="ts">
import { ref, onMounted, onUnmounted, watch } from 'vue';
import WaferMap from '@guwave/wafermap-v2';
import '@guwave/wafermap-v2/style.css';
import type { WaferMapOptions } from '@guwave/wafermap-v2';

const props = defineProps<{ options: WaferMapOptions }>();
const containerRef = ref<HTMLElement>();
let instance: WaferMap | null = null;

onMounted(() => {
  instance = new WaferMap(containerRef.value!, { zoomEnabled: true });
  instance.setOptions(props.options);
});

watch(() => props.options, (opts) => { if (opts) instance?.setOptions(opts); }, { deep: true });
onUnmounted(() => { instance?.destroy(); instance = null; });
</script>
```

## 场景示例

### 1. BinMap — 按 Bin 着色

```typescript
const colorMap: Record<string, string> = {
  'P-HBIN1-HBin1': '#34AA0C',
  'F-HBIN2-HBin2': '#ff600d',
  'F-HBIN4-HBin4': '#d254a5',
};

waferMap.setOptions({
  type: 'dashboard',
  clip: true,
  wafer: { diameter: 300, margin: 3, notch: { direction: 'down' } },
  axis: { show: true, directionX: 'right', directionY: 'up' },
  center: { dieCountInReticle: { x: 1, y: 1 } },
  die: {
    width: 0.82,
    height: 0.91,
    bgColor: (item) => colorMap[item.k] || '#eee',
    line: { show: true, width: 1, color: 'white' },
  },
  grid: { show: true },
  dataMap: { dieX: 'dieX', dieY: 'dieY', reticleX: 'rx', reticleY: 'ry' },
  tooltip: true,
  data: binData, // [{ dieX, dieY, rx, ry, k: 'P-HBIN1-HBin1', ... }, ...]
});
```

### 2. 含 Reticle 的晶圆图

```typescript
waferMap.setOptions({
  clip: false,
  wafer: { diameter: 26, margin: 2, notch: { direction: 'down' } },
  axis: { show: true, directionX: 'right', directionY: 'up' },
  center: {
    dieOffset: { xLength: 2, yLength: 3 },
    dieInReticle: { x: 1, y: 2 },
    dieCountInReticle: { x: 2, y: 3 },
  },
  die: { width: 4, height: 4, line: { show: true, width: 1 } },
  reticle: { show: true, line: { width: 4 }, label: { show: true } },
  dataMap: { reticleLabel: 'site' },
  data: [
    { reticleX: 0, reticleY: 0, dieX: 0, dieY: 0 },
    { reticleX: 0, reticleY: 0, dieX: 0, dieY: 1 },
    { reticleX: 0, reticleY: 0, dieX: 1, dieY: 0, site: 'A1' },
    { reticleX: 1, reticleY: 0, dieX: 2, dieY: 0 },
  ],
});
```

### 3. waferMap-config 模式（自动生成）

```typescript
waferMap.setOptions({
  type: 'waferMap-config',
  wafer: { diameter: 100, margin: 2, notch: { direction: 'down' } },
  axis: { show: true, directionX: 'right', directionY: 'up' },
  center: {
    dieCountInReticle: { x: 2, y: 3 },
  },
  die: { width: 4, height: 4 },
  siteData: [
    { reticleX: 1, reticleY: 1, reticleLabel: 'S1' },
    { reticleX: 2, reticleY: 1, reticleLabel: 'S2' },
  ],
});
```

### 4. 高亮与框选

```typescript
waferMap.setOptions({
  // ...基础 wafer/die/center 配置
  brush: {
    needBrushHighlight: true,
    supportAltSelected: true,
    styleOptions: {
      area: {
        line: { width: 1, color: 'rgba(93,45,205,1)' },
        bgColor: 'rgba(93,45,205,0.25)',
      },
    },
  },
  highlightLayer: {
    show: true,
    maskBgColor: 'rgba(255,255,255,0.725)',
    isHighlight: ({ shape }) => {
      if (shape.data.binType === 'FAIL') {
        return { die: { bgColor: 'red' } };
      }
      return undefined; // 不高亮
    },
  },
  data: dieData,
});

const unsubBrush = waferMap.onBrushChange((shapes) => {
  console.log('选中的 die:', shapes);
});
const unsubClick = waferMap.onClickDie((item) => {
  console.log('点击:', item);
  return true; // 返回 true 阻止默认行为
});

// 清理
unsubBrush();
unsubClick();
```

### 5. 导出图片

```typescript
const dataUrl = waferMap.toDataURL({ type: 'image/png' });

const blob = await waferMap.toBlob({ type: 'image/jpeg', quality: 0.9 });
if (blob) {
  const url = URL.createObjectURL(blob);
  // 下载或展示
}
```

## SublinePlugin 辅助线

```typescript
import { SublinePlugin } from '@guwave/wafermap-v2';

const waferMap = new WaferMap(container);
waferMap.use(new SublinePlugin());
waferMap.setOptions(options);

waferMap.subline.setSubline({
  row: { enable: true, count: 8 },
  col: { enable: true, count: 8 },
  circle: { enable: true, count: 3, splitType: 'RADIUS' }, // 'RADIUS' | 'AREA'
  sector: { enable: true, count: 6 },
  style: { color: '#999', lineWidth: 1, lineType: 'dashed' },
});
```

- 调用路径是 `waferMap.subline.setSubline(...)`，不是 `waferMap.setSubline(...)`
- 使用 subline 时，建议在 `setOptions` 中开启 `isSaveDiePos: true`（插件依赖该信息）
- **row / col**：水平行线 / 垂直列线，等分晶圆区域
- **circle**：同心圆，`RADIUS` 等分半径 / `AREA` 等分面积
- **sector**：扇形分区线
- **Ctrl/Alt + 双击**：选中区域内所有 die

## 颜色工具

```typescript
import {
  getColorByPFBinNum,
  getColorByPFBinNumV2,
  BinPFTypeEnum,
  makeColorBlockColors,
  BIN_PF_MAP_COLORS,
} from '@guwave/wafermap-v2/utils';

// 按 P/F 和 BinNumber 获取颜色
getColorByPFBinNum(BinPFTypeEnum.PASS, 1);  // '#34AA0C'
getColorByPFBinNum(BinPFTypeEnum.FAIL, 3);  // '#feed11'

// 生成连续色阶（可选 schema）
const { colorGroupArrayHex } = makeColorBlockColors(10, 'red_green');
// colorSchema: 'red_green' | 'green_red' | 'lightGreen_paleGreen' |
//   'paleGreen_lightGreen' | 'lightRed_paleRed' | 'paleRed_lightRed' |
//   'red_green_blue' | 'blue_green_yellow'
```

## 注意事项

- 容器元素**必须有明确的 CSS 宽高**，WaferMap 自动适配容器大小
- `data` 数组量大时不要频繁深拷贝，组件已优化大数据处理
- `die.line.show: 'auto'` 在缩放较小时自动隐藏边框线以提升性能
- 销毁前调用 `onClickDie`/`onBrushChange` 返回的取消函数移除监听
- 调用 `destroy()` 释放 Canvas 和事件资源

## 详细类型参考

完整 TypeScript 类型定义见 [reference.md](reference.md)
