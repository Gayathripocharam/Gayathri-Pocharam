# 🧬 StrokeGuard AI

> An interactive machine learning dashboard for stroke risk prediction, built as a Project-Based Learning (PBL) submission.

![React](https://img.shields.io/badge/React-19-61DAFB?logo=react&logoColor=white)
![Vite](https://img.shields.io/badge/Vite-7-646CFF?logo=vite&logoColor=white)
![License](https://img.shields.io/badge/License-MIT-green)

---

## 📌 Overview

**StrokeGuard AI** is a fully client-side React dashboard that visualizes a complete ML study on stroke prediction using the [Kaggle Stroke Dataset](https://www.kaggle.com/datasets/fedesoriano/stroke-prediction-dataset) (5,109 patients). It implements an in-browser **Logistic Regression scoring pipeline** so users can interactively compute their own stroke risk in real time — no backend needed.

---

## ✨ Features

| Tab | What It Shows |
|-----|--------------|
| 📊 **Dataset Overview** | Class imbalance visualization, preprocessing pipeline (SMOTE explanation), key dataset stats |
| 🏆 **Model Arena** | Side-by-side comparison of 7 classifiers (accuracy, precision, recall, F1, AUC-ROC, Cohen's Kappa), sortable metrics table, custom SVG ROC curves, confusion matrices |
| 🔬 **Feature Insights** | Random Forest Gini importance bars, SHAP beeswarm simulation, SHAP waterfall for a real patient, risk category donut chart |
| 🩺 **Patient Analyzer** | Live logistic regression scoring with sliders and toggles, retro dot-matrix risk display, per-feature contribution bars, clinical recommendation |

---

## 🤖 ML Pipeline (In-Browser)

The `src/data.js` file contains the full trained model's weights:

1. **Z-score normalize** continuous inputs (age, glucose, BMI) using training set statistics
2. **One-hot encode** categorical inputs (gender, work type, smoking status, etc.)
3. **Compute log-odds** using Logistic Regression coefficients
4. **Sigmoid** to get probability
5. **Threshold at 0.28** (tuned for high recall — missed strokes are more costly than false alarms)

**Best model: Logistic Regression** — AUC-ROC 0.818, Recall 0.776

---

## 🛠️ Tech Stack

| Tool | Purpose |
|------|---------|
| React 19 | UI framework |
| Vite 7 | Dev server & bundler |
| Recharts | Chart components |
| Lucide React | Icon set |
| DM Sans + IBM Plex Mono | Typography (Google Fonts) |
| Vanilla CSS | All styling (no Tailwind) |

---

## 🚀 Getting Started

### Prerequisites
- [Node.js](https://nodejs.org/) v18+
- npm (comes with Node.js)

### Installation

```bash
# 1. Clone the repository
git clone https://github.com/Gayathripocharam/strokeguard-ai.git
cd strokeguard-ai

# 2. Install dependencies
npm install

# 3. Start the dev server
npm run dev
```

Open [http://localhost:5173](http://localhost:5173) in your browser.

---

## 📁 Project Structure

```
src/
├── main.jsx               ← React entry point
├── index.css              ← Global styles & animations
├── data.js                ← All ML data + LR scoring logic
├── StrokeDashboard.jsx    ← Root component + tab navigation
└── components/
    ├── OverviewTab.jsx    ← Dataset overview
    ├── ModelArenaTab.jsx  ← Model comparison
    ├── FeatureTab.jsx     ← Feature importance & SHAP
    └── PatientTab.jsx     ← Live risk predictor
```

> See [EXPLANATION.md](./EXPLANATION.md) for detailed component documentation.

---

## 📜 Dataset

- **Source:** [Kaggle — Stroke Prediction Dataset](https://www.kaggle.com/datasets/fedesoriano/stroke-prediction-dataset)
- **Size:** 5,109 patients
- **Class imbalance:** 4.9% positive (stroke) vs 95.1% negative
- **Features:** Age, gender, hypertension, heart disease, avg glucose level, BMI, work type, smoking status, and more

---

## 👩‍💻 Author

**Gayathri Pocharam**  
[GitHub](https://github.com/Gayathripocharam)

---

## ⚠️ Disclaimer

This tool is for **educational purposes only** and does not constitute medical advice. Always consult a qualified healthcare professional for medical decisions.
