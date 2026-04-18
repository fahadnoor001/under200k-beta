---
title: "Under 200k — a field guide to heavy Claude Code sessions"
author: "Fahad Noor"
date: "2026"
version: "1.0"
---

# Under 200k

### A field guide to heavy Claude Code sessions

**Fahad Noor · 2026**

> Copyright © 2026 Fahad Noor. All rights reserved. This document and its scripts are licensed for personal use by the original purchaser. Redistribution, resale, or inclusion in derivative products is prohibited.

---

## Why this exists

You already know what the 200k wall feels like. You're forty turns into a refactor, Claude has the full mental model of your codebase, and somewhere around turn forty-one the context bar goes red. Autocompact triggers. Older messages get summarized into oblivion. The file contents Claude had read earlier? Gone. The design decision you argued out three hours ago? Summarized into one line that loses the reasoning.

You paste the file again. Claude thanks you and keeps working. But the thing you were building — the specific thread of intent — is now a lossy reconstruction.

This guide is about not getting there. Not because hitting 200k is a moral failure — it isn't — but because most of the tokens you're burning are noise you never needed to load. The 60k MCP tools you don't use in this session. The skill descriptions that trigger on phrases you never type. The memory files that haven't been relevant for six months. The capabilities index that gets re-injected every turn.

Four moves. Each grounded in something I verified against my own setup, not documentation.

**I.** A context audit you can run in under two minutes.
**II.** Where MCP sprawl actually lives.
**III.** A response protocol enforced by a Stop-hook.
**IV.** Verify the binary, not the docs.

The appendix has the two scripts — statusline and Stop-hook — ready to drop into `~/.claude/`. Together they're about 200 lines.

---

## How to use this guide

Read Chapter I today. Run the audit. If you're surprised by what you find (most people are), keep reading.

Don't skip to the scripts without reading Chapter III. The scripts enforce a protocol that's only useful if you understand why each piece exists — otherwise you'll delete the parts that annoy you and lose the value.

Every code block is meant to be copied. Every click path is the exact path. If something here doesn't match your version of Claude Code, grep the binary (Chapter IV shows you how) — don't assume I'm wrong until you've checked.

---

## Chapter I — A context audit you can run in under two minutes

### What the context window actually is

The 200k token window is a shared budget across eight or nine distinct components. You don't control most of them directly. But every turn, Claude Code sends the model a prompt shaped roughly like this:

```
[system prompt from the CLI binary]
[tool schemas — system tools like Read/Edit/Bash]
[tool schemas — MCP tools from connectors]
[skill definitions from ~/.claude/skills/]
[custom agent definitions from ~/.claude/agents/]
[CLAUDE.md files — user, project, memory index]
[every memory file referenced by MEMORY.md]
[the conversation transcript — every user turn, every tool call, every result]
[the current user turn]
```

Every single one of these eats tokens. Some are fixed overhead per session. Some grow every turn. Two of them — messages and MCP tool schemas — usually account for 70–90% of what you're burning.

### The audit checklist

Run this on a session that feels heavy. Takes 90 seconds if you know where to look.

**1. Check the headline number.**
In Claude Code, the context-window indicator is surfaced via the statusline. If you haven't installed one, skip to Appendix A and install it now, or use `/context` if your CLI version supports the slash command. You want one number: **percentage of 200k used**.

**2. Break it down by component.**
The `/context` command shows a per-component breakdown. Note the top three by token count. For most sessions at 50%+, the breakdown looks like:

| Component | Typical share at 50% |
|---|---|
| Messages | 40–60% |
| MCP tools (deferred) | 15–30% |
| Autocompact buffer (reserved) | 15% |
| Skills | 3–8% |
| Memory files | 2–6% |
| Custom agents | 2–5% |
| System prompt + tools | 5–8% |

The buffer is mandatory overhead — you can't reclaim it. Everything else is fair game.

**3. If Messages is >60%, that's the flight recorder leaking.**
Long conversations accumulate tool results. Every file you had Claude read is still sitting in the transcript verbatim. Every 500-line diff. Every Bash output. The fix isn't to be more careful — the transcript is append-only. The fix is to start a new session and carry only what matters (`/resume` with pruning, or manually re-state the active subproblem).

**4. If MCP tools is >25%, Chapter II is your chapter.**
Skip ahead. The fix takes five minutes and doesn't require uninstalling anything.

**5. If Skills + Agents + Memory files combined is >20%, you have declaration bloat.**
Skills with vague descriptions fire on too many prompts. Custom agents you defined six months ago for a one-off task still get loaded every session. Memory files that should have been consolidated into one. Walk `~/.claude/skills/` and `~/.claude/agents/` — anything you don't recognize, rename it with a `.disabled` suffix and see if you miss it in a week.

**6. If System tools + MCP tools (non-deferred) is your top line, rare but possible — you've enabled too many connectors.**
Connectors you "might use someday" are the single biggest preventable drag. Same browser-side fix as Chapter II.

**7. Note the autocompact-imminent threshold.**
Claude Code triggers autocompact at ~92% by default. If you're at 85%, you have roughly one heavy turn before you lose context. Wrap up the current subproblem, save any key state to a memory file or plan, then start fresh.

### The two usual suspects

Across every audit I've run on my own sessions and helped others run on theirs, the same pair dominates:

- **Messages** — the active transcript, which grows with every turn and can't be trimmed retroactively.
- **MCP tool schemas** — loaded once per session, but at 100k+ tokens for users with 15+ connectors enabled.

If your audit shows something else at the top, you're an outlier and the rest of this book is less urgent for you. If it shows these two, you're normal.

### What "under two minutes" actually means

Ninety seconds to run `/context` and read the output. Thirty seconds to mentally flag the top two. You do not need spreadsheets or screenshots. The audit is useful because it's fast enough to do mid-session when you notice the statusline turning yellow. Any slower and you won't do it.

---

## Chapter II — Where MCP sprawl actually lives

### The lie of the clean settings.json

You open `~/.claude/settings.json`. It has two or three MCP servers listed. You think you have a small footprint.

You run the context audit. MCP tools are 60k+ tokens.

The mismatch is that Claude Code loads MCP tools from **three places**, not one:

1. **Local CLI config** — `~/.claude/settings.json` (the one you edit).
2. **Plugin-bundled MCP servers** — any plugin installed from the marketplace can declare its own MCP servers in its `plugin.json`. These load automatically.
3. **Browser-enabled connectors on claude.ai** — this is the big one. Connectors you authorized in the browser (Notion, Google Drive, Supabase, Figma, Gmail, etc.) are available inside Claude Code too, and their tool schemas load on every session.

That third bucket is where the real weight lives. I once audited my own setup and found 200+ MCP tools across eleven connectors I barely used.

### The exact click path to audit + prune

This is the browser-side fix. It requires no CLI changes.

1. Open **claude.ai** in your browser, signed into the same account you use with Claude Code.
2. Top-right avatar → **Settings**.
3. Left sidebar → **Connectors** (sometimes labeled "Integrations" or "MCP servers" depending on rollout stage).
4. You'll see a list of every connector authorized on your account. For each one:
   - If you haven't used it in a month: **Disconnect**.
   - If you use it occasionally but not in Claude Code sessions: **Disconnect for CLI** (if the toggle exists) or fully disconnect and re-auth when needed.
   - If you use it regularly in the browser but never in Claude Code: same — disconnect for CLI only.

The tools don't disappear from the browser interface — browser Claude still works. But they stop loading into your CLI sessions.

5. After pruning, restart your Claude Code CLI (quit and reopen the terminal, or `/reset` if your version supports it). The next session's context-window audit should show a dramatic drop in MCP tool tokens.

### Budget heuristic

Aim for **under 20k tokens total for MCP tools** in any given session. If a connector's schema is more than 5k tokens on its own, you almost certainly don't need it loaded constantly — enable it per-session only when you're doing work that touches it.

### Deferred MCP tools

Claude Code 2.x introduced a "deferred" mode for MCP tool schemas — instead of loading all tool definitions upfront, it loads only tool names and descriptions (a small index), and fetches full schemas on-demand when Claude decides to use a tool. If your CLI version supports it, enable it. Check your settings for `mcp.deferred` or similar — the flag varies by version.

The trade-off: Claude needs one extra round-trip to discover a tool's exact parameters when it first uses it. For CLI workflows that use 2–3 specific tools per session, deferred mode can save 40k+ tokens.

### What this doesn't fix

Pruning connectors doesn't help with messages, skills, agents, or memory. Chapter I's audit tells you whether this chapter is your bottleneck. Don't prune connectors you actually need just because the number looks big — if they're earning their weight, they're earning their weight.

---

## Chapter III — A response protocol enforced by a Stop-hook

### Why responses drift

The longer a Claude session runs, the more its responses drift toward one of two failure modes:

- **Acknowledgment mode:** responses shrink into "Done!" / "Sure!" / "On it." with no teaching, no reflection, no flag on what could go wrong.
- **Wall-of-text mode:** responses bloat into multi-section explanations of what Claude did, heavy on "what," silent on "why."

Both modes make the session harder to review, harder to learn from, and harder to catch when Claude makes a bad judgment call. The response protocol fixes both by forcing a specific structure on every non-trivial response.

### The five mandatory sections

Every response over roughly 400 characters ends with these subheaders, in order:

#### `### Under the hood`

Teach the *why* behind this turn's non-obvious choices. Not a recap of what was done — a teaching of why. One-sentence reasons for tool selection, design trade-offs, ordering. Written so the reader could re-teach it to someone else.

**Why this exists:** most drift toward "what" and away from "why" because "why" is harder to generate. Forcing the section costs a sentence and saves hours of post-hoc debugging ("why did Claude pick X?"). Cap at 100 words.

#### `### Harshest critic`

Name specific weaknesses in *this* response with a counterfactual — "if I had done X instead of Y, outcome would have been Z." No performative humility ("I could have been clearer"). Real failure modes, named.

**Why this exists:** Claude is biased toward confidence. The section creates structural space to surface doubt, so the human can catch mistakes Claude would otherwise gloss over. Cap at 100 words.

#### `### Score`

Format: `Score: X/5 — <one-line improvement note>`.

**Why this exists:** a numerical self-assessment across turns makes it possible to spot sessions where Claude's own confidence is drifting down. When several turns in a row score 2/5 or 3/5, that's a signal to stop, reset context, or switch tactics.

#### `### Scalable paths`

One to three bullets, each tagged one of `[EXECUTE]`, `[DEFER: <unblocker>]`, `[REJECT: <reason>]`, or `[PROPOSE: <reason>]`.

- `[EXECUTE]` — done in this turn before emitting.
- `[DEFER]` — not done this turn because of a specific named blocker (credentials missing, user confirmation needed, dependency on another step).
- `[REJECT]` — considered and rejected, with the reason named.
- `[PROPOSE]` — worth considering, not blocked, not declined, but not autonomously actionable.

**Why this exists:** forces Claude to separate what was done, what's next, and what was considered and dismissed. Makes it possible to audit "did Claude explore alternatives" without asking.

#### `### Capabilities` — *optional*

Only present when at least one of Skills / Agents / Connectors / Plugins was actually used. Lists them as `[USING: name]` for active use or `[PROPOSED: name]` for a suggestion. If all four rows would be empty, the block is omitted entirely.

**Why this exists:** the inverse of tool-selection drift. If Claude is always using the same three skills, the session is leaving leverage on the table. The block makes that visible.

### The Stop-hook script

This is the script that enforces the protocol. Drop it at `~/.claude/hooks/response-protocol.sh` (macOS/Linux) and register it in `settings.json` under the `hooks.Stop` key.

Full script in **Appendix B**. The core logic:

1. On every Stop event, Claude Code runs the hook.
2. The hook reads the session transcript, finds the last assistant message.
3. If the message is under 400 chars, skip — short responses (yes/no/acknowledgments) are exempt.
4. Otherwise, grep for each of the five required subheaders.
5. If any are missing, emit a reminder to stdout. Claude Code surfaces hook stdout as additional context on the next turn — so the next response will see the reminder and self-correct.

The hook is **non-blocking**. It never exits non-zero, never mutates the transcript. It just nudges.

### How to install

```bash
# 1. Copy the script into the hooks directory
mkdir -p ~/.claude/hooks
# (paste the script from Appendix B, make it executable)
chmod +x ~/.claude/hooks/response-protocol.sh

# 2. Register in settings.json
# Open ~/.claude/settings.json and add (or merge into existing hooks):
```

```json
{
  "hooks": {
    "Stop": [
      {
        "matcher": "",
        "hooks": [
          { "type": "command", "command": "bash $HOME/.claude/hooks/response-protocol.sh" }
        ]
      }
    ]
  }
}
```

The `matcher: ""` means "every Stop event." If you want to scope it narrower (e.g., only certain projects), use a regex matching the session's working directory.

### Failure modes to expect

- **First few turns feel heavy.** The protocol adds ~300–500 tokens per response. Worth it.
- **Short responses stay short.** The 400-char exemption means "yes," "proceed," "done" don't get badgered.
- **Claude occasionally cheats.** Early on, Claude may write minimal `### Under the hood` sections that restate what was done instead of teaching the why. The fix is to call it out in conversation ("your under-the-hood is a recap, not a teaching"). It self-corrects within one or two turns.
- **The hook doesn't know about sarcasm.** If Claude writes `### Harshest critic` as one performative line of fake humility, the hook will still pass it. The protocol only works if you read the critic section and push back when it's weak.

### The `<100 words` rule

Sections 1 and 2 (`Under the hood` and `Harshest critic`) are capped at 100 words each. Without the cap, they drift toward multi-paragraph essays. The cap forces the writer to lead with the sharp point.

---

## Chapter IV — Verify the binary, not the docs

### Why docs lag

Claude Code ships weekly. Documentation ships quarterly. The CLI you installed yesterday has fields, flags, and hook events that don't appear in the public docs yet.

Trusting docs over the binary means:
- You build a statusline that uses the pre-2.0 statusline schema, and it silently fails on the new `context_window.used_percentage` field.
- You write a hook against a deprecated event name.
- You recommend a config key that was renamed three releases ago.

The fix is a sixty-second pattern that works on any npm/brew/pip-installed CLI.

### The 60-second grep pattern

```bash
# 1. Find where the CLI is actually installed
which claude                                   # returns the binary path
# or for npm globals:
npm ls -g --depth=0 --parseable | grep claude  # returns the package path

# 2. Find the bundled JS (most Node CLIs bundle to a single file)
ls -la "$(dirname $(which claude))/../lib/node_modules/@anthropic-ai/claude-code/" 2>/dev/null
# or on Windows:
# dir "C:\Users\$env:USERNAME\AppData\Roaming\npm\node_modules\@anthropic-ai\claude-code"

# 3. Grep for the feature you care about
grep -n "context_window" cli.js | head -20
grep -n "statusLine" cli.js | head -20
grep -n "used_percentage" cli.js | head -20
```

Step 3 gives you field names, nested paths, and default values directly from the shipped code. If the docs say "the statusline receives `model_info.name`" but grep shows `model.display_name`, trust grep.

### Walk-through: finding `context_window.used_percentage`

The Claude Code statusline docs, as of early 2026, describe the stdin schema as including `model`, `workspace`, `cost`, and a few other fields. Context window usage isn't documented.

But it's there. From `cli.js` in the bundled CLI (verified against 2.1.109):

```
context_window.used_percentage
context_window.remaining_percentage
context_window.current_usage
context_window.context_window_size
exceeds_200k_tokens
```

These are the fields that let you build a red/yellow/green context indicator without guessing. The statusline script in **Appendix A** uses exactly these fields.

### Why this matters beyond statuslines

The same pattern works for:
- **Hook event names.** The docs list five or six. Grep shows there are eleven or twelve, including undocumented ones useful for advanced workflows.
- **Tool-result shape.** When you write a custom agent or skill that needs to parse tool output, grep the code that generates it.
- **Config key names.** `settings.json` has more valid keys than the docs list — grep for `config.` or the getter functions.
- **Any third-party CLI.** The same trick works on `gh`, `supabase`, `doppler`, `gcloud`. Pip and brew installs are even easier — the source is almost always right there, un-minified.

### The sixty-second rule

If you're about to spend more than a minute reasoning about what a CLI tool might support, stop reasoning and grep. The answer is in the binary, not in your head and not in the docs.

---

## Appendix A — The statusline script

Drop this at `~/.claude/hooks/statusline.sh`, make it executable, and register it in `settings.json` under the `statusLine.command` key.

```bash
#!/usr/bin/env bash
# Claude Code statusLine (CLI 2.1.x). Reads session JSON on stdin.
# Schema verified against bundled cli.js 2.1.109:
#   model.display_name, workspace.current_dir, output_style.name,
#   cost.total_cost_usd,
#   context_window.{used_percentage, remaining_percentage, current_usage, context_window_size},
#   exceeds_200k_tokens.

set -u
JSON="$(cat)"
get() { printf '%s' "$JSON" | jq -r "$1 // empty" 2>/dev/null; }

MODEL=$(get '.model.display_name')
DIR=$(get '.workspace.current_dir')
COST=$(get '.cost.total_cost_usd')
USED_PCT=$(get '.context_window.used_percentage')
OVER=$(get '.exceeds_200k_tokens')

BASE="${DIR##*/}"; [[ -z "$BASE" ]] && BASE="$DIR"

# Context indicator — prefer real %, fall back to 200k boolean.
CTX="🟢"
if [[ -n "$USED_PCT" && "$USED_PCT" != "null" ]]; then
  PCT_INT=$(printf '%.0f' "$USED_PCT" 2>/dev/null || echo "")
  if [[ "$PCT_INT" =~ ^[0-9]+$ ]]; then
    if   (( PCT_INT >= 85 )); then CTX="🔴 ${PCT_INT}%"
    elif (( PCT_INT >= 65 )); then CTX="🟡 ${PCT_INT}%"
    else                           CTX="🟢 ${PCT_INT}%"
    fi
  fi
elif [[ "$OVER" == "true" ]]; then
  CTX="🔴 >200k"
fi

COST_STR=""
[[ -n "$COST" ]] && COST_STR=$(printf '$%.3f' "$COST" 2>/dev/null || echo "\$$COST")

printf '%s | %s | %s' "${MODEL:-?}" "$BASE" "$CTX"
[[ -n "$COST_STR" ]] && printf ' | %s' "$COST_STR"
```

### Installation

```bash
chmod +x ~/.claude/hooks/statusline.sh
```

Then in `~/.claude/settings.json`:

```json
{
  "statusLine": {
    "type": "command",
    "command": "bash $HOME/.claude/hooks/statusline.sh"
  }
}
```

Restart your terminal. Every Claude Code session will now show:

```
Opus 4.7 | your-project | 🟢 32% | $0.470
```

Thresholds: green 0–64%, yellow 65–84%, red 85%+. Tune in the script by editing the two `if (( PCT_INT >= X ))` lines.

### Windows PowerShell variant

If you're on Windows without WSL, translate the script to PowerShell. The key moves:

```powershell
# ~/.claude/hooks/statusline.ps1
$json = [Console]::In.ReadToEnd() | ConvertFrom-Json
$model = $json.model.display_name
$dir = Split-Path -Leaf $json.workspace.current_dir
$pct = [int]$json.context_window.used_percentage
$cost = $json.cost.total_cost_usd

$ctx = if ($pct -ge 85) { "🔴 $pct%" } elseif ($pct -ge 65) { "🟡 $pct%" } else { "🟢 $pct%" }
"$model | $dir | $ctx | `$$([math]::Round($cost,3))"
```

Register in `settings.json`:

```json
{
  "statusLine": {
    "type": "command",
    "command": "pwsh -File $HOME/.claude/hooks/statusline.ps1"
  }
}
```

**Note:** use `pwsh.exe` (PowerShell 7), not `powershell.exe` (PowerShell 5.1) — 5.1 has tokenizer edge cases that break some scripts.

---

## Appendix B — The Stop-hook script

Drop this at `~/.claude/hooks/response-protocol.sh`. The full script:

```bash
#!/usr/bin/env bash
# Stop hook — Layer A enforcement of the Response Protocol.
# Reads the session transcript, inspects the last assistant message, and if the
# canonical footer is missing, emits a reminder that Claude Code will surface as
# additional context on the next turn.
#
# Non-blocking: never exits non-zero, never mutates the transcript. Just nudges.

set -u
LOG="$HOME/.claude/response-protocol.log"
mkdir -p "$(dirname "$LOG")"

TS="$(date -Iseconds)"
SID="${CLAUDE_SESSION_ID:-unknown}"

HOOK_PAYLOAD="$(cat)"

TRANSCRIPT=""
if command -v jq >/dev/null 2>&1; then
  TRANSCRIPT="$(echo "$HOOK_PAYLOAD" | jq -r '.transcript_path // empty' 2>/dev/null)"
fi

log() { printf '%s\tresponse-protocol\t%s\t%s\n' "$TS" "$SID" "$1" >> "$LOG"; }

if [[ -z "$TRANSCRIPT" || ! -f "$TRANSCRIPT" ]]; then
  log "no-transcript"
  exit 0
fi

LAST_ASSISTANT=""
if command -v jq >/dev/null 2>&1; then
  LAST_ASSISTANT="$(tac "$TRANSCRIPT" 2>/dev/null | while IFS= read -r line; do
    role="$(echo "$line" | jq -r '.role // .message.role // empty' 2>/dev/null)"
    if [[ "$role" == "assistant" ]]; then
      echo "$line" | jq -r '..|.text? // empty' 2>/dev/null | tr '\n' ' '
      break
    fi
  done)"
fi

# Short responses are exempt
LEN=${#LAST_ASSISTANT}
if [[ $LEN -lt 400 ]]; then
  log "short-exempt:$LEN"
  exit 0
fi

HAS_SCORE=0; HAS_PATHS=0; HAS_UNDER_HOOD=0; HAS_CRITIC=0

echo "$LAST_ASSISTANT" | grep -Eiq 'Score[[:space:]]*:[[:space:]]*[1-5][[:space:]]*/[[:space:]]*5' && HAS_SCORE=1

if echo "$LAST_ASSISTANT" | grep -Eiq '(More scalable paths|Scalable paths)[[:space:]]*:?' \
   && echo "$LAST_ASSISTANT" | grep -Eq '\[(EXECUTE|DEFER|REJECT|PROPOSE)'; then
  HAS_PATHS=1
fi

echo "$LAST_ASSISTANT" | grep -Eiq '###[[:space:]]+Under the hood' && HAS_UNDER_HOOD=1
echo "$LAST_ASSISTANT" | grep -Eiq '###[[:space:]]+Harshest critic' && HAS_CRITIC=1

if [[ $HAS_SCORE -eq 1 && $HAS_PATHS -eq 1 && $HAS_UNDER_HOOD -eq 1 && $HAS_CRITIC -eq 1 ]]; then
  log "ok"
  exit 0
fi

MISSING=""
[[ $HAS_UNDER_HOOD -eq 0 ]] && MISSING+="'### Under the hood' section "
[[ $HAS_CRITIC -eq 0 ]] && MISSING+="'### Harshest critic' section "
[[ $HAS_SCORE -eq 0 ]] && MISSING+="'### Score' X/5 footer "
[[ $HAS_PATHS -eq 0 ]] && MISSING+="'### Scalable paths' tagged bullets "

log "missing:${MISSING// /_}"
cat <<REMINDER

🧭 [response-protocol] Your last response omitted: ${MISSING}.
Per the response protocol, every response ≥400 chars MUST end with:

  ### Under the hood
    (Teach the *why* behind this turn's non-obvious choices.)

  ### Harshest critic
    (Name specific weaknesses with a counterfactual: "if X instead of Y, outcome Z.")

  ### Score
    Score: X/5 — <one-line improvement note>

  ### Scalable paths
    - [EXECUTE] <did it this turn>
    - [DEFER: <named unblocker>] <bullet>
    - [REJECT: <reason>] <bullet>
    - [PROPOSE: <reason>] <bullet>

Apply this to your next response automatically.
REMINDER
exit 0
```

### Installation

```bash
chmod +x ~/.claude/hooks/response-protocol.sh
```

Add to `~/.claude/settings.json`:

```json
{
  "hooks": {
    "Stop": [
      {
        "matcher": "",
        "hooks": [
          { "type": "command", "command": "bash $HOME/.claude/hooks/response-protocol.sh" }
        ]
      }
    ]
  }
}
```

### What the script does NOT do

- It doesn't grade the quality of your `Under the hood` or `Harshest critic` sections. You have to push back on weak writing yourself.
- It doesn't enforce word limits. The 100-word cap on sections 1 and 2 is a discipline, not a machine-enforced rule.
- It doesn't require the optional `### Capabilities` block — but if the block is present with `[N/A]` rows, you'll want to extend the script to flag that (see the canonical version in my workspace for the full validation, or keep this minimal version and eyeball it yourself).

---

## Closing

You bought this because you hit a wall that shouldn't exist. It doesn't have to.

The audit takes 90 seconds. The MCP prune takes 5 minutes. The protocol pays for itself within an hour. The binary-grep pattern is a sixty-second rule you can apply to any CLI tool for the rest of your career.

If something here doesn't match your experience, grep the binary. Don't email me until you've done the grep — nine times out of ten, the answer is in there and you'll catch a thing the docs got wrong.

If the grep shows something the docs are actively lying about, that's worth an email. Send it to `fahad.mnoor97@gmail.com` with the exact file path, line number, and the claim it contradicts.

— F
