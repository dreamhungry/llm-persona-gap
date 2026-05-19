# Persona-Gap: Paper Draft Suggestions

This document collects **title** and **abstract** drafts for the Persona-Gap
research project. It is a working draft for discussion, not a finalized
submission.

> **TL;DR of the research idea**
> LLM agents are routinely conditioned on personality prompts (e.g.,
> "you are aggressive"), but it is unclear whether their *actions* in
> sequential decision-making actually reflect those *expressed* traits.
> Persona-Gap proposes a dual-channel measurement framework — extracting a
> 4-dimensional personality vector (risk, aggression, cooperation,
> deception) from text *and* from action trajectories — and quantifies
> the alignment, gap, and temporal drift between the two channels in
> strategic games such as Leduc Hold'em.

---

## 1. Title Suggestions

The titles are grouped by tone. Each option is paired with a one-line
"why this title" rationale so we can iterate quickly.

### A. Direct / question-style (recommended for venues like NeurIPS / ICLR / EMNLP main track)

1. **Do LLM Agents Act as They Say? Measuring the Persona Gap Between Expressed and Behavioral Personality**
   - Mirrors the project tagline; question form is memorable and frames a clear hypothesis.
2. **Saying vs. Doing: Quantifying the Behavioral–Expressed Personality Gap in LLM Agents**
   - "Saying vs. Doing" is concise, intuitive, and search-friendly.
3. **The Persona Gap: When LLM Agents Talk One Way and Play Another**
   - Slightly more narrative; emphasizes the gameplay setting.

### B. Mechanism-focused (recommended for ACL findings / AAAI / agent workshops)

4. **Persona-Gap: A Dual-Channel Framework for Auditing Personality Consistency in LLM Agents**
   - Highlights the *framework* contribution (text channel + action channel).
5. **Behavioral Auditing of Persona-Conditioned LLM Agents in Strategic Games**
   - "Behavioral auditing" positions the work as evaluation/safety research.
6. **From Prompt to Play: Trajectory-Grounded Evaluation of Personality in LLM Agents**
   - Connects prompt engineering to observable behavior.

### C. Findings-driven (use only if empirical results are strong)

7. **LLM Agents Speak Aggressively but Play Conservatively: A Study of Persona Drift in Imperfect-Information Games**
   - Punchy, but commits to a specific finding — only adopt after results are locked in.
8. **The Honesty Illusion: Detecting Deception Mismatch Between LLM Reasoning, Communication, and Action**
   - Emphasizes the deception dimension, which is the framework's most novel signal.

### Recommended primary title

> **Do LLM Agents Act as They Say? Measuring the Persona Gap Between Expressed and Behavioral Personality**

with a mechanism-flavored subtitle for camera-ready:

> **Persona-Gap: A Dual-Channel Framework for Behavioral Auditing of Persona-Conditioned LLM Agents**

---

## 2. Abstract Drafts

Three abstract variants targeting different lengths and audiences. All are
written assuming the experimental setup currently implemented in the
codebase: 4-dimensional personality vector p = (r, a, c, d) in [0,1]^4,
LLM agents playing Leduc Hold'em (and other RLCard / TextArena games)
across 50-100+ episodes, with three behavioral extractors
(annotation / state-aware scorer / LLM-as-judge) and two expressed
extractors (text-only / grounded).

### Variant 1 — Standard 200-word abstract (recommended default)

Large language model (LLM) agents are increasingly deployed with persona prompts that specify traits such as risk-taking, aggression, cooperation, or deception. Yet it remains unclear whether such *expressed* personalities actually shape the agent's *behavior* in sequential decision-making. We introduce **Persona-Gap**, a dual-channel framework that measures the gap between what an LLM agent says and what it does. Persona-Gap represents personality as a 4-dimensional vector and estimates it from two independent channels: an *expressed* channel obtained by LLM-as-judge analysis of private reasoning and public messages, and a *behavioral* channel obtained from action trajectories using state-aware scoring or LLM judging. We quantify alignment via L1 distance and cosine similarity, and track per-episode drift to characterize temporal consistency. Across imperfect-information games such as Leduc Hold'em, we find systematic persona gaps — in particular, deception expressed in language is often not realized in play, while behavioral aggression frequently exceeds what agents verbally claim. Persona-Gap provides a reproducible, game-agnostic toolkit for auditing personality fidelity in LLM agents and exposes failure modes hidden by purely text-based evaluation.

### Variant 2 — Short 120-word abstract (for workshops / extended abstracts)

Persona-conditioned LLM agents are common, but text-only evaluation cannot tell whether an agent prompted to be "aggressive" or "cooperative" actually behaves that way. We propose **Persona-Gap**, a framework that measures personality along four dimensions (risk, aggression, cooperation, deception) from two channels: an *expressed* channel based on agent reasoning and messages, and a *behavioral* channel derived from action trajectories in strategic games. We compute alignment, per-dimension gaps, and temporal drift across 50+ episodes of Leduc Hold'em and related games. Our results reveal consistent mismatches between stated and enacted personality, most notably in the deception dimension. Persona-Gap offers a reproducible, game-agnostic toolkit for auditing the behavioral fidelity of persona prompts.

### Variant 3 — Long 280-word abstract (for journal / extended track)

Large language model (LLM) agents are routinely steered by persona prompts that assign traits such as *risk-taking*, *aggression*, *cooperation*, or *deception*. Existing evaluations, however, mostly probe what agents *say* — through self-reports or text classification — and rarely test whether these traits are reflected in *what they do* when embedded in sequential, strategic environments. We introduce **Persona-Gap**, a dual-channel measurement framework for auditing personality fidelity in LLM agents. Persona-Gap encodes personality as a 4-dimensional vector p = (r, a, c, d) in [0,1]^4 and recovers it from two independent signals: an **expressed** channel that applies LLM-as-judge analysis to private reasoning and public communication, and a **behavioral** channel that scores action trajectories using either rule-based state-aware scoring (e.g., bluff and trap detection in poker) or LLM-based judging. We then compute alignment via L1 distance, cosine similarity, and per-dimension gaps, and analyze behavioral consistency and drift across 50-100 episodes. The framework is implemented as a registry-based, game-agnostic system supporting RLCard (Leduc Hold'em, UNO, Blackjack), TextArena, and custom environments, with checkpointing, batch sweeps over personality presets, and visualization tooling. Empirically, in imperfect-information games we observe systematic gaps between expressed and behavioral personality: agents readily talk about deception but rarely enact context-appropriate bluffs, while aggression in play often exceeds what is verbally claimed. We further show that behavioral personality drifts non-trivially over time, even under fixed prompts. Persona-Gap exposes failure modes invisible to text-only evaluation and provides a reproducible toolkit for behavioral auditing of persona-conditioned LLM agents.

---

## 3. Keyword Suggestions

For submission metadata / arXiv categories:

- LLM agents, persona prompting, behavioral evaluation
- alignment auditing, agent benchmarking
- imperfect-information games, Leduc Hold'em
- LLM-as-judge, deception detection
- temporal consistency, personality drift

---

## 4. Open Questions for the Authors

These should be resolved before locking the title and abstract:

1. **Primary venue?** A safety / evaluation venue (e.g., NeurIPS D&B, ICLR
   blog post, ACL Findings) favors Title A1 + Abstract Variant 1.
   A general agent venue (AAMAS, CoLM) may favor Title B4 + Variant 3.
2. **How many environments are reported in the paper?** If only Leduc
   Hold'em is evaluated end-to-end, the abstract should say
   "in Leduc Hold'em" rather than "in strategic games" to avoid
   over-claiming.
3. **Is there a quantitative headline number?** (e.g., "average L1 gap
   of 0.X across N personas".) If yes, splice it into Variant 1 to
   strengthen the abstract.
4. **Is the deception finding statistically robust** across personas and
   models? If yes, promote Title C8 from "optional" to a strong
   alternative.
5. **Are human ratings used to validate LLM-as-judge?** If yes, mention
   it in the abstract — it materially strengthens the contribution.

---

## 5. Next Steps

- [ ] Pick one primary title from section 1 and one abstract variant from section 2.
- [ ] Fill in the headline empirical numbers once experiments converge.
- [ ] Draft an introduction outline aligned with the chosen abstract.
- [ ] Add a "Limitations" paragraph (single environment family, judge-LLM
      bias, prompt sensitivity).
