# RememberRemember

*The gunpowder, treason and... PLOT.*

RememberRemember is a small AppKit charting framework for macOS, built for the result plots in
kiems — S-parameters, impedance, delay, Smith charts and eye diagrams from
Copper's FDTD simulations. It replaced DGCharts, keeping the same look (`ChartPalette` reproduces
DGCharts' default colours) without the much larger dependency.

## Views

| View | Use |
| --- | --- |
| `LineChartView` | The core chart: any number of curves on shared X values, left/right Y axes, linear or logarithmic X, optional min/max band and X intensity band. |
| `MultiCurveLineChartView` | Single-Y-axis wrapper with a Y-axis caption. Plain, styled, and banded (average + probes + min/max) variants. |
| `DualAxisLineChartView` | Magnitude on the left axis (solid), phase/angle on the right (dashed). |
| `SmithChartView` | Normalised-impedance Smith chart with a reflection-coefficient trace and a VSWR margin circle. |
| `EyeDiagramView` | Two-UI eye diagram drawn as an accumulated trace-density image, colourised on the GPU (`EyeDensityColorize.metal`) with a CPU fallback. |

All views are `NSView` subclasses that use Auto Layout and are created in code (`init(coder:)` is unavailable).

## Usage

```swift
import RememberRemember

let chart = MultiCurveLineChartView()
chart.configure(yAxisLabel: "Magnitude [dB]")
chart.setCurves(xValuesGHz: frequencies,
                curves: [("S11", s11dB), ("S21", s21dB)],
                minRange: (min: -40, max: 0))

let impedance = DualAxisLineChartView()
impedance.setData(xValuesGHz: frequencies,
                  left: (label: "|Z| [Ω]", values: magnitude),
                  right: (label: "∠Z [°]", values: phase))

let smith = SmithChartView()
smith.setData(port: 1, reGamma: re, imGamma: im, vswrMarginGamma: 0.333)

let eye = EyeDiagramView()
eye.setData(timeUI: time, traces: bitSlices)
```

`minRange` / `leftAxisMinRange` / `rightAxisMinRange` set a window the axis always shows *at least*,
so an outlier near the edge of the excitation bandwidth can't flatten the meaningful part of a curve.
Axis bounds and gridlines land on round values (`NiceAxisRange`, Heckbert's nice-numbers algorithm).

## Interaction

Line charts support:

- **Hover** — readout of the nearest curve's value.
- **Pinch** — zoom X around the pointer; hold **Option** to zoom Y.
- **Two-finger scroll** — pan when zoomed; otherwise passes through to the enclosing scroll view.
- **Double-click** — reset to the full data range.
- **Click a legend entry** — hide or show that curve.

## Building

Open `RememberRemember.xcodeproj` and build the `RememberRemember` scheme, or include the project in
a parent workspace (as kiems does) and link the framework. Requires macOS 26.5 and Metal.
