# Architecture: codex-evaluation-benchmark

This document explains how the three pipeline sections connect and the data flow through the system.

---

## **High-Level Pipeline**

```
Real (or Simulated) Developer Telemetry
    ↓
01. Developer Telemetry Simulation
    ├─ Generates: 11K+ events with model_version, latency, acceptance, etc.
    ├─ Output: telemetry_events.csv
    └─ Feeds into: 02, 03
    ↓
02. Code Evaluation Pipeline
    ├─ Generates: code solutions for 5 tasks
    ├─ Evaluates: correctness, edit distance, compilation
    ├─ Output: code_eval_results.json
    └─ Feeds into: dashboard (optional)
    ↓
03. Developer Productivity Analysis
    ├─ Ingests: telemetry_events.csv
    ├─ Analyzes: acceptance modeling, A/B testing, causal inference, NLP
    ├─ Outputs: Multiple (SQL db, plots, results CSV)
    └─ Feeds into: dashboard, business decisions
    ↓
Dashboard (Streamlit)
    └─ Visualizes: All analyses for stakeholders
```

---

## **Module Breakdown**

### **01: Developer Telemetry Simulation**

**Purpose:** Generate realistic developer telemetry data when real data is unavailable.

**Key Files:**
- `simulate_telemetry.py` → Main simulation engine
- `telemetry_schema.md` → Data contract (what each column means)
- `sample_output_head.csv` → Preview of generated data

**Data Generated:**
- **11,106 events** per run (configurable)
- **Dimensions:** 
  - Model versions (v1, v2)
  - User segments (beginner, intermediate, expert)
  - Languages (Python, JavaScript, Go, Rust, TypeScript)
  - Latencies (uniformly 500-1500ms)
  - Acceptance rates (biased by model version, user segment)
  - Task outcomes (compilation, test pass rates)

**Output:** `telemetry_events.csv` (1.3MB)
- One row per suggestion
- Includes: timestamps, model_version, accepted, latency_ms, language, user_segment, etc.

**Key Design:** Realistic but synthetic. Distributions are hand-picked, not fit to real data. This is intentional and documented.

---

### **02: Code Evaluation Pipeline**

**Purpose:** Generate code solutions and measure quality metrics (correctness, similarity to reference).

**Key Files:**
- `generate_code.py` → Uses model to generate solutions
- `tasks/tasks.json` → 5 programming tasks (fizzbuzz, binary_search, etc.)
- `code_solutions/` → Reference implementations
- `run_tests.py` → Unit tests + edit distance computation
- `evaluate_code_quality.py` → Semantic analysis

**Metrics Computed:**
- **Correctness:** Pass/fail on unit tests
- **Edit Distance:** Levenshtein distance from reference solution
- **Compilation Success:** % of suggested code that compiles

**Output:** `code_eval_results.json`
```json
{
  "task_name": {
    "passed": true,
    "edit_distance": 42,
    "compilation_success": 1
  }
}
```

**Note:** This section is somewhat independent of telemetry. Runs on 5 hardcoded tasks; doesn't consume telemetry data.

---

### **03: Developer Productivity Analysis**

**Purpose:** Analyze developer behavior and measure impact of model improvements.

**Subsections:**

#### **3a. Acceptance Rate Modeling**
**Files:** `acceptance_rate_model.py`
- **Input:** telemetry_events.csv
- **Method:** Logistic Regression with preprocessing
- **Output:** Classification report, AUC, feature importance

#### **3b. SQL Analysis**
**Files:** `run_sql_analysis.py`
- **Input:** telemetry_events.csv → creates SQLite DB
- **Output:** 7 aggregate queries (acceptance by model, user, language, etc.)

#### **3c. A/B Testing Framework**
**Files:** `ab_testing_framework.py`
- **Input:** telemetry_events.csv
- **Method:** Proportion test, t-test, chi-square, power analysis
- **Output:** Statistical test results, confidence intervals, recommendations
- **Key Finding:** Model v2 shows +13.4% acceptance improvement (p < 0.001)

#### **3d. Causal Inference**
**Files:** `causal_inference.py`
- **Input:** telemetry_events.csv
- **Methods:**
  - Propensity Score Matching (PSM)
  - Regression Adjustment
  - Latency impact analysis
- **Output:** Average Treatment Effect (ATE), 95% CI, interpretations

#### **3e. NLP Analysis**
**Files:** `nlp_analysis.py`
- **Input:** Generated code (from 02) + prompts
- **Method:** Semantic similarity, embedding distance
- **Output:** Alignment scores, code quality metrics

---

## **Data Flow Diagram**

```
telemetry_events.csv (11K rows)
    ├─→ acceptance_rate_model.py
    │   └─→ Model trained, AUC computed
    │
    ├─→ run_sql_analysis.py
    │   ├─→ Create telemetry.db (SQLite)
    │   └─→ 7 aggregate queries
    │
    ├─→ ab_testing_framework.py
    │   └─→ Statistical tests (v1 vs v2)
    │
    ├─→ causal_inference.py
    │   └─→ PSM, Regression Adjustment, Latency impact
    │
    └─→ Dashboard (if run)
        └─→ Visualize all results

code_eval_results.json (from section 02)
    └─→ nlp_analysis.py
        └─→ Semantic analysis + visualizations
```

---

## **Dependency Graph**

**Independence:**
- Sections 01 and 02 can run in any order
- Section 03 **requires** telemetry_events.csv from section 01

**Typical Workflow:**
```
01. Simulate telemetry
    ↓
02. Generate & evaluate code (parallel OK)
    ↓
03. Run all productivity analyses
```

---

## **Key Design Decisions**

| Decision | Rationale |
|----------|-----------|
| Synthetic telemetry | Enables reproducible demos without real data |
| CSV-first approach | Simple, inspectable, requires no infrastructure |
| SQLite for analysis | Fast, queryable, built-in Python support |
| Multiple frameworks | Logistic regression + A/B tests + causal inference shows breadth |
| Section 02 independence | Demonstrates code evaluation separately from telemetry |
| Multi-stage output | Each stage saves to file; enables incremental reruns |

---

## **Configuration & Extensibility**

### **Changing Telemetry Volume**
In `simulate_telemetry.py`:
```python
NUM_EVENTS = 11106  # Change this
```

### **Adding New Analysis**
1. Create new .py file in `03-developer-productivity-analysis/`
2. Add function to `app.py`
3. Load telemetry_events.csv in same way as others
4. Output results to CSV/JSON

### **Adding New Task to Code Evaluation**
In `tasks/tasks.json`:
```json
{
  "task_id": "my_task",
  "description": "What to implement",
  "example_input": "test case",
  "example_output": "expected output"
}
```

---

## **File Organization**

```
codex-evaluation-benchmark/
│
├── 01-developer-telemetry-simulation/
│   ├── simulate_telemetry.py        # Main generator
│   ├── telemetry_schema.md          # Data contract
│   ├── sample_output_head.csv       # Data preview
│   └── telemetry_events.csv         # OUTPUT (11K rows)
│
├── 02-code-evaluation-pipeline/
│   ├── tasks/                       # 5 programming tasks
│   ├── code_solutions/              # Reference implementations
│   ├── generate_code.py             # Generate solutions
│   ├── run_tests.py                 # Evaluate + edit distance
│   └── code_eval_results.json       # OUTPUT
│
├── 03-developer-productivity-analysis/
│   ├── acceptance_rate_model.py     # Logistic regression
│   ├── run_sql_analysis.py          # SQL queries
│   ├── ab_testing_framework.py      # A/B tests
│   ├── causal_inference.py          # PSM + regression
│   ├── nlp_analysis.py              # Semantic analysis
│   ├── telemetry.db                 # OUTPUT (SQLite)
│   └── nlp_analysis_results.csv     # OUTPUT
│
├── dashboard/
│   └── app.py                       # Streamlit visualization
│
├── docs/
│   ├── 01_QUICK_START.md            # 5-minute setup
│   ├── 02_DEVELOPER_ANALYTICS_GUIDE.md
│   ├── 03_NLP_ANALYSIS.md
│   ├── 04_SHOWCASE_SUMMARY.md
│   ├── 05_METHODOLOGY.md
│   └── ARCHITECTURE.md              # This file
│
├── app.py                           # Unified launcher
└── README.md                        # Entry point
```

---

## **Audience to Code Mapping**

| Audience | Key Files |
|----------|-----------|
| **Beginners** | README.md → /docs/01_QUICK_START.md |
| **Learners** | /docs/02_DEVELOPER_ANALYTICS_GUIDE.md → individual .py files |
| **Reviewers** | /docs/04_SHOWCASE_SUMMARY.md + /docs/05_METHODOLOGY.md + architecture.md |

---

## **Related Repo**

This evaluation benchmark pairs with `codex-systems-lab`:
- **Systems Lab:** Measures *how models behave* (latency, throughput, KV-cache)
- **This Repo:** Measures *how models impact developers* (acceptance, productivity, code quality)

Together, they form a feedback loop:
```
System measurements → Evaluation assumptions → Developer impact → System priorities
```

---

## **Future Directions**

1. **Connect to real telemetry** — Replace synthetic data with production events
2. **Add real code evaluation** — Use actual user-submitted code + reference solutions
3. **Extend dashboard** — More interactive visualizations, filtering
4. **Integrate with systems-lab** — Use actual latency measurements in acceptance model
5. **A/A testing** — Validate statistical framework with synthetic null results
