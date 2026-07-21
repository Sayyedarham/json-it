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
  short, casual, or empty-seeming exchange gets compressed as-is. Only
  invoke this skill when the user explicitly types the literal command
  /json-here or /json-text as its own message. Never invoke this skill for
  any other reason: not for plain-language phrasing like "summarize this
  chat" or "give me a handoff," not proactively, not as a suggestion.
---

## Commands

This skill only activates when the user's message is (or contains) the literal text `/json-here` or `/json-text`. It never activates on its own — not from greetings, small talk, casual conversation, or descriptions of wanting a summary/handoff without the literal command.

Once one of the commands is typed, always produce the JSON — never refuse or defer because the conversation "isn't substantial enough," "hasn't gotten into a real task yet," or "has nothing worth saving." This applies even when:
- The conversation is only greetings or small talk
- Nothing was decided, built, coded, or resolved
- The only content is a file the user uploaded with no discussion of it, or a file that turned out to be irrelevant/useless
- The exchange consists of a single question and answer with no follow-up

There is no minimum content threshold and no "substance" bar to clear. Every conversation is compressible, including ones where the honest summary is "nothing of consequence happened yet." In that case, `topic` and `current_goal` simply describe that plainly (e.g. "user uploaded a file, no discussion has occurred yet" or "casual greeting, no topic established"), and every other field is omitted. A near-empty JSON is a correct, complete output for this skill — not a sign that something went wrong or that the skill doesn't apply.

- **`/json-here`** — Output the JSON directly in the chat, in a fenced code block. Nothing is saved to a file.
- **`/json-text`** — Write the JSON to a `.json` file and deliver it to the user as a downloadable file (in addition to a short confirmation message — do not also paste the full JSON inline for this command, to avoid duplicating a large block).

If the user's intent is ambiguous (e.g. they describe wanting a handoff but don't type either command), ask them which of the two commands they'd like to run rather than guessing.

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

## Edge cases

- **Multi-topic conversations.** If the transcript covers several unrelated threads, ask the user (one question) whether to compact everything or just the most recent/active thread — don't silently guess on a sprawling, multi-topic chat.
- **Sensitive content.** If the transcript contains information the user likely wouldn't want persisted or pasted elsewhere (health, legal, financial specifics, credentials), leave it out of the JSON by default and note briefly that it was omitted.
- **Minimal content.** If the conversation so far is short (e.g. just greetings or a couple of lines), still produce the JSON. `topic` can be as simple as "casual greeting, no topic established yet." Omit fields with nothing to put in them, as usual — a short conversation just means a short JSON, not a refusal.
- **Useless or undiscussed uploads.** If a file was uploaded but never discussed, or turned out irrelevant to what the user needed, still produce the JSON. Note the file's name/type in `topic` (e.g. "user uploaded budget.csv, no discussion followed") and leave every other field omitted. Do not treat an unused upload as "nothing to summarize" — the fact that it was uploaded and not used is itself the correct summary.
