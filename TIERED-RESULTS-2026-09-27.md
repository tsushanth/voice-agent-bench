# Tiered Benchmark: Calldesk vs ThunderPhone Spark vs Bolt

> **Date:** 2026-09-27  
> **Scenario:** Medical appointment booking — collect name (spelled), preferred time, callback number, confirm, close  
> **Rounds:** 5 per tier (Spark completed, Bolt 4 of 5 due to hung call)  
> **Method:** Same AI shopper calls both backends simultaneously; blind LLM judge scores  
> **Latency:** Waveform-to-waveform from Twilio recordings  
> **Judge:** Claude Sonnet 4-6 via Anthropic API  

---

## Results Summary

| Tier | Calldesk Wins | ThunderPhone Wins | Notes |
|------|--------------|-------------------|-------|
| **Spark** ($0.02/min) | 2 / 5 | 3 / 5 | TP improved significantly with medical prompt + name spelling |
| **Bolt** ($0.05/min) | 4 / 4 | 0 / 4 | TP Bolt underperformed Spark on this task; hung call on round 5 |

**Overall (9 completed rounds):** Calldesk 6 – ThunderPhone 3

---

## Spark Tier — Round-by-Round

| Round | Winner | CD Turns | TP Turns | CD P50/P95 (ms) | TP P50/P95 (ms) | CD Dur | TP Dur |
|-------|--------|----------|----------|-----------------|-----------------|--------|--------|
| 1 | ThunderPhone | 7 | 10 | 2120/4680 | 2260/2440 | 88 s | 130 s |
| 2 | ThunderPhone | 12 | 10 | 2820/4760 | 2520/7180 | 153 s | 117 s |
| 3 | Calldesk | 5 | 12 | 3500/4780 | 2700/9280 | 61 s | 149 s |
| 4 | ThunderPhone | 10 | 8 | 2780/7480 | 2400/7000 | 112 s | 107 s |
| 5 | Calldesk | 8 | 11 | 2580/5180 | 2440/5800 | 104 s | 127 s |

**Spark aggregate:**
- Calldesk median turns: 8.4 | ThunderPhone: 10.2
- Calldesk median duration: 103.6 s | ThunderPhone: 126.0 s
- Calldesk median p50 latency: 2580 ms | ThunderPhone: 2440 ms
- Calldesk max latency: 7480 ms | ThunderPhone: 9280 ms

---

## Bolt Tier — Round-by-Round

| Round | Winner | CD Turns | TP Turns | CD P50/P95 (ms) | TP P50/P95 (ms) | CD Dur | TP Dur |
|-------|--------|----------|----------|-----------------|-----------------|--------|--------|
| 1 | Calldesk | 7 | 7 | 2420/2480 | 2540/5260 | 95 s | 147 s |
| 2 | Calldesk | 6 | 5 | 2380/3000 | 2680/2740 | 77 s | 110 s |
| 3 | Calldesk | 4 | 6 | 2500/2860 | 3200/4860 | 69 s | 102 s |
| 4 | Calldesk | 6 | 9 | 2880/4400 | 2100/4260 | 71 s | 103 s |
| 5 | — | — | — | — | — | — | — |

**Bolt aggregate (4 rounds):**
- Calldesk median turns: 6.0 | ThunderPhone: 6.8
- Calldesk median duration: 78.0 s | ThunderPhone: 115.5 s
- Calldesk median p50 latency: 2440 ms | ThunderPhone: 2610 ms
- Calldesk max latency: 4400 ms | ThunderPhone: 5260 ms

---

## What the Data Shows (no interpretation)

### Observation 1: Spark outperformed Bolt on this task
ThunderPhone Spark won 3 of 5 rounds; Bolt won 0 of 4. This is unexpected if higher tier = better performance. Possible explanations (not verified):
- Bolt's additional processing (thinking model, ensemble) added latency without benefit for this specific structured task
- Bolt may be optimized for different scenarios (complex reasoning, long prompts, multilingual)
- Sample size too small to draw firm conclusion

### Observation 2: Name-spelling requirement changed outcomes
In the earlier 3-round benchmark without name spelling, Calldesk swept 3-0 against Spark. With name spelling added:
- Spark improved to 3-2 (won 3 rounds)
- Bolt went 0-4
The prompt change helped ThunderPhone's simple prompt-based agent more than Calldesk's flow builder.

### Observation 3: Calldesk's efficiency advantage is inconsistent
- Best Calldesk round: 4 turns, 61 seconds (Bolt round 3)
- Worst Calldesk round: 12 turns, 153 seconds (Spark round 2)
- Turn count variance suggests flow instability — same flow, same shopper, different outcomes

### Observation 4: Latency outliers are a problem for both
- Calldesk max latency: 7480 ms (Spark round 4)
- ThunderPhone max latency: 9280 ms (Spark round 3)
- Both systems occasionally produce 5–9 second pauses that would feel like dropped calls

### Observation 5: One call hung indefinitely
Bolt round 5 TP call remained in-progress for 30+ minutes until manually terminated. Duration was 77 seconds but the call never signaled completion to Twilio. This is a reliability issue.

---

## Limitations

1. **Small sample:** 5 rounds per tier is marginal for statistical significance. 10–20 rounds would be needed for confidence.
2. **Single scenario:** Only medical appointment booking tested. Results may not generalize to outbound sales, support, or complex multi-intent calls.
3. **English only:** Both systems in en-US. TP advertises 47 languages — not tested.
4. **Storm tier unavailable:** Could not create Storm agent via API (`"storm" is not a valid choice`). May require enterprise access or different API key.
5. **Hung call:** Bolt round 5 excluded due to call stuck in-progress. Unknown if this is a Bolt-specific issue or transient infrastructure problem.
6. **No per-turn model visibility:** Cannot verify which TP models ran on which turns. All inferences about Spark vs Bolt internals are speculative.

---

## Calldesk Issues Observed (internal)

1. **Flow truncation:** Spark round 3 ended mid-sentence: *"Let me confirm that all looks good?"* with no customer response or close. Transcript cuts off.
2. **High turn-count variance:** Same flow produced 4 turns in one round, 12 in another. Suggests non-deterministic LLM behavior or race conditions in flow transitions.
3. **Latency spikes:** 7480 ms max on Spark round 4. Need to identify which turn triggers this.
4. **Phone number read-back issues:** In Bolt round 2, Calldesk captured "455-5014-7" (8 digits) without flagging the error.

---

## ThunderPhone Issues Observed (for reference)

1. **Bolt text-list confirmation:** Bolt round 1 closed with a robotic structured list read aloud: *"Name: A-L-E-X M-O-R-G-A-N / Appointment time: Tomorrow at 2:30 PM / Callback number: 5-5-5-0-1-4-7"* — clearly a formatted block rendered as TTS.
2. **Bolt re-asking provided info:** Asked for phone number in opening, then asked again in next turn.
3. **Bolt hung call:** Round 5 stuck in-progress for 30+ minutes.
4. **Spark latency spikes:** 9280 ms max on round 3.

---

## Next Steps to Make This Solid

1. **Run 10 more rounds per tier** (Spark + Bolt) for statistical confidence
2. **Fix Calldesk flow truncation** — add explicit end-of-call transition after confirmation
3. **Add retry/timeout handling** for hung calls in benchmark harness
4. **Test Storm tier** if/when API access becomes available
5. **Add second scenario** (e.g. rescheduling, FAQ, transfer) to test generalization
6. **Measure per-turn latency breakdown** in Calldesk to identify the 7+ second spikes

---

## Cost of This Run

| Tier | Rounds | Avg Duration | Rate | Estimated Cost |
|------|--------|-------------|------|----------------|
| Spark | 5 | ~118 s | $0.02/min | ~$0.20 |
| Bolt | 4 (+1 hung) | ~116 s | $0.05/min | ~$0.39 |
| Calldesk | 9 | ~95 s | $0.10/min | ~$1.43 |
| **Total** | | | | **~$2.02** |

TP test budget remaining: ~$7 (from $10 starting, ~$1.03 prior usage + ~$0.59 this run).
