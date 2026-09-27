---
dataset_info:
  features:
  - name: run_id
    dtype: string
  - name: scenario_id
    dtype: string
  - name: system_name
    dtype: string
  - name: transcript
    sequence: string
  - name: metrics
    dtype: dict
  - name: judge_scores
    dtype: dict
  splits:
  - name: train
    num_examples: 6
  config_name: default
  config_version: '0.0.1'
license: mit
language:
- en
task_categories:
- conversational
tags:
- voice-ai
- phone-agents
- benchmark
- evaluation
- automatic-speech-recognition
- text-to-speech
- natural-language-processing
size_categories:
- 1K<n<10K
---

# Voice Agent Bench

A reproducible, blind-evaluated benchmark for AI voice agents handling real-world phone tasks.

**What this dataset contains:**
- Full call transcripts (turn-by-turn) from head-to-head voice-agent evaluations
- Objective metrics (latency, turn count, duration) derived from Twilio recordings
- Neutral LLM-judge scores on conversation quality criteria
- Test scenarios, AI-shopper prompts, and judge rubric — so anyone can reproduce or extend

**Current evaluation:** Calldesk (flow-builder, cascaded pipeline) vs ThunderPhone (single-prompt, Spark tier) on medical-appointment booking.

---

## Why this exists

Existing audio benchmarks (e.g. Big Bench Audio) test reasoning on speech input — not whether a voice agent can reliably book an appointment, collect a phone number, or recover from a mis-hear. This dataset fills that gap with real phone-call transcripts, objective latency measurements, and a published scoring rubric.

---

## Structure

```
data/
  scenarios.jsonl    — test scenarios (persona, goal, success criteria)
  transcripts.jsonl  — full turn-by-turn transcripts from each system
  metrics.jsonl      — objective measurements + judge scores per round
rubric.md            — detailed scoring criteria used by the blind judge
REPRODUCE.md         — exact commands to re-run the benchmark yourself
```

---

## Latest results (2026-09-26)

| Round | Winner   | Calldesk turns | ThunderPhone turns | Key differentiator                                  |
|-------|----------|---------------:|-------------------:|------------------------------------------------------|
| 1     | Calldesk | 6              | 12                 | Name/number captured correctly; TP had "Alec" error  |
| 2     | Calldesk | 3              | 10                 | Compound slot-fill; TP asked name before knowing why |
| 3     | Calldesk | 3              | 14                 | 55s vs 162s; TP failed twice on "I'm flexible"       |

**Aggregate: Calldesk 3 – 0 ThunderPhone**

See `data/metrics.jsonl` for full per-criterion scores and `data/transcripts.jsonl` for raw dialogue.

---

## Contributing a system

To add your voice-agent platform to this benchmark:

1. Provide an inbound phone number with your agent assigned
2. Open a PR adding your system to `systems/` with config metadata
3. We run the same 3-round mystery-shopper protocol against it
4. Judge scores are appended to `data/metrics.jsonl`; transcripts to `data/transcripts.jsonl`

All evaluations are blind: the judge does not know which system is which until after scoring.

---

## Citation

```bibtex
@dataset{calldesk2026voiceagentbench,
  title = {Voice Agent Bench: Reproducible Evaluation of AI Phone Agents},
  author = {Calldesk},
  year = {2026},
  url = {https://huggingface.co/datasets/calldesk/voice-agent-bench}
}
```
