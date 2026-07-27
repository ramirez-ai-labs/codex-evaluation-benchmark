# Contributing to codex-evaluation-benchmark

Thank you for interest in extending this evaluation framework! This guide explains how to add new metrics, extend the pipeline, and contribute improvements.

---

## **Quick Start for Contributors**

1. Fork/clone the repo
2. Create a feature branch: `git checkout -b feature/new-metric`
3. Add your code to appropriate section (01, 02, or 03)
4. Update documentation and run end-to-end test
5. Submit pull request with description

---

## **Adding New Metrics or Analyses**

### **Which Section?**

- **01-developer-telemetry-simulation/** — Data generation, synthetic telemetry
- **02-code-evaluation-pipeline/** — Code quality metrics (correctness, edit distance, compilation)
- **03-developer-productivity-analysis/** — Developer behavior analysis (acceptance, causal inference, A/B testing)

### **Adding a New Metric to Section 03**

Create a new Python module following this structure:

```python
"""
[Metric Name] Analysis
======================
Brief description of what is measured and why it matters.

Methodology:
- What does this metric capture?
- How is it computed?
- Why is it useful for developer tools?

Use Cases:
- When should teams use this metric?
- What decisions does it inform?
"""

import pandas as pd
import numpy as np
from pathlib import Path
from typing import Dict
import warnings
warnings.filterwarnings('ignore')

def load_data(csv_path: str = None) -> pd.DataFrame:
    """Load telemetry data."""
    if csv_path is None:
        root = Path(__file__).parent.parent
        csv_path = root / "01-developer-telemetry-simulation" / "telemetry_events.csv"
    
    if not Path(csv_path).exists():
        raise FileNotFoundError(f"Telemetry data not found at {csv_path}")
    
    return pd.read_csv(csv_path)

def compute_metric(df: pd.DataFrame) -> Dict:
    """
    Compute the metric.
    
    Args:
        df: DataFrame with telemetry data
        
    Returns:
        Dictionary with results and interpretation
    """
    result = {
        "metric_name": "Your Metric",
        "value": 0.0,
        "interpretation": "What this number means"
    }
    
    return result

def print_results(result: Dict):
    """Pretty-print metric results."""
    print("\n" + "="*70)
    print(result["metric_name"].upper())
    print("="*70)
    print(f"Value: {result['value']}")
    print(f"Interpretation: {result['interpretation']}")
    print("="*70 + "\n")

def main():
    """Run metric computation."""
    try:
        df = load_data()
        result = compute_metric(df)
        print_results(result)
    except FileNotFoundError as e:
        print(f"❌ {e}")
        print("\nRun telemetry simulation first:")
        print("   python app.py simulate")

if __name__ == "__main__":
    main()
```

---

## **Adding to app.py**

Update the launcher to call your new metric:

```python
def my_new_metric():
    """Run your new metric."""
    metric_file = ROOT / "03-developer-productivity-analysis" / "my_metric.py"
    run([sys.executable, str(metric_file)])

# Add to run_all():
def run_all():
    # ... existing stages ...
    my_new_metric()  # Add your stage
```

---

## **Adding to 01: Telemetry Simulation**

To add new fields to simulated data:

1. Update `telemetry_schema.md` to document the new field
2. Add generation logic to `simulate_telemetry.py`
3. Update sample outputs in `sample_output_head.csv`
4. Document assumptions in code comments

Example:
```python
# In simulate_sessions() or similar
data["new_field"] = generate_new_field(...)

# In telemetry_schema.md
| new_field | string | [description of what this field captures] |
```

---

## **Adding to 02: Code Evaluation**

To add new code quality metrics:

1. Add metric computation to pipeline:
```python
def compute_new_metric(solution_code, reference_code) -> float:
    """Compute the metric."""
    return score

# Then integrate into run_tests.py
results[task_id]["new_metric"] = compute_new_metric(...)
```

2. Document in `evaluation_report.md`
3. Update `code_eval_results.json` schema

---

## **Testing Your Changes**

Before submitting, verify the full pipeline works:

```bash
# Run telemetry simulation
python app.py simulate

# Run your specific module
python 03-developer-productivity-analysis/my_metric.py

# Run full pipeline
python app.py all
```

**Verify:**
- ✅ No FileNotFoundError or import errors
- ✅ Output is printed and looks reasonable
- ✅ Documentation is clear (docstrings, inline comments)
- ✅ Results are saved to appropriate file (CSV, JSON, DB)

---

## **Code Style & Documentation**

- Use type hints on function signatures
- Write docstrings for every function explaining purpose, args, returns
- Add inline comments explaining *why*, not just *what*
- Use descriptive variable names
- Group related functions with section headers

Example:
```python
# ============================================================
# PROPENSITY SCORE MATCHING
# ============================================================

def estimate_ate_psm(df: pd.DataFrame) -> Dict:
    """
    Estimate Average Treatment Effect using Propensity Score Matching.
    
    This technique accounts for selection bias by matching treated and
    control units with similar propensity scores.
    
    Args:
        df: DataFrame with treatment, outcome, and confounders
        
    Returns:
        Dictionary with ate, ci_95_lower, ci_95_upper
    """
```

---

## **Documentation Requirements**

All new contributions should update relevant docs:

| Change | Update |
|--------|--------|
| New metric in section 03 | `/docs/`, module README, code comments |
| New telemetry field | `telemetry_schema.md`, docstring |
| New evaluation metric | `evaluation_report.md`, README |
| Architecture change | `/docs/ARCHITECTURE.md` |

---

## **Commit Message Guidelines**

```
[type] Brief description

Longer explanation of what and why.

Example:
[feature] Add latency percentile analysis

Computes p50, p95, p99 latency by model version.
Useful for understanding tail latency impact on UX.
```

Types: `[feature]`, `[fix]`, `[docs]`, `[refactor]`, `[test]`

---

## **Pull Request Checklist**

Before submitting, ensure:

- [ ] Code follows style guidelines (docstrings, type hints, comments)
- [ ] Full pipeline runs without errors (`python app.py all`)
- [ ] New metrics have clear interpretation text
- [ ] Documentation is updated
- [ ] Commit message is descriptive
- [ ] No hardcoded paths (use `Path(__file__).parent.parent`)

---

## **Asking Questions**

- **How should I structure this metric?** Open an issue with description
- **What data should I use?** Check `telemetry_schema.md` for available fields
- **Should I add this to the dashboard?** File an issue to discuss UX impact
- **How do I test locally?** Run `python app.py all` to verify end-to-end

---

## **Philosophy**

This repo demonstrates applied ML and data science in developer tools context. Contributions should:

- ✅ Be grounded in real use cases (e.g., "teams need to measure X for Y decision")
- ✅ Include clear interpretation (e.g., "this metric increases by X% when...")
- ✅ Have beginner-friendly explanations
- ✅ Include confidence intervals or uncertainty estimates where appropriate
- ✅ Acknowledge limitations

---

## **Code of Conduct**

- Be respectful and constructive
- Welcome contributions from all backgrounds
- Focus on learning and teaching
- Assume good intent

Thank you for contributing!
