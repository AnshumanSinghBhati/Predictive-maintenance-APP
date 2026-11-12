# Predictive Maintenance of Industrial Machines

A single-file, interactive web app that estimates the **probability of machine failure** from five sensor readings. A small Random Forest is embedded directly in the page and evaluated in JavaScript in the browser — there is no backend and no API call for inference. The same app is also packaged as an Android APK.

#Repository Link :https://github.com/AnshumanSinghBhati/Predictive-maintenance-APP

## Repository contents

```
.
├── index.html        # The complete app (HTML + CSS + JS + embedded model) — open this
├── src/
│   ├── styles.css    # All CSS, extracted from index.html
│   ├── app.js        # All application JavaScript, extracted from index.html
│   └── model.json    # The embedded Random Forest + dataset stats, extracted from index.html
├── apk/
│   └── Predictive_Maintenance_of_Industrial_Machines.apk   # Android build
└── README.md
```

`index.html` is the source of truth. The files in `src/` are the same code split out so it is easy to read and diff — `styles.css` and `app.js` are byte-for-byte what is inside `index.html`, and `model.json` is the parsed content of its `<script id="DATA">` block.

---

## What the app does

The interface has three tabs.

**Dashboard**
- A semicircular gauge and percentage showing the predicted failure risk, with a status label (Normal / Low / Moderate / High / Critical).
- Five sliders, one per sensor input, each showing the dataset min, median and max.
- Six one-tap presets: *Normal machine, Moderate wear, High temperature, High tool wear, High-speed op, Reset inputs*.
- KPI tiles (current risk, status, and the model's test-set F1 / recall / ROC-AUC, plus dataset size).
- A "Why is the risk changing?" panel ranking the inputs furthest from the dataset median, weighted by the model's feature importance. This is a heuristic for association, not proof of causation.

**Analytics**
- Doughnut chart of normal vs. failure rows in the dataset.
- Bar chart of the Random Forest's feature importance.
- A risk curve: pick a parameter and the app sweeps it across its observed range through the real model, holding the other sliders fixed.
- A histogram of the dataset for a chosen parameter, with the current slider value highlighted.

**Model & Dataset**
- Dataset facts, model metrics (precision, recall, F1, ROC-AUC, PR-AUC, confusion matrix) and a "limitations & honesty notes" section.

### Inputs

| Slider | Unit | Dataset range | Median |
|---|---|---|---|
| Air Temperature | K | 297.02 – 305.53 | 299.22 |
| Process Temperature | K | 306.11 – 316.56 | 309.09 |
| Rotational Speed | rpm | 800 – 3000 | 1493 |
| Torque | Nm | 3.0 – 83.27 | 39.89 |
| Tool Wear | min | 0 – 253 | 92 |

---

## The model

- **Algorithm:** Random Forest, 8 trees, max depth 4. The app notes it was selected over Logistic Regression on F1 / PR-AUC.
- **Output:** the mean of each tree's leaf value, i.e. a probability between 0 and 1.
- **Feature importance (measured):** Tool wear 63.7%, Rotational speed 15.4%, Torque 14.3%, Air temperature 4.9%, Process temperature 1.8%.
- **Risk bands** (chosen for readability, not statistically derived): Normal < 5%, Low < 15%, Moderate < 35%, High < 65%, Critical ≥ 65%.

### Held-out test-set metrics (n = 1,500)

| Precision | Recall | F1 | ROC-AUC | PR-AUC |
|---|---|---|---|---|
| 0.3897 | 0.7910 | 0.5222 | 0.9013 | 0.4926 |

Confusion matrix (test): TN 1350 · FP 83 · FN 14 · TP 53. Split: 7,000 train / 1,500 validation / 1,500 test.

### How the trees are stored

Each tree in `src/model.json` is one flat array. The first number is the node count; after that, every node takes 5 consecutive values:



Feature indices follow this order: `0` Air temperature, `1` Process temperature, `2` Rotational speed, `3` Torque, `4` Tool wear. A node with `left = -1` is a leaf and `value` is its failure probability. Evaluation (`evalTree` in `src/app.js`) walks from node 0, going left when `x[feature] <= threshold`.

### Example outputs of the built-in presets

Computed by running the app's own `predict()` function on its own preset values:

| Preset | What changes | Risk | Status |
|---|---|---|---|
| Normal machine | Median inputs, tool wear at 5th percentile (9 min) | 23.8% | Moderate Risk |
| Moderate wear | Median inputs, tool wear 150 min | 20.7% | Moderate Risk |
| High temperature | Air & process temp at 95th pct, speed at 5th pct | 39.8% | High Risk |
| High tool wear | Torque and tool wear at 95th pct | 74.8% | Critical |
| High-speed op | Speed at 95th pct, torque at 5th pct | 31.7% | Moderate Risk |
| Reset inputs | All medians | 23.7% | Moderate Risk |

---

## Dataset

Based on the **AI4I 2020 Predictive Maintenance Dataset** — UCI Machine Learning Repository, DOI [10.24432/C5HS5C](https://doi.org/10.24432/C5HS5C), licensed CC BY 4.0 (S. Matzka, 2020).



## Running it

**In a browser** — open `index.html`

**On Android** — install `apk/Predictive_Maintenance_of_Industrial_Machines.apk` 

> **Internet is needed on first load.** `index.html`

## About the APK

The APK was produced by converting `index.html` with an HTML-to-APK tool: it is a thin Android WebView wrapper (package `com.htmltoapk.predictivemaintenanceofindustr`) that loads the page from `assets/www/index.html`.

- The bundled `assets/www/index.html` is **identical** to the `index.html` in this repo (same SHA-256, below).

