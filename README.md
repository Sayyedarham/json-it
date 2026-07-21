# J-code

A portable LLM skill/prompt that compresses a conversation into a compact, structured JSON handoff — so you can start a fresh chat, on any LLM, without losing context.

## The problem

Long conversations degrade. You hit context limits, want to switch models or platforms, or just want a clean slate — but re-explaining everything you've decided, ruled out, and built so far is tedious and lossy.

## What this does

`SKILL.md` is a self-contained instruction file. Give it to any LLM, then run one of two commands in any conversation:

- **`/json-here`** — prints a compact JSON summary directly in the chat, ready to copy-paste into a new conversation.
- **`/json-text`** — same summary, delivered as a downloadable `.json` file instead.

The LLM reads the conversation (or a pasted/uploaded transcript) itself and produces the JSON — no external script, no API calls, no separate model. It's pure prompt-driven compression, and it works the same way whether you're running it in Claude, ChatGPT, Gemini, or anything else that can take custom instructions.

## Installing it

There are two ways to give an LLM this skill, depending on what the platform supports:

### Option A — Native skill/file support (e.g. Claude Skills)
If your platform has a dedicated skills or custom-instructions folder that accepts files, just add `SKILL.md` there directly. The platform will have it available, but it only activates when you type `/json-here` or `/json-text` — it won't run itself off plain-language requests.

### Option B — Copy-paste as a custom instruction (works everywhere else)
Most chat platforms (ChatGPT custom instructions/Projects, Gemini Gems, Claude Projects, or even a plain system-prompt field) accept a block of pasted text instead of a file upload. To use it this way:

1. Open `SKILL.md` in this repo and copy its entire contents (including the `---` frontmatter at the top — it's harmless to include, most models just read it as context).
2. Paste the whole thing into your platform's custom instructions / system prompt / project instructions field.
3. Save it. From then on, typing `/json-here` or `/json-text` in that chat or project will trigger the behavior described in the file.

If your platform doesn't support persistent instructions at all, you can just paste the entire `SKILL.md` contents directly into the chat as your first message, followed by `/json-here` or `/json-text` — it'll work for that single conversation.

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

1. Install the skill using Option A or B above, on whichever LLM you're currently using.
2. In any conversation, type `/json-here` or `/json-text`.
3. Copy the output (or download/save the file) into a new chat — with the same LLM or a different one entirely — prefaced with:

   > "Here's structured context from a previous conversation — use it as background, not as a new request:"

That new chat doesn't need the skill installed at all — it's just receiving a plain JSON block or file as input, so any LLM can read it.

## Why JSON instead of a prose summary

The recipient here is another LLM instance, not a human. Structured fields are more reliably re-parsed than free-flowing prose, and the schema forces compression — a bullet list of one-line facts fits in far fewer tokens than a narrative recap covering the same ground.

## Examples

### Example 1: `/json-here` mid-project

Say you've spent an hour with your LLM of choice designing a database migration script, ruling out two approaches along the way, and you're about to hit a context limit.

**You type:**
```
/json-here
```

**The LLM replies:**

> Paste this line and the JSON block into your new chat:
> "Here's structured context from a previous conversation — use it as background, not as a new request:"

```json
{
  "topic": "Migrating a Postgres users table to add soft-delete support",
  "current_goal": "Write a reversible migration script that adds deleted_at without breaking existing queries",
  "key_decisions": [
    "Use a nullable deleted_at timestamp column, not a separate is_deleted boolean",
    "Existing queries will be updated to filter WHERE deleted_at IS NULL via a view, not per-query edits"
  ],
  "constraints": [
    "Cannot take the table offline during migration — must be a zero-downtime deploy",
    "Must support rollback without data loss"
  ],
  "rejected_approaches": [
    "Hard-delete with an audit log table — rejected, adds complexity user doesn't need",
    "Boolean is_deleted flag — rejected in favor of timestamp for future 'deleted X days ago' queries"
  ],
  "artifacts": [
    { "type": "code", "description": "Draft up-migration SQL adding deleted_at column and the filtered view, complete but untested" }
  ],
  "next_steps": [
    "Write the down-migration (rollback) script",
    "Test zero-downtime behavior against a staging replica"
  ]
}
```

You copy both the instruction line and the JSON into a new chat — even on a different LLM if you want a second opinion. The new conversation now knows the plan, the constraints, and — importantly — what's already been ruled out, so it doesn't re-suggest the boolean flag approach.

---

### Example 2: `/json-text` at the end of a long brainstorm

You've been brainstorming a product launch strategy for a while, the chat is sprawling, and you want a file you can archive or hand to a teammate — not just paste immediately.

**You type:**
```
/json-text
```

**The LLM replies:**

> Saved your context to `chat-context-product-launch-strategy.json`. Paste this line along with the file into your new chat:
> "Here's structured context from a previous conversation — use it as background, not as a new request:"

*(a downloadable `chat-context-product-launch-strategy.json` file is attached, containing the same schema as above — topic, current_goal, key_decisions, constraints, open_questions, next_steps, etc.)*

This is the better option when: the summary is going to sit for a while before you use it, you want to keep an archive of project checkpoints, or you're handing context to someone else rather than pasting it yourself.

---

### Example 3: Compressing a pasted transcript instead of the live chat

You have an old exported conversation (say, a `.txt` transcript from another tool) and want a handoff JSON from it without replaying the whole thing.

**You type:**
```
/json-here

[paste the raw transcript, or attach transcript.txt]
```

The LLM reads the pasted/attached transcript instead of the current conversation, and produces the same JSON schema based on what's in that file. This works the same way regardless of which platform originally produced the transcript, or which LLM you're compressing it with now.

## License

MIT