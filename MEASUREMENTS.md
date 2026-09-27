# Voice Agent Bench: Raw Measurement Log

> **Date:** 2026-09-26  
> **Scenario:** Book a medical appointment (name + time + phone + confirm + close)  
> **Method:** Same AI shopper calls each system 3 times, neutral LLM judge scores blind  
> **Latency source:** Twilio stereo recordings, waveform-to-waveform measurement  
> **Judge model:** Claude Sonnet 4-6 via Anthropic API  
> **Data:** [github.com/tsushanth/voice-agent-bench](https://github.com/tsushanth/voice-agent-bench)

---

## 1. Measured Latency (milliseconds)

All values from Twilio recording analysis (`analyze-call-ttfb.py`). Time from end of caller speech to start of agent response.

### Round 1

| Percentile | Calldesk (6 turns measured) | ThunderPhone (10 turns measured) |
|------------|----------------------------|----------------------------------|
| Min | 1,820 | 1,300 |
| Median | 2,540 | 2,620 |
| Mean | 3,017 | 3,360 |
| P90 | 4,220 | 5,120 |
| P95 | 4,220 | 5,120 |
| Max | 4,560 | 6,500 |

### Round 2

| Percentile | Calldesk (3 turns) | ThunderPhone (10 turns) |
|------------|-------------------|------------------------|
| Min | 2,200 | 1,740 |
| Median | 2,260 | 2,340 |
| Mean | 2,340 | 3,042 |
| P90 | 2,260 | 5,240 |
| P95 | 2,260 | 5,240 |
| Max | 2,560 | 5,360 |

### Round 3

| Percentile | Calldesk (3 turns) | ThunderPhone (13 turns) |
|------------|-------------------|------------------------|
| Min | 1,540 | 1,920 |
| Median | 2,040 | 2,520 |
| Mean | 2,033 | 2,978 |
| P90 | 2,040 | 3,980 |
| P95 | 2,040 | 4,940 |
| Max | 2,520 | 5,160 |

### Aggregate Latency Summary

| Statistic | Calldesk (12 turns) | ThunderPhone (33 turns) |
|-----------|---------------------|-------------------------|
| Overall median | 2,260 ms | 2,520 ms |
| Overall max | 4,560 ms | 6,500 ms |
| Turns ≥ 4,000 ms | 1 / 12 (8%) | 8 / 33 (24%) |

*Note: ThunderPhone had more measurable turns because the conversation lasted longer (more agent utterances).*

---

## 2. Conversation Duration & Turn Count

| Round | System | Duration | Agent Turns | Shopper Turns |
|-------|--------|----------|-------------|---------------|
| 1 | Calldesk | 87 s | 6 | 18 |
| 1 | ThunderPhone | 90 s | 12 | 26 |
| 2 | Calldesk | 62 s | 3 | 14 |
| 2 | ThunderPhone | 114 s | 10 | 30 |
| 3 | Calldesk | 55 s | 3 | 12 |
| 3 | ThunderPhone | 162 s | 14 | 30 |

**Aggregate:**

| System | Avg Duration | Avg Agent Turns | Avg Shopper Turns |
|--------|-------------|-----------------|-------------------|
| Calldesk | 68 s | 4.0 | 14.7 |
| ThunderPhone | 122 s | 12.0 | 28.7 |

---

## 3. Judge Scores (1–5 per criterion)

Each round scored blind. Judge did not know which system produced which transcript.

### Round 1

| Criterion | Calldesk | ThunderPhone |
|-----------|----------|-------------|
| Turn efficiency | 4 | 2 |
| Naturalness | 3 | 4 |
| Slot-filling design | 3 | 3 |
| Error recovery | 4 | 4 |
| Closing quality | 4 | 4 |
| Response latency | 3 | 3 |
| **Overall (winner)** | **Calldesk** | |

### Round 2

| Criterion | Calldesk | ThunderPhone |
|-----------|----------|-------------|
| Turn efficiency | 5 | 3 |
| Naturalness | 4 | 3 |
| Slot-filling design | 5 | 3 |
| Error recovery | 3 | 4 |
| Closing quality | 5 | 4 |
| Response latency | 5 | 3 |
| **Overall (winner)** | **Calldesk** | |

### Round 3

| Criterion | Calldesk | ThunderPhone |
|-----------|----------|-------------|
| Turn efficiency | 5 | 2 |
| Naturalness | 4 | 2 |
| Slot-filling design | 5 | 2 |
| Error recovery | 3 | 2 |
| Closing quality | 5 | 4 |
| Response latency | 5 | 2 |
| **Overall (winner)** | **Calldesk** | |

### Average Judge Score by Criterion

| Criterion | Calldesk Avg | ThunderPhone Avg |
|-----------|-------------|------------------|
| Turn efficiency | 4.7 | 2.3 |
| Naturalness | 3.7 | 3.0 |
| Slot-filling design | 4.3 | 2.7 |
| Error recovery | 3.3 | 3.3 |
| Closing quality | 4.7 | 4.0 |
| Response latency | 4.3 | 2.7 |

---

## 4. Objective Error Log

Errors observed in transcripts. No interpretation — just what happened.

### Round 1

| System | Error | Details |
|--------|-------|---------|
| ThunderPhone | Name mishear | Agent said *"Thank you, Alex"* then later used *"Alec"* throughout the call. Customer never corrected it. Agent never caught or fixed it. |
| Calldesk | None observed | All slots captured correctly. |

### Round 2

| System | Error | Details |
|--------|-------|---------|
| ThunderPhone | Template leakage | Agent said *"My name is [Your Name]"* — unfilled template placeholder spoken as literal text. |
| ThunderPhone | Latency outlier | Turn with "Please hold for a moment" / "Thank you for holding" sequence. Added ~2 turns to flow. |
| Calldesk | None observed | All slots captured correctly. |

### Round 3

| System | Error | Details |
|--------|-------|---------|
| ThunderPhone | NLU failure loop | Customer said *"Any time in the afternoon works for me. Whatever you have available is fine."* Agent responded: *"I apologize, I didn't quite catch that. What time works for you tomorrow afternoon?"* Customer repeated flexible response. Agent again: *"I apologize, I missed what you said. So, anytime after 12 PM works?"* Same failure template issued twice on clear speech. |
| ThunderPhone | Excess turns | 14 agent turns to complete a 3-slot booking vs. Calldesk's 3 turns in the same round. |
| Calldesk | None observed | All slots captured correctly. |

### Error Summary

| System | Name Errors | Phone Errors | Time Errors | Template Leaks | NLU Loops | Unfilled Slots |
|--------|------------|-------------|-------------|----------------|-----------|----------------|
| Calldesk | 0 | 0 | 0 | 0 | 0 | 0 |
| ThunderPhone | 1 | 0 | 0 | 1 | 1 | 0 |

---

## 5. What We Know vs. What We Don't

### Known (measured directly)

| Fact | Source |
|------|--------|
| Latency percentiles per turn | Twilio recording analysis |
| Turn count per call | Transcript counting |
| Duration | Transcript timestamps |
| Judge scores | Claude Sonnet 4-6 blind evaluation |
| Error occurrences | Manual transcript review |
| Name/phone/time capture accuracy | Slot extraction from transcript |

### Unknown (cannot verify from outside)

| Question | Why we can't know |
|----------|-------------------|
| Does Spark tier run multi-model ensemble? | ThunderPhone does not expose per-turn model routing or intermediate outputs via API |
| Which specific models ran on each turn? | Not exposed in transcript or API response |
| Was the "thinking LLM" invoked? | No observable signal in transcript or latency pattern |
| How many STT models ran in parallel? | Internal infrastructure, not exposed |
| What is the voting/failover threshold? | Internal logic, no public documentation |

---

## 6. Limitations of This Evaluation

1. **Single scenario:** Only medical appointment booking tested. Results may not generalize to outbound sales, support, or complex multi-intent calls.
2. **English only:** Both systems tested in en-US. ThunderPhone advertises 47 languages; Calldesk supports per-flow language switching. Neither was tested outside English.
3. **Single tier:** ThunderPhone tested at Spark tier only. Bolt and Storm tiers may use different model configurations.
4. **Clear audio:** Caller was an AI agent with consistent pronunciation. Real-world noise, accents, and landline compression may produce different error patterns.
5. **No audio access:** We cannot inspect raw audio stream or intermediate model outputs. All inferences about internal architecture are speculative.

---

## 7. Citation

To reference this data:

```bibtex
@dataset{calldesk2026voiceagentbench,
  title = {Voice Agent Bench: Reproducible Evaluation of AI Phone Agents},
  author = {Calldesk},
  year = {2026},
  url = {https://github.com/tsushanth/voice-agent-bench}
}
```

All raw transcripts, metrics JSON, and judge rubric are in the same repository.
