# Divide and Conquer

A Claude skill that keeps big projects from dying in one long chat. It splits a large task into small sequential stages and writes a compact handoff file after each one, so there is always a checkpoint to restart from, even if a session hits its limit.

## The problem: the chat that can't continue

Long, winding chats on complex projects tend to fail the same way:

1. The conversation grows. Every new message re-sends the entire history, so each request costs more than the last.
2. Eventually the chat is so long that a single request exhausts your usage limit or context window. Pressing **Continue** just re-sends everything and fails again, often in a loop.
3. You're stuck. You can't continue in this chat. Starting a new chat means losing all the context. And you can't ask Claude to write a summary or handoff now, because that request also re-sends the whole conversation and hits the same wall.

The trap is that **the handoff has to be written before the wall, but you only discover the wall when you hit it.** By then it's too late to ask for one.

## The idea: checkpoint early, not at the end

This skill moves the handoff from "emergency rescue" to "routine output of every stage."

- The task is split into small subtasks up front, so no single chat ever needs to grow large.
- Every subtask ends by producing its changed files and a short `HANDOFF_NN.md` automatically, while the chat is still small and cheap.
- The next stage starts in a **fresh chat** with only the handoff and the files it needs. Nothing long is carried forward, so the request size stays small and the loop never forms.

If a chat does die mid-stage, you lose at most one small stage of work. You restart that stage from the previous handoff, instead of losing the whole project's context.

| | One long chat | With this skill |
|---|---|---|
| Request size | Grows with every message | Stays small each stage |
| Context lives in | The chat history | Handoff files you hold |
| When the limit hits | Everything is at risk, no way to summarize | At most one stage is at risk |
| Handoff created | Only on request, often too late | Automatically at every stage |
| Starting over | Re-explain the whole project | Upload the last handoff and files |

## Important limits of this approach

- **It is preventive, not a rescue.** If you are already inside a stuck chat, the skill can't pull a handoff out of it. Use it from the start of a project, or the next time you begin a large task.
- **Salvaging an already-stuck chat is manual.** Copy out whatever final files or decisions you still have, write a short summary yourself, and use it to start a fresh stage with this skill.
- **The model can't see its own budget.** Stages are sized with heuristics (size tags, file limits), not real token counts. When in doubt, split smaller. A few extra handoffs are cheap; an overrun is not.
- **Overruns are handled mid-stage.** If a stage turns out too large, the skill is designed to stop at a clean point and emit a partial handoff rather than push on until the limit hits.

## How it works

The skill has two modes, chosen from what you provide.

**Plan mode** (you give a fresh task)

1. The skill checks it has the files it needs to plan.
2. It produces a plan: a requirement-to-subtask map, ordered subtasks with size tags and done conditions, and the reasoning for the order.
3. It waits for your approval. Nothing is implemented before that.
4. It saves `PLAN.md`, executes subtask 1, and emits `HANDOFF_02.md`.

**Relay mode** (you provide a `HANDOFF_NN.md`)

1. The skill does not re-plan. It executes only the subtask in the handoff.
2. It verifies the done condition, outputs the changed files, and emits the next handoff.
3. On the final subtask it emits a completion summary instead.

If a task is small enough to finish in one session, the skill says so and just does it.

## Usage

Start a task:

```
/divide-and-conquer Build a CLI expense tracker with SQLite storage, CSV export, and tests.
```

Continue in a **new chat** for the next stage:

```
/divide-and-conquer HANDOFF_03.md
```

Upload the files listed in the previous stage's chat message (the changed files, the handoff, and optionally `PLAN.md`). Keep these files somewhere safe between stages. They are your checkpoint.

If your environment doesn't support slash invocation, describe the intent instead, for example "continue from HANDOFF_03.md", and the skill's description should trigger it.

## Installation

**Claude Code:** copy the `divide-and-conquer/` folder into your skills directory.

```bash
# personal (all projects)
cp -r divide-and-conquer ~/.claude/skills/

# or project-only
cp -r divide-and-conquer .claude/skills/
```

**Claude.ai:** zip the `divide-and-conquer/` folder (so the folder itself is at the top level of the zip) and upload it through the skills section of your settings. Menu names change, so check Anthropic's current documentation.

Install the skill in every environment where you plan to run later stages, since relay mode relies on it being loaded.

## Repository layout

```
.
├── README.md
├── LICENSE
└── divide-and-conquer/
    ├── SKILL.md
    └── assets/                    # optional
        └── handoff-template.md
```

Only the `divide-and-conquer/` folder is the skill. The README and license are for the repository and are not needed at install time.

## Handoff file format

Each handoff is capped at about 300 words and contains:

- Position and overall goal ("Subtask N of M")
- What is done so far (decisions, interfaces, conventions)
- This subtask's goal and acceptance criteria
- Files to read, files to modify, and a do-not-touch list
- Known issues or open questions

Handoff files never address the user. Instructions for you (which files to upload next) appear in the chat message.

## Other limitations

- Verification depends on the environment. It is strongest for code with a runnable environment and weaker for writing or research tasks.
- If a plan proves wrong mid-project, return to plan mode with a revised scope.
- This skill has had limited testing. Feedback from real multi-session runs is welcome.
