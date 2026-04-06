# StrokeGuard AI — File & Component Explanation

## Project Structure

```
ML PBL/
├── HOW_TO_RUN.md              ← Setup & run instructions
├── EXPLANATION.md             ← This file
├── index.html                 ← HTML shell (Vite entry point)
├── package.json               ← Dependencies & scripts
├── vite.config.js             ← Vite + React plugin config
└── src/
    ├── main.jsx               ← Mounts React into #root
    ├── index.css              ← Global fonts, animations, input styling
    ├── data.js                ← All ML data constants + LR scoring logic
    ├── StrokeDashboard.jsx    ← Root component + tab navigation
    └── components/
        ├── OverviewTab.jsx    ← Tab 1: Dataset overview
        ├── ModelArenaTab.jsx  ← Tab 2: Model comparison
        ├── FeatureTab.jsx     ← Tab 3: Feature importance & SHAP
        └── PatientTab.jsx     ← Tab 4: Live risk predictor
```

---

## `src/index.css` — Global Styles

| Rule | Purpose |
|---|---|
| `@import` Google Fonts | Loads **DM Sans** (UI) and **IBM Plex Mono** (numbers) |
| `* { box-sizing }` | Makes all elements size predictably |
| `@keyframes fadeUp` | Fade + slide-up animation used on every tab switch |
| `@keyframes barGrow` | Scale-in animation for Patient Analyzer contribution bars |
| `input[type=range]` | Custom red-thumb slider (overrides ugly browser default) |
| `::-moz-range-thumb` | Firefox equivalent of the slider thumb |
| `::-webkit-scrollbar` | Thin 6px scrollbar with gray rounded thumb |

---

## `src/main.jsx` — Entry Point

```jsx
ReactDOM.createRoot(document.getElementById('root')).render(<StrokeDashboard />)
```

Single job: mounts the root `<StrokeDashboard>` component into `index.html`'s `<div id="root">`.

---

## `src/data.js` — The ML Brain

All pre-computed study data lives here as JavaScript constants. Nothing is fetched from an API.

### Data Constants

| Constant | What it holds |
|---|---|
| `MODEL_RESULTS` | 7-row array — one object per classifier with accuracy, precision, recall, F1, AUC-ROC, specificity, kappa, CV AUC ± std |
| `ROC_DATA` | Each model's AUC + display color; `thick:true` for best model, `dashed:true` for random baseline |
| `CONFUSION_MATRICES` | TN / FP / FN / TP for Logistic Regression and AdaBoost |
| `RF_IMPORTANCE` | 15 features with Gini importance scores from the Random Forest |
| `WATERFALL_PATIENT` | SHAP decomposition for Patient #61 (true positive) — baseline, finalScore, per-feature contributions |
| `RISK_CATEGORIES` | 3 groups (Demographic / Clinical / Lifestyle) with SHAP % contribution |
| `LR_COEFFICIENTS` | Logistic Regression weights in standardized feature space |
| `FEATURE_STATS` | Mean & std of continuous features from training data — needed for z-score normalization |
| `DIGIT_BITMAPS` | 5×7 dot-matrix bitmaps for digits 0–9 and `%` |

### Key Functions

#### `standardize(value, feature)`
Z-score normalizes a raw input value:
```
z = (x - mean) / std
```
Must be applied to `age`, `avg_glucose_level`, and `bmi` before scoring.

#### `computeStrokeRisk(inputs)`
Full logistic regression pipeline in JavaScript:

1. **Standardize** the 3 continuous inputs
2. **One-hot encode** all categorical inputs (gender, married, residence, workType, smoking)
3. **Compute log-odds** = intercept + Σ(coefficient × feature value)
4. **Sigmoid** → `probability = 1 / (1 + e^(-logOdds))`
5. **Sort** per-feature contributions by `|contribution|` for the waterfall chart
6. **Threshold** at **0.28** (not the default 0.5) — tuned to maximize recall so real strokes are not missed

Returns: `{ probability, logOdds, prediction, contributions, threshold }`

---

## `src/StrokeDashboard.jsx` — Navigation Shell

**State:** `activeTab` (string — `'overview'` | `'arena'` | `'features'` | `'patient'`)

**Sticky header** contains:
- Logo: `🧬 StrokeGuard AI`
- 4 pill-shaped navigation buttons — active tab gets white bg + red text + shadow
- Caption showing dataset info (top-right, muted)

**Content area** renders one tab component at a time. `key={activeTab}` forces a fresh mount on switch, retriggering CSS animations.

---

## `src/components/OverviewTab.jsx` — Tab 1

### Hero Stats Row
4 white cards in a CSS grid. Values hardcoded from the study summary:
`5,109 patients` / `4.9% prevalence` / `0.818 AUC` / `Age = top predictor`

### Class Imbalance Visual
Two flexbox divs with `flex: 95.1` and `flex: 4.9` — proportional area representation. The thin red box makes the severity of imbalance visually obvious at a glance.

### Preprocessing Pipeline Stepper
Array of 7 steps rendered as horizontal numbered circles with connecting lines. Step 7 (SMOTE) has `accent: true` → **red circle + glow ring** to signal it's the critical/debated step. A callout below explains why SMOTE must never be applied to the test set.

---

## `src/components/ModelArenaTab.jsx` — Tab 2

### Metrics Table
- Click any column header to sort by that metric
- `bests` object pre-computes the highest value per column → **rendered in bold red**
- **Recall** and **AUC-ROC** columns show inline mini-bars instead of raw numbers (red/blue respectively)
- Logistic Regression row has a red left-border + light pink background (best model callout)

### ROC Curves (Custom SVG)
- Curve shape approximated with: `tpr = fpr ^ ((1 - AUC) / AUC)` — a concave curve whose area under it matches the given AUC
- 20 sample points per curve, drawn as `<path>` SVG elements
- **Hover legend item** → that curve stays at opacity 1.0, all others drop to 0.15

### Confusion Matrices
2×2 grid for Logistic Regression and AdaBoost. Color-coded by clinical severity:
- 🟢 **TN** — correctly cleared (green)
- 🟡 **FP** — false alarm, unnecessary imaging (amber)  
- 🔴 **FN** — **MISSED STROKE** — 4.5-hour tPA window lost (red, highest emphasis)
- 🔵 **TP** — stroke correctly caught (blue)

### CV vs Test AUC Gap
Horizontal bar overlay per model showing CV AUC vs Test AUC. Large gaps flagged in red as SMOTE leakage — when SMOTE is applied before cross-validation splits, synthetic data leaks between folds, inflating CV scores artificially.

---

## `src/components/FeatureTab.jsx` — Tab 3

### RF Gini Importance Bars
Horizontal bars, width proportional to importance (max = 0.298 for `avg_glucose_level`). Top 3 features are bolded. Color gradient from deep red → light pink.

### SHAP Beeswarm (SVG)
Simulates a standard SHAP beeswarm summary plot:
- 8 rows (one per feature), 30 manually-placed dots each
- X-axis = SHAP value (−6 to +6)
- Dot color: **red** = high feature value, **blue** = low feature value
- Positions designed to match the real patterns from the study (e.g., `age` dots cluster heavily to the right)

### SHAP Waterfall — Patient #61
A confirmed true positive stroke case. Bars extend:
- **Right (red)** = contribution pushes prediction toward stroke
- **Left (blue)** = contribution pushes prediction away from stroke

High glucose (+1.13) and smoking (+0.87) dominate as the strongest drivers.

### Risk Category Donut Chart
SVG donut with 3 arcs (Demographic / Clinical / Lifestyle). Uses `polarToXY()` to convert angles to SVG coordinates. Key insight: Demographic factors (age, gender) contribute 42.1% — **non-modifiable** — but Clinical + Lifestyle together cover 57.9% — **actionable**.

---

## `src/components/PatientTab.jsx` — Tab 4

### Input Form Components

| Component | Input Type | Description |
|---|---|---|
| `SliderField` | `<input type="range">` | Continuous inputs (age, BMI, glucose); value shown live |
| `ToggleField` | CSS pill div | Binary toggles (hypertension, heart disease, married); animates on/off |
| `RadioField` | Styled buttons | Binary choice (gender, residence type) |
| `SelectField` | `<select>` | Multi-option lists (work type, smoking status) |

Default values represent a realistic high-risk patient: age=67, glucose=230, hypertension=Yes, heartDisease=Yes.

### `DotMatrixDisplay` Component
The centrepiece UI element — renders a number like `77%` as a retro dot-matrix display:
1. Split string into individual characters (`'7'`, `'7'`, `'%'`)
2. For each character, look up its 35-bit bitmap in `DIGIT_BITMAPS`
3. Render a 5×7 CSS grid of `<div>` circles — lit (colored) or unlit (gray)
4. On mount, `requestAnimationFrame` loop counts from 0 → target over **1.5 seconds** with cubic ease-out

### Feature Contribution Bars
All 10 features rendered sorted by `|contribution|`. Positive bars (red) extend left→right; negative bars (blue) extend right→left. Input value shown beside each label.

### Clinical Recommendation Box
Dynamic based on result:
- **HIGH RISK**: shows top 2 positive contributors, recommends cardiovascular workup
- **LOW RISK**: shows top contributor, recommends annual screening

Footer note explains the 0.28 threshold choice and clinical disclaimer.

---

## Design System

| Token | Value | Used For |
|---|---|---|
| Background | `#F4F5F7` | Page background |
| Card | `#FFFFFF` | All cards and panels |
| Accent Red | `#E8323A` | CTAs, HIGH RISK, highlights |
| Red Dim | `#FDECEA` | Callout backgrounds |
| Text Primary | `#1A1A2E` | Headings, key values |
| Text Secondary | `#6B7280` | Labels, descriptions |
| Text Muted | `#9CA3AF` | Captions, metadata |
| Success | `#10B981` | LOW RISK, TN cells |
| Info Blue | `#3B82F6` | AUC bars, negative SHAP |
| Warning | `#F59E0B` | FP cells |

**Fonts:**
- `DM Sans` — all UI text, headings, labels
- `IBM Plex Mono` — all numeric values, metric numbers, log-odds readouts
