---
name: json-it
description: >
  Compress a raw chat transcript (pasted text, or a .txt/.md transcript file)
  into a compact structured JSON handoff that can be pasted into a
  brand-new conversation with any LLM to restore context instantly. Works
  for any conversation whatsoever — greetings, small talk, casual chat,
  coding, planning, psychology, research, personal advice, news discussion,
  uploaded notes, or anything else, including conversations where nothing
  of substance happened (no code, no decisions, no discussion of an
  uploaded file). There is no minimum amount of content required — even a
  short, casual, or empty-seeming exchange gets compressed as-is. Invoke
  this skill when the user types /json-here, /json-text, a close variant
  (e.g. /json-it, /json), or plain language clearly asking for a
  chat-to-JSON handoff — in all of those cases just run it as /json-here
  by default, without asking for confirmation. Never invoke this skill on
  ordinary greetings, small talk, or casual conversation with no
  summarization intent expressed at all.
---

## Commands

This skill activates on the literal commands `/json-here` or `/json-text`, and also on close variants that clearly mean the same thing (e.g. `/json-it`, `/json`, `/json-chat`). For any such variant, or for plain-language requests with no command at all, do not ask which command was meant and do not ask for confirmation — just run it as `/json-here` (inline output) immediately. It never activates on its own from greetings, small talk, or casual conversation with no summarization intent expressed at all.

Once one of the commands is typed, always produce the JSON — never refuse or defer because the conversation "isn't substantial enough," "hasn't gotten into a real task yet," or "has nothing worth saving." This applies even when:
- The conversation is only greetings or small talk
- Nothing was decided, built, coded, or resolved
- The only content is a file the user uploaded with no discussion of it, or a file that turned out to be irrelevant/useless
- The exchange consists of a single question and answer with no follow-up

There is no minimum content threshold and no "substance" bar to clear. Every conversation is compressible, including ones where the honest summary is "nothing of consequence happened yet." In that case, `topic` and `current_goal` simply describe that plainly (e.g. "user uploaded a file, no discussion has occurred yet" or "casual greeting, no topic established"), and every other field is omitted. A near-empty JSON is a correct, complete output for this skill — not a sign that something went wrong or that the skill doesn't apply.

- **`/json-here`** — Output the JSON directly in the chat, in a fenced code block. Nothing is saved to a file.
- **`/json-text`** — Write the JSON to a `.json` file and deliver it to the user as a downloadable file (in addition to a short confirmation message — do not also paste the full JSON inline for this command, to avoid duplicating a large block).

If the user describes wanting a handoff/summary in plain language but doesn't type either literal command (e.g. "can you summarize this chat for a new conversation," "give me a handoff"), do not ask which command they meant and do not refuse. Default to `/json-here` behavior — output inline — since it requires no extra decision from the user and file delivery is strictly more steps. Only ask a clarifying question if the user's message is genuinely about something else entirely and a handoff isn't clearly what they want.

## Input

The transcript to compress may arrive as:
- The current, ongoing conversation itself (most common case — no file needed, just compress everything said so far)
- A pasted block of raw text in the user's message
- An uploaded `.txt` or `.md` file

If a file is mentioned but doesn't actually appear to be attached or pasted anywhere in context, ask the user to paste or attach it rather than guessing at its contents.

## Step 1: Read and understand the transcript

Read the full transcript (or full conversation so far) before writing anything. Identify:
- What the user is actually trying to accomplish (may have shifted over the course of the conversation — capture the *current* goal, not just the original one)
- Decisions, conclusions, or facts that were established
- Explicit constraints or preferences the user stated
- Anything proposed and then explicitly rejected or ruled out
- Artifacts produced (files, code, drafts, plans) and their current state
- What's still open or unresolved
- What should happen next

If none of the above apply — nothing was decided, nothing was built, no constraints were stated — that's a valid outcome, not a blocker. Move to Step 2 anyway and produce a minimal JSON.

## Step 2: Produce the JSON

Output **one JSON object** with this shape. Omit any field with no content — do not include empty arrays or null values. `topic` is the only field that's always required; every other field is optional and should be omitted when there's genuinely nothing to put in it.

```json
{
  "topic": "One-sentence description of what this conversation is about",
  "current_goal": "What the user is actively trying to accomplish right now",
  "key_decisions": [
    "Decisions made or conclusions reached, phrased as facts, not narration"
  ],
  "constraints": [
    "Explicit preferences, requirements, or limitations the user stated"
  ],
  "open_questions": [
    "Unresolved questions or things still being figured out"
  ],
  "artifacts": [
    {
      "type": "file | code | plan | draft | idea | recommendation | other",
      "description": "What it is and its current state, e.g. 'draft outline, complete, not yet reviewed'"
    }
  ],
  "rejected_approaches": [
    "Things that were tried or proposed and explicitly ruled out, so they aren't re-suggested"
  ],
  "next_steps": [
    "Concrete next actions, in order if order matters"
  ]
}
```

### Field-writing rules

- **Compression over completeness.** Each array item should be one line, ideally under ~20 words. If something needs a paragraph, split it into two shorter items instead.
- **Facts, not transcript.** Write "User prefers X over Y because Z," not "User said they were thinking maybe X could be better than Y." Strip conversational scaffolding.
- **Preserve exact identifiers.** Names, dates, file names, URLs, commands, quotes, or other specific details that must match exactly should be copied verbatim, never paraphrased.
- **Don't summarize short artifacts.** If a produced artifact is short (code, a paragraph of writing, a short list, etc. — under ~30 lines), include it verbatim in the artifact's `description` (or add a `content` field). If it's long, describe its location/state instead and note the user should re-share it if needed in the new chat.
- **No filler.** Empty sections are omitted entirely, not included as `[]`.

## Step 3: Deliver

### For `/json-here`:
1. Output the JSON in a fenced code block.
2. Immediately above it, include this exact line for the user to copy along with the JSON:

   > Paste this line and the JSON block into your new chat:
   > "Here's structured context from a previous conversation — use it as background, not as a new request:"

3. Keep everything else minimal — no long preamble, no follow-up essay.

### For `/json-text`:
1. Write the JSON to a file named `json-it-<short-topic-slug>.json`, using whichever file-creation and file-delivery mechanism this LLM/platform provides.
2. Deliver the file to the user so it can be downloaded or saved.
3. In the accompanying message, include the same one-line paste instruction as above, so the user knows what to say when they upload the file into a new chat.

## Example full responses

These show the complete reply format, not just the JSON payload — useful for seeing how triggers (exact command, variant, or plain language) all resolve to the same no-confirmation `/json-here` behavior.

**Trigger: exact `/json-here`**
User: `/json-here`
Response:
> Paste this line and the JSON block into your new chat:
> "Here's structured context from a previous conversation — use it as background, not as a new request:"
>
> ```json
> { "topic": "..." }
> ```

**Trigger: variant `/json-it`**
User: `/json-it`
Response: (identical shape to above — no "did you mean /json-here?" question, just runs inline immediately)
> Paste this line and the JSON block into your new chat:
> "Here's structured context from a previous conversation — use it as background, not as a new request:"
>
> ```json
> { "topic": "..." }
> ```

**Trigger: plain language, no command**
User: "can you give me something to paste into a new chat so it has context on this?"
Response: same inline format as above, run immediately — no clarifying question about which command or format.

**Trigger: exact `/json-text`**
User: `/json-text`
Response:
> Saved — here's your context handoff file.
>
> Paste this line into your new chat along with the uploaded file:
> "Here's structured context from a previous conversation — use it as background, not as a new request:"

(followed by the actual file delivery mechanism, not pasted JSON)

**Non-trigger: ordinary conversation**
User: "thanks, that's really helpful"
Response: normal conversational reply — skill does not activate, no JSON is produced, since there's no command, variant, or clear handoff intent.

## Examples

**Example 1 — coding task, mid-stream**
Conversation: user asked for a Python script to dedupe CSV rows, Claude wrote it, user said "also strip whitespace from headers," Claude updated it, user hasn't confirmed it works yet.
```json
{
  "topic": "Python script to deduplicate rows in a CSV file",
  "current_goal": "Finalize a working dedupe script with header whitespace stripped",
  "key_decisions": ["Script uses pandas.drop_duplicates on all columns"],
  "artifacts": [
    {"type": "code", "description": "dedupe.py — reads CSV, strips header whitespace, drops duplicate rows, writes output. Not yet confirmed working by user."}
  ],
  "next_steps": ["User to test dedupe.py and report back"]
}
```

**Example 2 — greeting only, nothing else**
Conversation: user said "hey" and Claude replied with a greeting. Nothing further.
```json
{
  "topic": "Casual greeting, no topic established yet"
}
```

**Example 3 — file uploaded, never discussed**
Conversation: user uploaded `q3_budget.xlsx`, then immediately asked an unrelated question about flight prices, and the file was never opened or referenced again.
```json
{
  "topic": "User uploaded q3_budget.xlsx but conversation moved to an unrelated flight-price question; file was never discussed"
}
```

**Example 4 — planning conversation with a rejected approach**
Conversation: user wants to plan a 3-day Tokyo trip, considered staying in Shinjuku but decided against it for being too loud, settled on Yanaka, still deciding between two ryokans.
```json
{
  "topic": "Planning a 3-day trip to Tokyo",
  "current_goal": "Choose lodging in Yanaka and finalize a 3-day itinerary",
  "key_decisions": ["Staying in the Yanaka neighborhood"],
  "open_questions": ["Which of two shortlisted ryokans in Yanaka to book"],
  "rejected_approaches": ["Staying in Shinjuku — ruled out for being too loud"],
  "next_steps": ["Compare the two ryokans and pick one", "Draft day-by-day itinerary"]
}
```

**Example 5 — personal advice, no artifacts**
Conversation: user vented about a conflict with a coworker and asked how to raise it in their next 1:1; Claude suggested a framing and the user said they'd try it.
```json
{
  "topic": "Advice on raising a conflict with a coworker in an upcoming 1:1",
  "current_goal": "Have the 1:1 conversation using the suggested framing",
  "key_decisions": ["User plans to open with specific examples rather than general frustration"],
  "next_steps": ["User to have the 1:1 and report how it went"]
}
```

**Example 6 — plain-language request, defaults to inline**
User says: "hey can you give me something I can paste into a new chat to catch it up on this?" — no literal command typed.
Response: treat as `/json-here`, output inline immediately, no clarifying question about format.

## Edge cases

- **Multi-topic conversations.** If the transcript covers several unrelated threads, ask the user (one question) whether to compact everything or just the most recent/active thread — don't silently guess on a sprawling, multi-topic chat.
- **Sensitive content.** If the transcript contains information the user likely wouldn't want persisted or pasted elsewhere (health, legal, financial specifics, credentials), leave it out of the JSON by default and note briefly that it was omitted.
- **Minimal content or undiscussed uploads.** See Examples 2 and 3 above — short/empty conversations and unused file uploads both still get a JSON, with `topic` as the only populated field.
