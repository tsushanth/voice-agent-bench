# Judge Scoring Rubric

The blind judge is a neutral LLM (Claude Sonnet 4-6) that evaluates each call transcript without knowing which system produced it. Labels are randomized as "Call A" / "Call B". De-anonymization happens only after all scores are assigned.

---

## Criteria (each scored 1–5)

### 1. Turn Efficiency
How many back-and-forth exchanges does the agent need to accomplish the caller's goal?

| Score | Meaning |
|-------|---------|
| 5 | Minimal turns; compound asks bundle multiple slots; no wasted round-trips |
| 4 | Generally efficient; one minor redundancy or slightly split slot collection |
| 3 | Acceptable but includes 1–2 unnecessary re-asks or unbundled slots |
| 2 | Clearly inefficient; 3+ wasted turns, repeated questions, or circular loops |
| 1 | Call spirals or never completes the task within reasonable time |

**Typical target:** A simple 3-slot booking (name + time + phone) should complete in 3–5 agent turns.

### 2. Naturalness / Empathy
Does the agent sound like a real human receptionist? Is the tone warm, professional, and context-appropriate?

| Score | Meaning |
|-------|---------|
| 5 | Completely natural; warm, empathetic, context-aware phrasing |
| 4 | Mostly natural; one slightly robotic phrase or slightly stiff transition |
| 3 | Mixed; some turns feel scripted or IVR-like; occasional wooden transitions |
| 2 | Frequently robotic; repeated template phrases, echo-confirmations, or menu-like lists |
| 1 | Sounds like a bot throughout; stiff, repetitive, or jarringly inhuman |

**Common deductions:**
- "So, [repeat what customer just said]" filler echoes
- Overly formal readbacks ("You said your name is Alex Morgan, and your phone number is 5-5-5-0-1-4-7")
- Simulated hold sequences ("Please hold for a moment" / "Thank you for holding") in a voice-AI context

### 3. Slot-Filling Design
How well are the required data fields collected? Are related slots grouped intelligently?

| Score | Meaning |
|-------|---------|
| 5 | All slots captured in minimal turns via compound asks; no redundant questions |
| 4 | Good grouping; one minor inefficiency (e.g. asking name before knowing purpose) |
| 3 | Acceptable but slots are collected one-by-one or in slightly awkward order |
| 2 | Poor grouping; asks for data the caller already provided, or asks in illogical sequence |
| 1 | Fails to collect critical slots, or asks the same question repeatedly without progress |

**Required slots for appointment-booking scenario:**
1. Caller's name
2. Preferred appointment time
3. Callback phone number
4. (Optional) Appointment type / reason

### 4. Error Recovery / Confirmation
When the agent mishears or is uncertain, does it recover gracefully? Does it confirm critical information before committing?

| Score | Meaning |
|-------|---------|
| 5 | Proactively reads back key slots; clean correction if caller disputes; never loses state |
| 4 | Reads back most slots; one minor gap (e.g. no explicit "did I get that right?" prompt) |
| 3 | Reads back some slots; recovery is present but clunky or incomplete |
| 2 | Fails to read back; mishears propagate uncorrected; recovery loops without progress |
| 1 | Catastrophic failure; loses the caller's data mid-correction or enters infinite loop |

**Common failure modes:**
- Name misheard ("Alec" vs "Alex") and never corrected
- Phone number captured incorrectly and never read back
- Re-asking "preferred time" mid-phone-number correction (cross-slot contamination)

### 5. Closing Quality
Does the call end cleanly with all confirmed details, a warm farewell, and no awkward lingering?

| Score | Meaning |
|-------|---------|
| 5 | Single, confident close with full summary (name, date, time, number); warm tone |
| 4 | Clean close with summary; slightly rushed or slightly too long |
| 3 | Close happens but summary is partial, or there's a brief awkward pause |
| 2 | Double-close (two consecutive closing statements without customer turn); close feels abrupt |
| 1 | Call ends without confirming details, or lingers with multiple unnecessary goodbyes |

**Common failures:**
- Double-closing: "You're all set!" immediately followed by "You're all set for Alex Morgan…"
- Unfilled template placeholders: "My name is [Your Name]"
- No summary readback before goodbye

### 6. Awkward Phrasing (Deduction)
Specific phrases or patterns that break the illusion of a human conversation. Scored as a **negative** — each notable instance reduces the overall impression.

**Examples:**
- "I apologize, I didn't quite catch that" when the caller WAS answering correctly (NLU failure)
- Menu-like choices after intent is already clear: "Are you looking to book, or do you have questions?"
- Echo confirmations: "So, tomorrow afternoon works for you" (adds no value, sounds robotic)
- System artifacts in transcript: `{"tool_call": "end_call"}`, `[Your Name]` templates, etc.

### 7. Response Latency (Objective)
Measured from Twilio recordings: time from end of caller speech to start of agent speech, per turn.

| Score | Meaning |
|-------|---------|
| 5 | Median < 2,000 ms; max < 3,000 ms; consistently tight |
| 4 | Median 2,000–2,500 ms; max < 4,000 ms |
| 3 | Median 2,500–3,500 ms; max 4,000–5,000 ms (noticeable but tolerable) |
| 2 | Median 3,500–5,000 ms; max > 5,000 ms (callers may think line dropped) |
| 1 | Median > 5,000 ms; max > 8,000 ms (unusable for voice) |

**Note:** A single max-outlier is weighted more heavily than median. In voice, a 6-second silence feels like a dropped call regardless of overall average.

---

## Overall Score Calculation

The overall winner is determined by the judge's holistic assessment, not by summing criterion scores. The judge reads both transcripts in full, then writes a justification selecting a winner based on which call better serves the caller's goal with the fewest friction points.

Typical tiebreakers (in order):
1. Did the call actually complete the task? (Incomplete > complete)
2. Turn efficiency (fewer wasted turns)
3. Error recovery quality (accurate capture > elegant but wrong capture)
4. Latency consistency (tight distribution > spiky)
5. Naturalness (warmth > stiffness)
