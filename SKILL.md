---
name: chat-context-export
description: Compress a raw chat transcript (pasted text, or a .txt/.md transcript file) into a compact structured JSON handoff that can be pasted into a brand-new conversation to restore context instantly. Trigger this skill whenever the user types the commands /json-here or /json-text, or asks in plain language to "summarize this chat for a new conversation," "export context," "give me a handoff," "compact this conversation," or says a chat is getting long and they want to continue it elsewhere. This is not a narrative or literary-style summary for a human to read — the output is a dense, machine-parseable JSON object meant to be re-read by another LLM instance at the start of a new session.
---

# Chat Context Export

Turn a raw conversation transcript into the smallest, most information-dense JSON object that still lets a *new* chat session pick up the work with full context.

## Commands

This skill is invoked with one of two explicit commands. If the user just asks in plain language ("summarize this for a new chat," "give me a handoff," etc.) without typing a command, default to `/json-here` behavior unless they mention wanting a file/download, in which case use `/json-text` behavior.

- **`/json-here`** — Output the JSON directly in the chat, in a fenced code block. Nothing is saved to a file.
- **`/json-text`** — Write the JSON to a `.json` file and deliver it to the user as a downloadable file (in addition to a short confirmation message — do not also paste the full JSON inline for this command, to avoid duplicating a large block).

## Input

The transcript to compress may arrive as:
- The current, ongoing conversation itself (most common case — no file needed, just compress everything said so far)
- A pasted block of raw text in the user's message
- An uploaded `.txt` or `.md` file

If a file is mentioned but not actually present in context, check `/mnt/user-data/uploads/` before asking the user to re-upload it.

## Step 1: Read and understand the transcript

Read the full transcript (or full conversation so far) before writing anything. Identify:
- What the user is actually trying to accomplish (may have shifted over the course of the conversation — capture the *current* goal, not just the original one)
- Decisions, conclusions, or facts that were established
- Explicit constraints or preferences the user stated
- Anything proposed and then explicitly rejected or ruled out
- Artifacts produced (files, code, drafts, plans) and their current state
- What's still open or unresolved
- What should happen next

## Step 2: Produce the JSON

Output **one JSON object** with this shape. Omit any field with no content — do not include empty arrays or null values.

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
      "type": "file | code | plan | draft | other",
      "description": "What it is and its current state, e.g. 'SKILL.md draft, complete, not yet tested'"
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
- **Preserve exact identifiers.** File names, variable names, URLs, commands, and version numbers must be copied verbatim, never paraphrased.
- **Don't summarize short code.** If a code artifact is small (under ~30 lines), include it verbatim in the artifact's `description` (or add a `content` field). If it's long, describe its location/state instead and note the user should re-share it if needed in the new chat.
- **No filler.** Empty sections are omitted entirely, not included as `[]`.

## Step 3: Deliver

### For `/json-here`:
1. Output the JSON in a fenced code block.
2. Immediately above it, include this exact line for the user to copy along with the JSON:

   > Paste this line and the JSON block into your new chat:
   > "Here's structured context from a previous conversation — use it as background, not as a new request:"

3. Keep everything else minimal — no long preamble, no follow-up essay.

### For `/json-text`:
1. Write the JSON to a file (suggested name: `chat-context-<short-topic-slug>.json`) in the outputs directory.
2. Deliver the file to the user.
3. In the accompanying message, include the same one-line paste instruction as above, so the user knows what to say when they upload the file into a new chat.

## Edge cases

- **Multi-topic conversations.** If the transcript covers several unrelated threads, ask the user (one question) whether to compact everything or just the most recent/active thread — don't silently guess on a sprawling, multi-topic chat.
- **Sensitive content.** If the transcript contains information the user likely wouldn't want persisted or pasted elsewhere (health, legal, financial specifics, credentials), leave it out of the JSON by default and note briefly that it was omitted.
- **Too little to export.** If there's only a couple of exchanges with no real state (no decisions, artifacts, or constraints yet), say so plainly rather than producing a near-empty JSON.