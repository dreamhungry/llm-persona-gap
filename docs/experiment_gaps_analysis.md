# Experiment Design Gap Analysis

This document identifies gaps, weaknesses, and improvement opportunities
in the current Persona-Gap experimental design, with a focus on what is
needed to produce a publishable paper.

---

## Executive Summary

The framework design is solid (clean architecture, extensible patterns,
three behavioral extraction methods). However, the **empirical evidence**
is far from paper-ready. The main blockers are:

1. Only one game environment fully implemented (Leduc Hold'em)
2. Experiment scale too small (5-104 episodes, no repetitions)
3. No statistical rigor (no confidence intervals, significance tests)
4. No human validation of LLM-as-judge
5. Expressed personality extraction unstable (some runs yield all-zero)

---

## 1. Critical Issues (Must Fix Before Submission)

### 1.1 Single Game Environment

**Problem**: Only Leduc Hold'em has full support (observation formatter,
action annotations, StateParser, ScoreCalculator, PromptBuilder). The
README claims support for UNO, Blackjack, TextArena, but code only has
stub-level support (generic observation formatter, no scorer/judge).

**Impact**: Reviewers will question generalizability. A "game-agnostic
framework" paper with one game is unconvincing.

**Recommendation**: Implement at least 2 additional environments:
- One **cooperative** game (e.g., Hanabi via TextArena, or a social
  deduction game) to stress-test cooperation/deception dimensions
- One **different information structure** (e.g., Blackjack for
  single-agent risk, or UNO for multi-player aggression)

Effort estimate: ~2-3 days per game (StateParser + ScoreCalculator +
PromptBuilder + action annotations + observation formatter).

### 1.2 Insufficient Sample Size

**Problem**: Published experiments have 5 episodes (test1, test2) or 104
episodes (default, but using the weakest annotation method). For a
behavioral study, this is far below what peer reviewers expect.

**Recommendation**:
- Minimum 50 episodes per (persona × model × game) condition
- At least 3 independent runs per condition for variance estimation
- Total experiment matrix example:
  ```
  4 personas × 2 models × 3 games × 50 episodes × 3 runs = 3,600 episodes
  ```

### 1.3 No Statistical Rigor

**Problem**: Results are reported as single point estimates. No confidence
intervals, no significance tests, no effect sizes.

**Recommendation**:
- Report mean ± std across runs
- Use bootstrap confidence intervals for L1/cosine metrics
- Paired t-test or Wilcoxon signed-rank for comparing methods
- Cohen's d for effect size of persona gap
- Add a `stats.py` module to the analysis pipeline

### 1.4 Expressed Personality Extraction Instability

**Problem**: In test1, agent_0's expressed personality is all zeros. In
the default run, expressed values are all 0.5 (placeholder). This
suggests the LLM-as-judge pipeline has reliability issues.

**Root Cause Hypotheses**:
- Judge LLM call failures not properly handled (silent fallback to 0.0)
- JSON parsing errors in judge response
- Insufficient reasoning text for grounded evaluation

**Recommendation**:
- Add retry + fallback logic for judge failures
- Log every judge call and response (TracingLLMBackend exists but may
  not be used consistently)
- Validate expressed extraction with synthetic test cases (known
  aggressive reasoning → should yield high aggression score)
- Report judge call success rate as a data quality metric

### 1.5 No Human Validation

**Problem**: LLM-as-judge is the primary measurement instrument for
expressed personality, but its accuracy is never validated against human
raters.

**Recommendation**:
- Sample 50-100 (reasoning, message, action) tuples
- Have 2-3 human annotators rate each on 4 dimensions
- Compute inter-annotator agreement (Krippendorff's alpha)
- Compute LLM-judge vs human correlation (Pearson/Spearman per dimension)
- Report as Table 1 in the paper to establish method validity

---

## 2. Major Issues (Strongly Recommended)

### 2.1 No Cross-Model Comparison

**Problem**: test1 uses one model, test2 uses another, but there is no
systematic comparison. Different LLMs may have fundamentally different
persona fidelity.

**Recommendation**:
- Test at least 3 models (e.g., GPT-4o, Claude 3.5, DeepSeek-V3)
- Same persona × same game → compare gap across models
- This becomes a key contribution: "which LLMs are most persona-faithful?"

### 2.2 No Ablation Studies

**Problem**: BatchExperimentConfig supports sweeps over memory and
communication, but no results exist. The reader cannot tell which
components matter.

**Recommendation**: Run ablations for:
- **Memory**: with vs without episode memory
- **Communication**: with vs without inter-agent messaging
- **Expressed context mode**: text-only vs grounded
- **Temperature**: 0.3 vs 0.7 vs 1.0
- Report delta in persona gap for each ablation

### 2.3 Cooperation Dimension Semantics in Zero-Sum Games

**Problem**: In Leduc Hold'em (zero-sum), "cooperation" is semantically
awkward. The current scorer maps `call → cooperative`, but calling can
also be strategic (pot odds justify it). This conflates cooperation with
passive play.

**Recommendation**:
- Option A: Drop cooperation from Leduc analysis, only report it in
  cooperative game environments
- Option B: Redefine cooperation in zero-sum context as "non-exploitative
  play" and justify in the paper
- Option C: Add a truly cooperative environment where cooperation has
  clear behavioral signals

### 2.4 No Baseline Comparisons

**Problem**: The paper proposes a new measurement framework but does not
compare it against any existing approach.

**Recommendation**:
- **Self-report baseline**: Ask the agent directly "rate your risk-taking
  on a scale of 0-1" → compare with behavioral measurement
- **BFI-style questionnaire**: Administer Big Five Inventory questions
  and map to the 4 dimensions → compare alignment
- **Random baseline**: What is the expected gap if behavioral personality
  is random? (establishes a lower bound for the metric)

### 2.5 Judge Bias Quantification

**Problem**: LLM judges may have systematic biases (e.g., over-reporting
deception, under-reporting risk). This is not measured.

**Recommendation**:
- **Self-agreement test**: Run the same trajectory through the judge
  3 times → report consistency (should be >0.9 correlation)
- **Calibration test**: Generate synthetic trajectories with known ground
  truth personality → measure judge accuracy
- **Cross-judge test**: Use 2 different judge models → report agreement

---

## 3. Minor Issues (Nice to Have)

### 3.1 Interaction Effects Not Analyzed

The opponent's personality may influence an agent's behavioral
expression. E.g., a "cooperative" agent facing an "aggressive" opponent
may become more aggressive.

**Recommendation**: Report personality gap as a function of opponent type.
This is a 4×4 matrix (self-persona × opponent-persona) that would make
a compelling figure.

### 3.2 Prompt Sensitivity Not Tested

The `personality_to_text()` function uses fixed thresholds (0.3/0.7) and
template phrasing. Different wordings might yield different gaps.

**Recommendation**: Test 2-3 prompt templates for the same personality
vector and report variance in behavioral output.

### 3.3 Temporal Analysis Under-Utilized

The temporal drift machinery exists but is not prominently featured in
outputs. Drift patterns could be a standalone finding.

**Recommendation**:
- Plot drift curves for all personas across 50+ episodes
- Identify if certain dimensions drift more than others
- Test whether drift correlates with memory length or game outcomes

### 3.4 Communication Channel Analysis

Messages between agents are captured but not analyzed independently.
There could be a "communication persona" distinct from both expressed
(private reasoning) and behavioral (actions).

**Recommendation**: Extract personality from public messages only →
compare with private reasoning and actions → potentially reveal a
"social face" vs "inner personality" gap.

### 3.5 Batch Sweep Over agent_1 Missing

`BatchExperimentConfig.sweep_personalities` only iterates over agent_0
presets. Agent_1 is fixed. This means we never test how persona gap
changes when both agents have varied personalities.

**Recommendation**: Extend batch sweep to cover all agent combinations.

---

## 4. Code-Level Issues Affecting Reproducibility

| Issue | File | Impact |
|-------|------|--------|
| Exception handling commented out in `act()` | `llm_agent.py:107-115` | Silent failures may corrupt data |
| Temporal consistency normalization assumes max variance = ndim | `temporal.py` | Scores can exceed [0,1] in edge cases |
| No seed control for LLM sampling | `backend.py` | Results not reproducible across runs |
| `annotation` method maps fold/check to no tags | `action_annotations.toml` | Under-counts behavioral signals |
| No data quality checks in analysis pipeline | `run_analysis.py` | Bad episodes (all-fallback) included in stats |

---

## 5. Suggested Paper Experiment Plan

### Phase 1: Fix Foundations (1-2 weeks)
- [ ] Fix expressed extraction stability (debug zero-output issue)
- [ ] Add statistical analysis module (CI, significance tests)
- [ ] Implement at least 1 additional game environment
- [ ] Add seed control and run determinism checks

### Phase 2: Core Experiments (2-3 weeks)
- [ ] Run full matrix: 4 personas × 2+ models × 2+ games × 50 eps × 3 runs
- [ ] Compute all metrics with confidence intervals
- [ ] Generate headline numbers (average gap, per-dimension findings)

### Phase 3: Validation & Ablations (1-2 weeks)
- [ ] Human annotation study (50-100 samples)
- [ ] Judge self-consistency and calibration tests
- [ ] Ablation: memory, communication, temperature, prompt template
- [ ] Cross-model comparison

### Phase 4: Analysis & Writing (1-2 weeks)
- [ ] Interaction effect analysis (self × opponent)
- [ ] Temporal drift deep-dive
- [ ] Generate all figures (radar, heatmap, drift, comparison)
- [ ] Write results section with statistical reporting

---

## 6. Estimated Effort to Paper-Ready

| Category | Minimum (workshop) | Full (main track) |
|----------|-------------------|-------------------|
| Additional environments | 1 game (~3 days) | 2+ games (~1 week) |
| Experiment runs | 400 episodes (~2 days compute) | 3,600+ episodes (~1 week) |
| Human validation | Skip for workshop | 50-100 annotations (~1 week) |
| Statistical analysis | Basic CI (~1 day) | Full suite (~3 days) |
| Ablations | 1 ablation (~2 days) | 4+ ablations (~1 week) |
| Cross-model | 2 models (~2 days) | 3+ models (~4 days) |
| **Total** | **~2 weeks** | **~5-6 weeks** |

---

## 7. Strengths to Emphasize in the Paper

Despite the gaps above, the project has clear strengths:

1. **Novel framing**: "expressed vs behavioral" dual-channel measurement
   is a genuinely new evaluation paradigm for LLM agents
2. **Three-method hierarchy**: annotation → scorer → LLM-judge provides
   a speed/accuracy tradeoff that others can adopt
3. **Deception detection**: The scorer's bluff/trap detection is a unique
   technical contribution (not present in prior work)
4. **Temporal analysis**: Drift measurement over episodes is rarely
   studied in LLM agent evaluation
5. **Reproducible toolkit**: Clean architecture means others can extend
   to new games easily
6. **Actionable findings**: Gap patterns (high verbal deception / low
   behavioral deception) have practical implications for AI safety

---

## 8. Priority Ranking

If time is limited, focus on these in order:

1. **Fix expressed extraction** → without this, half the framework is broken
2. **Scale up Leduc experiments** → 50+ episodes, 3 runs, 2 models
3. **Add one more game** → breaks the "single game" criticism
4. **Add statistical tests** → transforms "observations" into "findings"
5. **Human validation** → establishes method credibility
6. **Ablations** → deepens understanding of what drives the gap

Everything else (interaction effects, communication analysis, prompt
sensitivity) is bonus material that strengthens but is not required.
