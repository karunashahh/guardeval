# GuardEval

**GuardEval** is an automated evaluation framework for testing security guardrails used in LLM-powered agents.

The project evaluates a hybrid guard architecture that combines:

1. Deterministic rule-based input filtering
2. Semantic security classification
3. Centralized risk policy enforcement
4. Automated benchmark evaluation
5. Baseline comparison
6. Interactive Streamlit visualization
7. CI-oriented quality gates

The framework is designed to measure both security detection and false-positive behavior rather than relying only on individual examples.

---

## Key Features

- Prompt-injection detection
- Jailbreak detection
- Data-exfiltration detection
- Tool-abuse detection
- Malicious-code detection
- Deterministic rule guard
- Semantic-risk classifier
- Hybrid security guard
- Precision, recall, F1, accuracy, specificity
- False-positive and false-negative rates
- Confusion-matrix analysis
- Category-level evaluation
- Decision-method coverage
- Baseline comparison
- Controlled benchmark datasets
- Unseen holdout evaluation
- JSON evaluation reports
- Streamlit dashboard
- CI quality gate

---

# Architecture

```text
User Prompt
     │
     ▼
┌─────────────────────┐
│   Rule-Based Guard  │
│                     │
│ Injection patterns  │
│ Sensitive requests  │
│ Dangerous actions   │
└──────────┬──────────┘
           │
      Block? ──────── Yes ──────► Policy ──► BLOCK
           │
           │ No
           ▼
┌─────────────────────┐
│ Semantic Guard      │
│                     │
│ Mock classifier     │
│ or real LLM         │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│    Risk Policy      │
│                     │
│ Final decision      │
└──────────┬──────────┘
           │
           ▼
       ALLOW / BLOCK
```
The semantic layer can operate in two modes:

* mock — deterministic local semantic-risk simulator
* real — OpenAI-backed semantic classifier

##  Security Benchmark

GuardEval uses controlled benchmark datasets containing benign requests and security attacks.

The primary development benchmark contains:

* 400 total samples
* 380 benign samples
* 20 attack samples
* 95% benign / 5% attack distribution
* 4 examples per attack category

Attack categories:

* Prompt injection
* Data exfiltration
* Tool abuse
* Jailbreak
* Malicious code

The 400-sample benchmark is a controlled evaluation dataset, not 400 independent real-world observations. Some benign prompts are reused across controlled variants to test consistency.

400-Sample Development Benchmark

The development benchmark was evaluated using:

```bash
GUARDEVAL_DATASET=dataset/test_realistic_95_5_400.jsonl \
GUARDEVAL_LLM_MODE=mock \
python -m evaluation.evaluator
```
## Results

| System | Accuracy | Attack Detection | Precision | F1 | FPR | FNR |
|---|---:|---:|---:|---:|---:|---:|
| No Guard | 95.00% | 0.00% | 0.00% | 0.00% | 0.00% | 100.00% |
| Rule Only | 96.75% | 35.00% | 100.00% | 51.85% | 0.00% | 65.00% |
| Hybrid Guard | 100.00% | 100.00% | 100.00% | 100.00% | 0.00% | 0.00% |

### Hybrid Confusion Matrix

| | Actual Benign | Actual Attack |
|---|---:|---:|
| **Predicted Benign** | 380 | 0 |
| **Predicted Attack** | 0 | 20 |

| Metric | Count |
|---|---:|
| True Negatives | 380 |
| False Positives | 0 |
| False Negatives | 0 |
| True Positives | 20 |

The hybrid guard detected all 20 attacks in this controlled development benchmark while producing no false positives.

> **Important:** This result should not be interpreted as general real-world security performance.

## Unseen 200-Sample Holdout Benchmark

To test generalization, GuardEval evaluates the hybrid guard on a separate 200-sample holdout dataset that was not used to develop or modify the classifier.

The holdout contains:

| Property | Value |
|---|---:|
| Total samples | 200 |
| Benign samples | 180 |
| Attack samples | 20 |
| Benign / Attack | 90% / 10% |
| Attack categories | 5 |
| Examples per category | 4 |
| ID range | 1001–1200 |

The classifier was **not modified after observing the holdout results**.

### Run

```bash
GUARDEVAL_DATASET=dataset/test_holdout_200.jsonl \
GUARDEVAL_LLM_MODE=mock \
python -m evaluation.evaluator
```

## Holdout Results

| Metric | Result |
|---|---:|
| Accuracy | 98.00% |
| Precision | 100.00% |
| Attack Detection / Recall | 80.00% |
| F1 Score | 88.89% |
| False Positive Rate | 0.00% |
| False Negative Rate | 20.00% |
| Specificity | 100.00% |

### Confusion Matrix

| | Actual Benign | Actual Attack |
|---|---:|---:|
| **Predicted Benign** | 180 | 4 |
| **Predicted Attack** | 0 | 16 |

### Attack-Category Performance

| Attack Category | Detection |
|---|---:|
| Prompt Injection | 50.00% |
| Data Exfiltration | 75.00% |
| Tool Abuse | 100.00% |
| Jailbreak | 100.00% |
| Malicious Code | 75.00% |

The holdout demonstrates that the deterministic semantic simulator does not perfectly generalize to unseen attack phrasing.

In particular, prompt-injection, data-exfiltration, and malicious-code variants produced false negatives even though the same broad attack categories were detected on the development benchmark.

> **Holdout finding:** The current semantic simulator achieves 80% attack detection on the unseen holdout, compared with 100% on the controlled development benchmark. The difference shows that the current pattern-based classifier does not consistently recognize all unseen phrasings of the evaluated attack categories.

This difference is important because it demonstrates why benchmark performance on a controlled development dataset should not be treated as equivalent to real-world security performance.
## Baseline Comparison

GuardEval evaluates simpler baselines alongside the hybrid architecture on the controlled 400-sample development benchmark.

### No Guard

A system with no security guardrail allows every request.

| Metric | Result |
|---|---:|
| Accuracy | 95.00% |
| Attack Detection | 0.00% |
| Precision | 0.00% |
| F1 Score | 0.00% |
| False Positive Rate | 0.00% |
| False Negative Rate | 100.00% |

#### Confusion Matrix

| Metric | Count |
|---|---:|
| True Negatives | 380 |
| False Positives | 0 |
| False Negatives | 20 |
| True Positives | 0 |

The high accuracy is caused by the 95/5 class distribution and does not indicate useful attack detection.

### Rule-Only Guard

The deterministic rule guard uses explicit security-related patterns.

| Metric | Result |
|---|---:|
| Accuracy | 96.75% |
| Attack Detection | 35.00% |
| Precision | 100.00% |
| F1 Score | 51.85% |
| False Positive Rate | 0.00% |
| False Negative Rate | 65.00% |

#### Confusion Matrix

| Metric | Count |
|---|---:|
| True Negatives | 380 |
| False Positives | 0 |
| False Negatives | 13 |
| True Positives | 7 |

The rule-only guard misses attacks whose wording does not match its explicit patterns.

### Hybrid Guard

The hybrid architecture combines deterministic rules with semantic classification.

#### 400-Sample Development Benchmark

| Metric | Result |
|---|---:|
| Accuracy | 100.00% |
| Attack Detection | 100.00% |
| Precision | 100.00% |
| F1 Score | 100.00% |
| False Positive Rate | 0.00% |
| False Negative Rate | 0.00% |

#### 200-Sample Unseen Holdout

| Metric | Result |
|---|---:|
| Accuracy | 98.00% |
| Attack Detection | 80.00% |
| Precision | 100.00% |
| F1 Score | 88.89% |
| False Positive Rate | 0.00% |
| False Negative Rate | 20.00% |

The difference between the development and holdout evaluations illustrates the importance of testing on data that was not used during classifier development.

---

## Benchmark Configuration

| Property | Value |
|---|---|
| Dataset | `dataset/test_realistic_95_5_400.jsonl` |
| Samples | 400 |
| Benign | 380 |
| Attacks | 20 |
| Benign / Attack Distribution | 95% / 5% |
| LLM Mode | `mock` |
| Configured Model | `gpt-5.6-luna` |

The current benchmark uses the deterministic mock semantic classifier.

The configured real model is `gpt-5.6-luna`, but the development and holdout benchmarks reported above did **not** call the OpenAI API.

---

## Mock Semantic Classifier

The current semantic component is a deterministic local simulator.

It is **not an actual LLM**.

The mock classifier uses pattern-based rules to approximate semantic-risk detection so that the evaluation framework can be tested without requiring API access or credits.

The real OpenAI-backed semantic classifier is implemented separately and can be enabled with:

```bash
GUARDEVAL_LLM_MODE=real
```

The current reported benchmark results should therefore be interpreted as **mock-mode evaluation results**, not measurements of real-LLM performance.

This makes it useful for:

* CI
* reproducible experiments
* local development
* regression testing
* benchmark generation
* architecture testing

However, mock-mode results should not be interpreted as measurements of actual LLM security performance.


## Real LLM Mode

GuardEval also supports an OpenAI-backed semantic classifier.

### Configuration

Set the LLM mode:

```bash
export GUARDEVAL_LLM_MODE=real
```

Provide your OpenAI API key:

```bash
export OPENAI_API_KEY="your-api-key"
```

The configured model is:

```bash
export OPENAI_MODEL="gpt-5.6-luna"
```

### Run a Real-LLM Benchmark

```bash
GUARDEVAL_DATASET=dataset/test_realistic_95_5_400.jsonl \
GUARDEVAL_LLM_MODE=real \
python -m evaluation.evaluator
```

The real-LLM benchmark has not been reported in this repository because the available OpenAI API account currently has insufficient quota.

No real-LLM performance numbers are fabricated or substituted with mock results.

> **Important:** The benchmark results reported in this README are from `mock` mode unless explicitly stated otherwise. They should not be interpreted as measurements of actual LLM security performance.

## Evaluation Metrics

GuardEval reports the following metrics:

| Metric | Definition |
|---|---|
| **Accuracy** | Overall fraction of correctly classified requests. |
| **Precision** | Fraction of blocked requests that were actually attacks. |
| **Attack Detection Rate / Recall** | Fraction of attacks correctly blocked. |
| **F1 Score** | Harmonic mean of precision and recall. |
| **False Positive Rate (FPR)** | Fraction of benign requests incorrectly blocked. |
| **False Negative Rate (FNR)** | Fraction of attacks incorrectly allowed. |
| **Specificity** | Fraction of benign requests correctly allowed. |


## CI Quality Gate

The dashboard applies the following quality thresholds:

| Metric | Requirement |
|---|---:|
| **Minimum Attack Detection Rate** | **≥ 95%** |
| **Maximum False Positive Rate** | **≤ 5%** |

The quality gate passes only when **both conditions** are satisfied:

| Condition | Required |
|---|---:|
| Attack Detection Rate / Recall | ≥ 95% |
| False Positive Rate | ≤ 5% |

### Holdout Quality Gate

The unseen 200-sample holdout currently achieves:

| Metric | Holdout Result | Threshold | Status |
|---|---:|---:|---|
| Attack Detection Rate | 80.00% | ≥ 95% | ❌ Below threshold |
| False Positive Rate | 0.00% | ≤ 5% | ✅ Within threshold |

The holdout therefore does **not** satisfy the configured 95% attack-detection threshold.

This is intentional. The holdout is used to expose generalization limitations rather than to tune the classifier until the threshold passes.



## Streamlit Dashboard

GuardEval includes an interactive Streamlit dashboard for inspecting benchmark results.

### Dashboard Features

| Feature | Description |
|---|---|
| Benchmark Configuration | Dataset, sample counts, and LLM mode |
| Evaluation Metrics | Accuracy, precision, recall, F1, and error rates |
| Confusion Matrix | True/false positives and negatives |
| Category Performance | Detection performance by security category |
| Decision-Method Coverage | Rule vs. semantic classification coverage |
| CI Quality Gate | Detection and false-positive thresholds |

### Live Dashboard

[Open the GuardEval Dashboard](https://ishapageni-guardeval-dashboardapp-mbstvj.streamlit.app/)

### Run Locally

```bash
streamlit run dashboard/app.py
```

The dashboard will be available at:

```text
http://localhost:8501
```

---

## Evaluation

Run the standard development benchmark:

```bash
GUARDEVAL_DATASET=dataset/test_realistic_95_5_400.jsonl \
GUARDEVAL_LLM_MODE=mock \
python -m evaluation.evaluator
```

The evaluator writes the results to:

```text
evaluation_report.json
```

Run the unseen holdout:
```
GUARDEVAL_DATASET=dataset/test_holdout_200.jsonl \
GUARDEVAL_LLM_MODE=mock \
python -m evaluation.evaluator
```
The evaluator writes:
```
evaluation_report.json
```
## Project Structure
```
guardeval/
│
├── dashboard/
│   └── app.py
│
├── dataset/
│   ├── test_extended.jsonl
│   ├── test_legacy_regex_tuned.jsonl
│   ├── test_realistic_95_5.jsonl
│   ├── test_realistic_95_5_direct.jsonl
│   ├── test_realistic_95_5_adversarial.jsonl
│   ├── test_realistic_95_5_400.jsonl
│   └── test_holdout_200.jsonl
│
├── evaluation/
│   ├── evaluator.py
│   └── baselines.py
│
├── guardrails/
│   ├── input_guard.py
│   ├── policy.py
│   ├── hybrid_guard.py
│   ├── mock_llm_guard.py
│   └── llm_guard.py
│
├── generate_realistic_95_5.py
├── generate_adversarial_95_5.py
├── generate_realistic_95_5_400.py
├── generate_holdout_200.py
│
├── evaluation_report.json
├── requirements.txt
└── README.md
```
## Limitations

The current evaluation has several important limitations.

| Limitation | Description |
|---|---|
| **Controlled dataset** | The benchmark is constructed rather than sampled from production traffic. |
| **Reused benign prompts** | The 400-sample development benchmark contains controlled prompt variants, so the 380 benign samples should not be interpreted as 380 completely independent real-world observations. |
| **Deterministic mock classifier** | The current benchmark uses a local pattern-based semantic simulator rather than an actual LLM. |
| **Holdout generalization gap** | Attack detection decreased from 100% on the development benchmark to 80% on the unseen 200-sample holdout. |
| **Real LLM performance unmeasured** | The OpenAI-backed classifier is implemented, but a representative real-LLM benchmark has not been reported because the available API account has insufficient quota. |

### Holdout Generalization

The unseen 200-sample holdout achieved:

| Metric | Development Benchmark | Unseen Holdout |
|---|---:|---:|
| Accuracy | 100.00% | 98.00% |
| Attack Detection / Recall | 100.00% | 80.00% |
| Precision | 100.00% | 100.00% |
| F1 Score | 100.00% | 88.89% |
| False Positive Rate | 0.00% | 0.00% |
| False Negative Rate | 0.00% | 20.00% |

> **Holdout finding:** Attack detection decreased from **100%** on the controlled development benchmark to **80%** on the unseen holdout. This indicates that the deterministic semantic simulator is sensitive to differences in attack phrasing.

The holdout result is important because it demonstrates why performance on a controlled development dataset should not be treated as equivalent to real-world security performance.

### Real LLM Performance Is Unmeasured

The real OpenAI-backed semantic classifier is implemented, but representative real-LLM benchmark results are not reported because the available API account currently has insufficient quota.

No real-LLM performance numbers are fabricated or substituted with mock results.

### No Claim of Production Security

These experiments demonstrate the evaluation architecture and benchmark methodology.

They do **not** establish that GuardEval provides complete protection against real-world attacks or that the reported mock-mode results represent actual LLM security performance.

---

## Conclusion

GuardEval provides a reproducible framework for evaluating layered security controls for LLM-powered agents.

The controlled development benchmark demonstrates how deterministic rules and semantic classification can be evaluated together, while the unseen holdout provides a separate measurement of generalization.

The holdout achieved **80% attack detection with 0% false positives** across 200 previously unseen samples. This result exposes false negatives that were not visible in the controlled development benchmark.

The evaluation therefore highlights the importance of:

- **Held-out security datasets**
- **False-negative analysis**
- **Category-level evaluation**
- **Baseline comparison**
- **Reproducible CI benchmarks**
- **Separating mock evaluation from real-LLM evaluation**

Future work includes evaluating the framework with a real LLM, expanding the holdout dataset, adding more diverse attack categories, and testing additional semantic classifiers.

## Contributors

- Karuna Shah
- Ishapageni
