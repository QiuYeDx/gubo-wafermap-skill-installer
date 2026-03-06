# @guwave/wafermap-v2 完整类型参考

## 构造函数

```typescript
class WaferMap extends WaferMapEvent {
  constructor(dom: HTMLElement | number, config?: Config);
}

interface Config {
  dpr?: number;
  zoomEnabled?: boolean; // 默认 true
}
```

`dom` 传入 `HTMLElement` 渲染到该容器；传入 `number` 用于 OffscreenCanvas 场景（此时 zoomEnabled 强制 false）。

## WaferMapOptions

```typescript
interface WaferMapOptions {
  type?: 'dashboard' | 'waferMap-config';
  padding?: number;
  clip?: boolean;
  rotate?: number;
  isSaveDiePos?: boolean;
  resetZoom?: boolean;

  axis?: {
    show?: boolean;
    directionX?: 'left' | 'right';
    directionY?: 'up' | 'down';
    line?: ILine;
    font?: IFont;
  };

  grid?: {
    show?: boolean;
    line?: ILine;
  };

  wafer?: {
    show?: boolean;
    notch?: IWaferNotch;
    diameter?: number;
    margin?: number;
    inner?: IWaferCircle;
    outer?: IWaferCircle;
  };

  center?: {
    die?: ICoord;
    dieOffset?: IOffset;
    reticle?: ICoord;
    reticleOffset?: IOffset;
    dieInReticle?: ICoord;
    dieCountInReticle?: ICoord;
  };

  die?: {
    show?: boolean;
    width?: number;
    height?: number;
    margin?: number;
    bgColor?: string | ((item: WaferMapDataItem) => string);
    line?: ILine & { showByRatio?: number };
    label?: ILabel;
  };

  reticle?: {
    show?: boolean;
    line?: ILine;
    bgColor?: string | ((item: WaferMapDataItem) => string);
    label?: ILabel;
    centerReticleLine?: ILine;
  };

  dieRange?: DieRange;
  tooltip?: WaferMapTooltipOptions;
  dataMap?: Record<keyof WaferMapDataItem, string>;
  data?: WaferMapDataItem[];
  siteData?: WaferMapSiteData[];
  isDrawDataItem?: (item: WaferMapDataItem) => boolean;
  brush?: WaferMapBrushOptions;
  highlightLayer?: HighlightLayerOptions & { maskBgColor: string };
}
```

## 数据类型

```typescript
interface WaferMapDataItem {
  dieLabel?: string | number;
  reticleLabel?: string | number;
  dieX?: number;
  dieY?: number;
  reticleX?: number;
  reticleY?: number;
  diePos?: { x: number; y: number; dieWidth: number; dieHeight: number };
  [key: string]: any;
}

interface WaferMapSiteData {
  reticleX: number;
  reticleY: number;
  reticleLabel: string | number;
}
```

## 基础样式类型

```typescript
interface ICoord { x: number; y: number }
interface IOffset { xLength: number; yLength: number }
interface IFont { fontSize?: number; color?: string }
interface ILine { show?: boolean | 'auto'; width?: number; color?: string }
interface IWaferLine extends ILine { type?: 'solid' | 'dashed' | 'dotted' }
interface ILabel extends IFont { show?: boolean | 'auto' }

interface IWaferCircle {
  show?: boolean;
  line?: IWaferLine;
  bgColor?: string;
}

interface IWaferNotch {
  show?: boolean;
  direction?: 'up' | 'down' | 'left' | 'right';
  size?: number;
  bgColor?: string;
}

interface DieRange {
  minX: number;
  minY: number;
  maxX: number;
  maxY: number;
}
```

## Tooltip 类型

```typescript
type WaferMapTooltipOptions = boolean | WaferMapTooltipObjOptions;

interface WaferMapTooltipObjOptions {
  render: (params: WaferMapTooltipRenderParams) => string | HTMLElement;
  debounce?: number;
}

interface WaferMapTooltipRenderParams {
  data: WaferMapDataItem;
  reticleData: WaferMapDataItem;
  event: MouseEvent;
}
```

## Brush 框选类型

```typescript
interface WaferMapBrushOptions {
  alwaysSupportHighlight?: boolean;
  needBrushHighlight?: boolean;
  supportAltSelected?: boolean;
  multiBrush?: boolean;
  styleOptions?: {
    mark?: Partial<IStyle>;
    mask?: Partial<IStyle>;
    area?: Partial<IStyle>;
  };
}

interface IStyle {
  line?: { show?: boolean | 'auto'; width?: number; color?: string };
  bgColor?: string;
  borderColor?: string;
  borderWidth?: number;
}
```

## HighlightLayer 类型

```typescript
interface HighlightLayerOptions {
  show?: boolean;
  maskBgColor?: string;
  isHighlight?: (params: {
    shape: { data: WaferMapDataItem; type: 'die' | 'reticle' };
  }) => IsHighLightRes | undefined;
}

interface IsHighLightRes {
  die?: Partial<IStyle>;
  reticle?: Partial<IStyle>;
}
```

## Plugin 插件接口

```typescript
type Plugin = {
  name: string;
  init: (instance: any, ...options: any[]) => any;
  destroy: () => void;
  [key: string]: any;
};
```

## SublinePlugin 辅助线

```typescript
import { SublinePlugin } from '@guwave/wafermap-v2';

class SublinePlugin implements Plugin {
  name: 'subline';
  init(waferMap: WaferMap): void;
  setSubline(options: SetSublineOptions): void;
  destroy(): void;
}

// 使用后 waferMap 实例会挂载 subline 对象
interface WaferMap {
  subline: SublinePlugin;
}

type SetSublineOptions = {
  row?: OptionItem;
  col?: OptionItem;
  sector?: OptionItem;
  circle?: OptionItem;
  style?: LineStyle;
};

type OptionItem = {
  enable: boolean;
  count: number;
  splitType?: 'RADIUS' | 'AREA'; // 仅 circle 有效
  showLine?: boolean;
};

type LineStyle = {
  color?: string;
  lineWidth?: number;
  lineType?: 'solid' | 'dashed';
  lineDash?: number[];
  zIndex?: number;
  showLine?: boolean;
};
```

## DrawOption（getData 返回值）

```typescript
interface DrawOption {
  wafer: {
    innerRadius: number;
    outerRadius: number;
    notch: Required<IWaferNotch>;
    x: number;
    y: number;
  };
  die: {
    width: number;
    height: number;
    margin: number;
    dieOffset: IOffset;
    centerDiePos: ICoord;
    range: DrawItemRange;
  };
  reticle: {
    width: number;
    height: number;
    reticleOffset: IOffset;
    centerReticlePos: ICoord;
    range: DrawItemRange;
  };
}

type DrawItemRange = {
  xStart: number;
  xCount: number;
  yStart: number;
  yCount: number;
};
```

## 颜色工具（@guwave/wafermap-v2/utils）

```typescript
enum BinPFTypeEnum {
  PASS = 'P',
  FAIL = 'F',
  UNKNOWN = 'U',
}

// 预设颜色
const BIN_PF_MAP_COLORS: {
  PASS: string[];  // 8 种绿色系
  FAIL: string[];  // 50 种颜色
  ALG: string[];   // 14 种颜色
};

const PF_MAP_COLORS: { PASS: string; FAIL: string };

// 按 P/F 和 BinNum 取色
function getColorByPFBinNum(pf?: BinPFTypeEnum, binNum?: number): string;
function getColorByPFBinNumV2(
  pf?: BinPFTypeEnum,
  binNum?: number,
  index?: number,
  colorSchema?: string[],
): string;

type ColorSchema =
  | 'red_green'
  | 'green_red'
  | 'lightGreen_paleGreen'
  | 'paleGreen_lightGreen'
  | 'lightRed_paleRed'
  | 'paleRed_lightRed'
  | 'red_green_blue'
  | 'blue_green_yellow';

function makeColorBlockColors(
  colorGroupCnt: number,
  colorSchema: ColorSchema,
): {
  colorGroupArray: number[];
  colorGroupArrayHex: string[];
  colorGroupArrayHexMap: Record<string, string>;
};

function hslToHex(h: number, s: number, l: number): string;
```

## 已导出常量

```typescript
const STANDARD_OPTIONS: {
  notch: 'down';
  directionX: 'right';
  directionY: 'up';
  centerDie: { x: 0; y: 0 };
  centerReticle: { x: 0; y: 0 };
};

const DEFAULT_OPTIONS: {
  type: 'dashboard';
  padding: 10;
  clip: true;
  isSaveDiePos: false;
  axis: { show: false; directionX: 'right'; directionY: 'up'; /* ... */ };
  grid: { show: true; line: { width: 1; color: '#e3e3e3' } };
  center: {
    die: { x: 0; y: 0 };
    dieOffset: { xLength: 0; yLength: 0 };
    reticle: { x: 0; y: 0 };
    reticleOffset: { xLength: 0; yLength: 0 };
    dieInReticle: { x: 0; y: 0 };
    dieCountInReticle: { x: 1; y: 1 };
  };
  wafer: {
    show: true;
    notch: { show: true; direction: 'down'; size: 10; bgColor: '#000' };
    inner: { show: true; line: { color: '#858585'; width: 1 }; bgColor: '#ececec' };
    outer: { show: true; line: { color: '#858585'; width: 1 }; bgColor: '#ececec' };
  };
  die: {
    show: true;
    margin: 0;
    bgColor: '#189A1F';
    line: { show: true; width: 2; color: '#3a3a3a' };
    label: { fontSize: 6; show: false; color: '#000' };
  };
  reticle: {
    show: false;
    line: { width: 2; color: '#3a3a3a' };
    label: { show: false; fontSize: 16; color: '#000' };
  };
  tooltip: true;
  highlightLayer: { maskBgColor: 'rgba(255,255,255,0.725)' };
  resetZoom: false;
};
```

## 事件常量

```typescript
const ON_CLICK_DIE = 'ON_CLICK_DIE';
const ON_BRUSH_CHANGE = 'ON_BRUSH_CHANGE';
const ON_DRAG = 'ON_DRAG';
const ON_ZOOM = 'ON_ZOOM';
const ON_BEFORE_DRAW = 'ON_BEFORE_DRAW';
```

## 完整导出清单

```typescript
// 从 '@guwave/wafermap-v2'
export default WaferMap;
export { WaferMap };
export type WaferMapInstance = InstanceType<typeof WaferMap>;
export { STANDARD_OPTIONS, DEFAULT_OPTIONS };
export { SublinePlugin };

// 所有 typings 类型
export type {
  Config, WaferMapOptions, WaferMapDataItem, WaferMapSiteData,
  WaferMapTooltipOptions, WaferMapTooltipObjOptions, WaferMapTooltipRenderParams,
  WaferMapBrushOptions, HighlightLayerOptions, IsHighLightRes,
  ICoord, IOffset, IFont, ILine, IWaferLine, ILabel, IWaferCircle, IWaferNotch, IStyle,
  DieRange, DrawOption, DrawItemRange, DrawCtx, CanvasShape, DieShape,
  HighlightItem, EventBus, CanvasItem, Plugin,
  NotchDirection, XAxisDirection, YAxisDirection,
};

// 所有 brush 类型
export type { BrushOptions, Position, ContainerElement };

// SublinePlugin 相关类型
export type { SetSublineOptions, WaferRect, OptionItem, LineStyle };

// 从 '@guwave/wafermap-v2/utils'
export {
  getColorByPFBinNum, getColorByPFBinNumV2,
  BinPFTypeEnum, BIN_PF_MAP_COLORS, PF_MAP_COLORS,
  hslToHex, makeColorBlockColors,
};
export type { ColorSchema };
```
