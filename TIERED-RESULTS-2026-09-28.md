# Tiered Benchmark: Calldesk vs ThunderPhone Spark vs Bolt

> **Date:** 2026-09-28  
> **Scenario:** Medical appointment booking — collect name (spelled), preferred time, callback number, confirm, close  
> **Rounds:** 5 per tier (Spark completed, Bolt 3 of 5 clean — 1 empty transcript, 1 API crash)  
> **Method:** Same AI shopper calls both backends simultaneously; blind LLM judge scores  
> **Latency:** Waveform-to-waveform from Twilio recordings  
> **Judge:** Claude Sonnet 4-6 via Anthropic API  
> **Caldesk config:** greeting→booking→confirmation→goodbye flow, brief shopper prompt, deterministic phone guard

---

## Results Summary

| Tier | Calldesk Wins | ThunderPhone Wins | Notes |
|------|--------------|-------------------|-------|
| **Spark** ($0.02/min) | 4 / 5 | 1 / 5 | Calldesk won 4 of 5 after flow fixes |
| **Bolt** ($0.05/min) | 1 / 3 | 2 / 3 | Bolt more competitive than prior run; 2 rounds excluded due to infra issues |

**Overall (8 clean rounds):** Calldesk 5 – ThunderPhone 3

---

## Spark Tier — Round-by-Round

| Round | Winner | CD Turns | TP Turns | CD P50/P95 (ms) | TP P50/P95 (ms) | CD Dur | TP Dur |
|-------|--------|----------|----------|-----------------|-----------------|--------|--------|
| 1 | ThunderPhone | 21 | 12 | 2560/4360 | 2200/6500 | 128 s | 90 s |
| 2 | Calldesk | 11 | 14 | 2560/7960 | 2440/7040 | 65 s | 109 s |
| 3 | Calldesk | 15 | 16 | 2160/7100 | 2080/3760 | 80 s | 166 s |
| 4 | Calldesk | 12 | 12 | 2020/2280 | 2180/6100 | 66 s | 98 s |
| 5 | Calldesk | 14 | 15 | 2860/9080 | 2120/5540 | 77 s | 124 s |

**Spark aggregate:**
- Calldesk median turns: 12 | ThunderPhone: 14
- Calldesk median duration: 77 s | ThunderPhone: 109 s
- Calldesk median p50 latency: 2560 ms | ThunderPhone: 2200 ms
- Calldesk max latency: 9080 ms | ThunderPhone: 7040 ms

---

## Bolt Tier — Round-by-Round

| Round | Winner | CD Turns | TP Turns | CD P50/P95 (ms) | TP P50/P95 (ms) | CD Dur | TP Dur |
|-------|--------|----------|----------|-----------------|-----------------|--------|--------|
| 1 | Calldesk | 11 | 13 | 2320/7840 | 4380/10560 | 63 s | 137 s |
| 2 | ThunderPhone | 9 | 10 | 2640/8180 | 2140/2420 | 45 s | 112 s |
| 3 | ThunderPhone | 18 | 11 | 1960/8140 | 2180/3960 | 99 s | 154 s |
| 4 | — | — | — | — | — | — | — | Empty transcript |
| 5 | — | — | — | — | — | — | — | Script crash |

**Bolt aggregate (3 clean rounds):**
- Calldesk median turns: 11 | ThunderPhone: 11
- Calldesk median duration: 63 s | ThunderPhone: 137 s
- Calldesk median p50 latency: 2320 ms | ThunderPhone: 2180 ms
- Calldesk max latency: 8180 ms | ThunderPhone: 10560 ms

---

## What Changed Since Prior Run (2026-09-27)

### Calldesk fixes applied
1. **Shopper prompt rewritten** — strict 1-2 sentence limit, 15-word max, explicit negative examples. Eliminated verbose cooperative answers.
2. **Deterministic phone guard** — regex intercepts any non-canonical phone number in shopper text before TTS, replaces with "five five five zero one four seven".
3. **Confirmation node** — single-ask confirmation gate between booking and goodbye. Reads back once, accepts yes, immediately transitions. No loops.
4. **Booking prompt tightened** — extracts all three fields (name, time, phone) from a single caller turn without re-asking.
5. **Goodbye prompt locked** — no follow-up questions after confirmation. Clean close.
6. **Server-side:** tag stripping, soft phone validation, transition validation.

### Result impact
- Spark swung from 2-3 (TP leading) to 4-1 (Caldesk leading)
- Bolt went from 4-0 (Caldesk sweep) to 1-2 (more competitive)

---

## What the Data Shows (no interpretation)

### Observation 1: Spark and Bolt are closer than pricing suggests
Spark ($0.02/min) and Bolt ($0.05/min) produced similar conversation quality against the same task. Bolt did not demonstrate clear superiority over Spark in this scenario.

### Observation 2: Calldesk duration is consistently shorter
Across all clean rounds, Calldesk median call duration was 63–77 s vs. ThunderPhone 109–137 s. This is partly because Calldesk's shopper is now brief, but also reflects Calldesk flow efficiency.

### Observation 3: Latency outliers affect both systems
- Calldesk max: 9080 ms (Spark round 5)
- ThunderPhone max: 10560 ms (Bolt round 1)
Both systems produce 8–10 second pauses occasionally.

### Observation 4: Bolt infrastructure less stable
2 of 5 Bolt rounds failed (empty transcript, script crash). Spark completed all 5. This could indicate Bolt tier has higher operational variance.

---

## Limitations

1. **Small sample:** 5 rounds per tier is marginal. 10–20 rounds needed for confidence.
2. **Single scenario:** Only medical appointment booking. Generalization unknown.
3. **English only:** Both in en-US. ThunderPhone advertises 47 languages — not tested.
4. **Storm tier unavailable:** Cannot create via API. May require enterprise access.
5. **2 Bolt rounds excluded:** Empty transcript and script crash. Infrastructure instability.
6. **Judge variance:** Same judge criteria, but LLM judgment has inherent subjectivity.

---

## Configuration Details (Reproducibility)

### Calldesk flow (6 nodes)
```
greeting → booking → confirmation → goodbye
take_message → goodbye
transfer (no edges)
```

**Booking node:** Collects name (spelled), preferred time, callback number. Extracts all fields from single turn. Calls `transition_flow` when all collected.

**Confirmation node:** Reads back once. Asks "Does that look right?" ONE time. Accepts yes → goodbye. No loops, no follow-ups.

**Goodbye node:** Warm farewell. No questions.

### Shopper configuration
- Model: claude-3-5-haiku-20241022
- Prompt: Strict 1-2 sentence limit, 15-word max, no cooperativeness, canonical phone number
- Phone guard: regex intercepts non-canonical digits before TTS

### ThunderPhone agents
- Spark: `id=253`, voice=john, product=spark, medical booking prompt
- Bolt: `id=254`, voice=john, product=bolt, same prompt

---

## Cost of This Run

| Tier | Rounds | Avg Duration | Rate | Estimated Cost |
|------|--------|-------------|------|----------------|
| Spark | 5 | ~83 s | $0.02/min | ~$0.14 |
| Bolt | 3 (+2 failed) | ~118 s | $0.05/min | ~$0.30 |
| Calldesk | 8 | ~80 s | $0.10/min | ~$1.07 |
| **Total** | | | | **~$1.51** |

---

## Files

- `data/transcripts.jsonl` — all turn-by-turn transcripts (appended)
- `data/metrics.jsonl` — latency measurements (appended)
- `flow-config-medical-booking.json` — Calldesk flow definition
