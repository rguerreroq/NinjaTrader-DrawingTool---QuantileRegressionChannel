# NinjaTrader-DrawingTool---QuantileRegressionChannel
End-to-end implementation of a Quantile Regression Channel in NinjaTrader 8, focused on robust statistics, numerical stability, and production-level chart tooling.
# Quantile Regression Channel for NinjaTrader 8

Production-oriented **Quantile Regression Channel** drawing tool for NinjaTrader 8, built for robust trend analysis under non-Gaussian market behavior (heavy tails, outliers, skewed moves).

It draws three lines from a selected bar range:

- **Upper quantile line** (e.g., 0.95)
- **Median quantile line** (0.50)
- **Lower quantile line** (e.g., 0.05)

Unlike a classical regression channel (OLS + standard deviation bands), this tool uses **quantile regression** to model conditional distribution levels directly.

---

## Why this tool?

Classical linear regression channels assume a symmetric error structure around the mean and rely on standard deviation scaling. In real market data, returns are often asymmetric and heavy-tailed.

This implementation is designed to be more robust by:

- targeting **quantiles** instead of mean ± k·σ
- reducing sensitivity to large outliers
- offering a clearer view of tail behavior (upper/lower distribution envelopes)

---

## Core Features

- Quantile channel with configurable:
  - `LowerQuantile` (default `0.05`)
  - `UpperQuantile` (default `0.95`)
  - median fixed at `0.50`
- Configurable `PriceType`:
  - Close, Open, High, Low, Median, Typical, Weighted
- Extend lines left/right (`ExtendLeft`, `ExtendRight`)
- Built-in alert conditions:
  - Upper line
  - Median line
  - Lower line
- Caching for efficient redraws during interaction
- Numeric stability guards and finite-value checks
- Native-like drawing behavior (anchors, move/edit/hit-test flow)

---

## Mathematical Model

For each quantile level \\(\tau \in (0,1)\\), the line is:

\\[
\hat{y}_{\tau}(x) = \beta_{0,\tau} + \beta_{1,\tau}x
\\]

Estimated by minimizing the pinball loss:

\\[
\min_{\beta_0,\beta_1}\sum_{i=1}^{n}\rho_{\tau}\left(y_i - (\beta_0+\beta_1x_i)\right)
\\]

with

\\[
\rho_{\tau}(u)=
\begin{cases}
\tau u, & u\ge0 \\\\
(\tau-1)u, & u<0
\end{cases}
\\]

Current implementation solves this via **IRLS-style iterative weighted least squares** approximation.

---

## Quantile Channel vs Classical Regression Channel

### Classical channel
- center: OLS mean regression line
- bands: mean ± k·std(residuals)
- sensitive to outliers and variance spikes

### Quantile channel
- center: median quantile line (τ = 0.50)
- bands: direct upper/lower quantile lines (e.g., τ = 0.95 and τ = 0.05)
- more robust under asymmetry and heavy tails

---

## Parameters (Drawing Tool UI)

### Parameters
- **Price Type**: source price used by the fit
- **Lower Quantile**: lower τ in (0.01, 0.49)
- **Upper Quantile**: upper τ in (0.51, 0.99)
- **Iterations**: IRLS iterations (3–300)

### Visual
- **Extend Left**
- **Extend Right**
- **Upper Channel** (stroke style)
- **Median Channel** (stroke style)
- **Lower Channel** (stroke style)

---

## Installation (NinjaTrader 8)

1. Open **NinjaTrader 8**.
2. Go to **New > NinjaScript Editor**.
3. Create a new Drawing Tool file (or paste into your custom drawing tool file).
4. Paste the `QuantileRegressionChannel` code.
5. Compile (`F5`).
6. Open a chart and add the drawing tool from drawing tools list.

> If compilation fails, verify:
> - namespace is `NinjaTrader.NinjaScript.DrawingTools`
> - all required `using` directives are present
> - no ambiguity between `System.Windows.Point` and `SharpDX` types

---

## Usage

1. Select **Quantile Regression Channel** from drawing tools.
2. Click start anchor on chart.
3. Click end anchor on chart.
4. Adjust parameters in properties panel:
   - choose quantiles (e.g., 0.1 / 0.9, or 0.05 / 0.95)
   - increase iterations if needed
   - toggle line extensions

---

## Alerts

The tool exposes three alert condition items:

- `Upper Quantile Line`
- `Median Quantile Line`
- `Lower Quantile Line`

Use NinjaTrader alert UI to trigger on:
- Greater / Less / CrossAbove / CrossBelow, etc.

---

## Performance Notes

- Computed channel prices are cached and recalculated only when necessary:
  - anchor movement
  - parameter changes
  - price type changes
  - bar updates
- This reduces render overhead during interaction.

---

## Limitations (Current Version)

- IRLS provides an approximation to quantile regression; in extreme edge cases, convergence quality depends on:
  - selected quantiles
  - sample length
  - market regime
- Very short ranges can produce unstable slope estimates.
- Current anti-crossing logic is a numeric safeguard, not a constrained optimizer.

---

## Roadmap

- [ ] Add optional **strict monotonic quantile enforcement** across x-domain
- [ ] Add optional **warm-start + adaptive stopping** for faster convergence
- [ ] Add optional **Huberized/regularized quantile fit**
- [ ] Add diagnostics panel:
  - objective value
  - iteration count
  - convergence flag
- [ ] Add unit/integration tests for numerical edge cases

---

## Repository Structure (Suggested)

```text
.
├─ src/
│  └─ QuantileRegressionChannel.cs
├─ docs/
│  ├─ math-notes.md
│  └─ screenshots/
├─ examples/
│  └─ parameter-presets.md
└─ README.md
