# MPChart 使用指南

> 由 `harmonyos-app-dev` 在使用 `@ohos/mpchart` 构建图表时读取。官方包名: `@ohos/mpchart`;包中心可从 `https://ohpm.openharmony.cn/#/cn/home` 搜索 `mpchart` 或 `@ohos/mpchart`。

## 适用场景

`mpchart` 是 ArkTS/ArkUI 图表库,适合业务趋势图、柱状图、饼图、雷达图、仪表盘等。需要折线趋势、可拖动区间、坐标轴、图例、Marker 或多数据集时,优先考虑它,不要手写 Canvas 复刻完整图表能力。

## 安装

在使用图表的 module 下安装:

```bash
cd <module-dir>
ohpm install @ohos/mpchart
```

提交对应 module 的 `oh-package.json5` 和 `oh-package-lock.json5`,不要提交 `oh_modules`。

## LineChart 基础范式

核心对象:

- `LineChartModel`:图表模型和交互/坐标轴配置。
- `EntryOhos`:单个点,通常是 `(x, y)`。
- `JArrayList<EntryOhos>`:mpchart 使用的列表容器。
- `LineDataSet`:一组折线数据和样式。
- `LineData`:图表数据包,可包含多个 `ILineDataSet`。
- `LineChart({ model })`:ArkUI 组件渲染入口。

最小结构:

```ets
import {
  ChartColor,
  EntryOhos,
  ILineDataSet,
  JArrayList,
  LineChart,
  LineChartModel,
  LineData,
  LineDataSet,
  Mode,
  XAxis,
  XAxisPosition,
  YAxis,
  YAxisLabelPosition
} from '@ohos/mpchart';

@Component
struct TrendCard {
  private model: LineChartModel = new LineChartModel();

  aboutToAppear(): void {
    this.refreshChart();
  }

  private buildData(): LineData {
    const values: number[] = [72, 70, 74, 82, 78, 76, 68];
    let entries: JArrayList<EntryOhos> = new JArrayList<EntryOhos>();
    for (let i = 0; i < values.length; i++) {
      entries.add(new EntryOhos(i + 1, values[i]));
    }

    let dataSet: LineDataSet = new LineDataSet(entries, '心率');
    dataSet.setMode(Mode.CUBIC_BEZIER);
    dataSet.setCubicIntensity(0.18);
    dataSet.setLineWidth(3);
    dataSet.setColorByColor(ChartColor.rgb(255, 125, 73));
    dataSet.setCircleColor(ChartColor.rgb(255, 125, 73));
    dataSet.setCircleHoleColor(Color.White);
    dataSet.setCircleRadius(4);
    dataSet.setCircleHoleRadius(2);
    dataSet.setDrawValues(false);
    dataSet.setDrawFilled(true);
    dataSet.setFillColor(ChartColor.rgb(255, 232, 222));
    dataSet.setFillAlpha(70);

    let sets: JArrayList<ILineDataSet> = new JArrayList<ILineDataSet>();
    sets.add(dataSet);
    let data: LineData = new LineData(sets);
    data.setDrawValues(false);
    return data;
  }

  private refreshChart(): void {
    const leftAxis: YAxis | null = this.model.getAxisLeft();
    if (leftAxis) {
      leftAxis.setAxisMinimum(50);
      leftAxis.setAxisMaximum(110);
      leftAxis.setLabelCount(4, false);
      leftAxis.setPosition(YAxisLabelPosition.OUTSIDE_CHART);
      leftAxis.setTextSize(10);
    }

    const rightAxis: YAxis | null = this.model.getAxisRight();
    if (rightAxis) {
      rightAxis.setEnabled(false);
    }

    const xAxis: XAxis | null = this.model.getXAxis();
    if (xAxis) {
      xAxis.setPosition(XAxisPosition.BOTTOM);
      xAxis.setDrawGridLines(false);
      xAxis.setGranularity(1);
    }

    this.model.setTouchEnabled(true);
    this.model.setDragEnabled(true);
    this.model.setScaleEnabled(false);
    this.model.setPinchZoom(false);
    this.model.setData(this.buildData());
    this.model.notifyDataSetChanged();
    this.model.invalidate();
  }

  build() {
    LineChart({ model: this.model })
      .width('100%')
      .height(220)
  }
}
```

## 坐标轴文本与深色模式

`setTextColor` 的签名是:

```ets
setTextColor(color: string | number | CanvasGradient | CanvasPattern): void
```

它不是 ArkUI `fontColor`,不要直接传 `$r('app.color.xxx')` 这类 `ResourceColor`。如果图表需要跟随系统深浅色:

1. 在 Ability 中把运行时颜色模式写入 `AppStorage`,例如 `currentColorMode`。
2. 图表组件用 `@StorageProp('currentColorMode') @Watch(...)` 订阅。
3. 在 watch 回调里重新设置 X/Y 轴文本、网格线、轴线颜色,然后 `notifyDataSetChanged()` + `invalidate()`。

推荐颜色与应用文本层级保持一致:

```ets
private axisTextColor(): string {
  return this.currentColorMode === ConfigurationConstant.ColorMode.COLOR_MODE_DARK ? '#B3FFFFFF' : '#99000000';
}
```

网格线和轴线用 `ChartColor.argb(alpha, r, g, b)` 返回 `number`,不要在深色模式继续用固定浅灰。

## 数据刷新

- 数据点变化:重新构造 `LineData` 或更新 `LineDataSet`,然后 `model.setData(...)`。
- 坐标轴范围、formatter、图例等配置变化:更新对应对象后调用 `model.notifyDataSetChanged()`。
- 需要立即重绘:调用 `model.invalidate()`。
- 长列表趋势图:用 `setVisibleXRangeMaximum(...)` 限制可视点数,需要拖动时启用 `setDragEnabled(true)` / `setDragXEnabled(true)`。

## 常见配置

- 隐藏描述:取 `const description = model.getDescription()` 后判空,再 `description.setEnabled(false)`。
- 隐藏图例:取 `const legend = model.getLegend()` 后判空,再 `legend.setEnabled(false)`。
- 隐藏右轴:取 `const rightAxis = model.getAxisRight()` 后判空,再 `rightAxis.setEnabled(false)`。
- X 轴在底部: `xAxis.setPosition(XAxisPosition.BOTTOM)`。
- 关闭缩放: `setScaleEnabled(false)` + `setPinchZoom(false)`。
- 曲线: `dataSet.setMode(Mode.CUBIC_BEZIER)` 并控制 `setCubicIntensity(...)`,医疗/健康趋势不要过度平滑到误导读数。

## 常见坑

- `LineChartModel` 是普通成员即可;改了普通成员本身不会触发 ArkUI 状态刷新,但 MPChart 依赖 `model.invalidate()` 重绘。
- 不要在 `build()` 内频繁创建 model 或 data,会导致重绘和交互状态不稳定。
- mpchart 的颜色 API 分两类:文本可用字符串;网格线/轴线多数用 `number` 颜色。
- `ChartColor.rgb(...)` 不带 alpha;需要透明度时用 `ChartColor.argb(...)`。
- 构建可能出现 mpchart 包内部 ArkTS warning;只要业务代码无 error 且 `BUILD SUCCESSFUL`,先记录为三方包内部警告。
