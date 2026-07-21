# chat-context-export

A Claude Skill that compresses a conversation into a compact, structured JSON handoff — so you can start a fresh chat without losing context.

## The problem

Long conversations degrade. You hit context limits, want to switch models, or just want a clean slate — but re-explaining everything you've decided, ruled out, and built so far is tedious and lossy.

## What this does

Drop `SKILL.md` into your Claude Skills folder, then run one of two commands in any conversation:

- **`/json-here`** — prints a compact JSON summary directly in the chat, ready to copy-paste into a new conversation.
- **`/json-text`** — same summary, delivered as a downloadable `.json` file instead.

Claude reads the conversation (or a pasted/uploaded transcript) itself and produces the JSON — no external script, no API calls, no local model. It's pure prompt-driven compression.

## Output shape

```json
{
  "topic": "...",
  "current_goal": "...",
  "key_decisions": ["..."],
  "constraints": ["..."],
  "open_questions": ["..."],
  "artifacts": [{ "type": "file | code | plan | draft | other", "description": "..." }],
  "rejected_approaches": ["..."],
  "next_steps": ["..."]
}
```

Empty fields are omitted — no filler, no empty arrays. The goal is maximum context density per token, not a complete transcript.

## Usage

1. Install the skill (copy `SKILL.md` into your Claude Skills directory, or upload it wherever your Claude client supports custom skills).
2. In any conversation, type `/json-here` or `/json-text`.
3. Copy the output (or download the file) into a new chat, prefaced with:

   > "Here's structured context from a previous conversation — use it as background, not as a new request:"

## Why JSON instead of a prose summary

The recipient here is another LLM instance, not a human. Structured fields are more reliably re-parsed than free-flowing prose, and the schema forces compression — a bullet list of one-line facts fits in far fewer tokens than a narrative recap covering the same ground.

## License

MIT