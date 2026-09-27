# Reproducing the Benchmark

## Prerequisites

- Node.js 20+ and `npm` (for the judge script)
- Python 3.11+ (for the API wrapper)
- Bash (for the runner)
- Accounts and keys for:
  - **Twilio** (for placing real phone calls)
  - **ThunderPhone** (the system being evaluated)
  - **Calldesk** (the system being evaluated)
  - **Anthropic API** (for the blind judge; or `claude` CLI)

## Environment variables

```bash
export THUNDERPHONE_API_KEY="sk_live_..."
export THUNDERPHONE_NUMBER="+15072603370"   # TP inbound number with benchmark agent
export THUNDERPHONE_AGENT_ID="240"          # TP agent ID for transcript filtering
export CALLDESK_NUMBER="+12245061194"       # Calldesk inbound number
export TEST_CALL_SECRET="..."               # call-loop-poc /place-test-call secret
export TWILIO_ACCOUNT_SID="AC..."
export TWILIO_AUTH_TOKEN="..."
```

## Running a single round

```bash
cd calldesktech/bench/
./thunderphone-bench.sh --rounds 1
```

This will:
1. Place two simultaneous outbound calls from the same Twilio number
2. Wait for both calls to complete
3. Pull transcripts from both systems
4. Download Twilio recordings
5. Run objective latency analysis on both recordings
6. Run the blind judge
7. Print a summary

## Adding a new system

To evaluate your own voice-agent platform against the same scenario:

1. **Set up your inbound number** with an agent that handles:  
   *"Book a medical appointment for tomorrow afternoon — collect name, time, callback number, confirm, close."*

2. **Add your system to the benchmark harness** by editing `thunderphone-bench.sh`:
   - Add `YOUR_NUMBER` and `YOUR_AGENT_ID` (or equivalent lookup)
   - Add a transcript-fetching function (REST API, webhook, or log scraping)
   - Add your system to the comparison loop

3. **Run 3 rounds** and capture:
   - Full turn-by-turn transcripts
   - Twilio recordings (for objective latency)
   - System config (STT/LLM/TTS used, version)

4. **Submit a PR** to `github.com/calldesk/voice-agent-bench` adding:
   - Your transcripts to `data/transcripts.jsonl`
   - Your metrics to `data/metrics.jsonl`
   - System metadata to a new file in `systems/`

## Running the judge standalone

If you already have two transcript files:

```bash
node mystery-shopper-judge-neutral.mjs \
  --a round-1-caldesk.txt --label-a "Calldesk" \
  --b round-1-thunderphone.txt --label-b "ThunderPhone" \
  --metrics-a round-1-caldesk-metrics.json \
  --metrics-b round-1-thunderphone-metrics.json
```

The judge reads both transcripts, scores each on the criteria in `rubric.md`, and outputs a blind verdict.

## Extending the scenario

Edit `data/scenarios.jsonl` to add new test cases:
- Rescheduling an existing appointment
- Transferring to a human
- Handling an angry caller
- Bilingual switching mid-call
- ...etc

Each new scenario needs:
1. A shopper persona with specific goals and constraints
2. Success criteria the judge will check for
3. A system prompt for the AI shopper

## Known limitations

- **Shopper consistency:** The AI shopper's behavior varies slightly across LLM temperature settings. We pin `temperature=0.7` and use a deterministic system prompt to minimize variance.
- **STT variance:** Real telephone audio introduces noise, compression, and carrier-specific artifacts that vary call-to-call. We run 3 rounds and report aggregate to smooth this.
- **Single scenario:** Currently only medical-appointment booking. Expanding to multi-scenario is the next priority.
- **English only:** Both systems tested in en-US. Calldesk supports per-flow language switching; ThunderPhone advertises 47 languages — neither has been tested outside English in this dataset.
