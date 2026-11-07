---
name: divide-and-conquer
description: Plans and divides a large, complex task into sequential subtasks sized for limited-budget sessions, executes them one at a time, and writes a self-contained handoff file after each so the next session can continue cold. Use when the user gives a large or multi-session task and says "break this down", "plan and hand off", or "continue from HANDOFF", or provides a HANDOFF_NN.md or PLAN.md file.
metadata:
  version: "1.2"
---

# Divide and Conquer

Break a large task into subtasks that each fit comfortably in one session, run them one at a time, and leave a handoff file so the next session (possibly with zero memory of this one) can continue.

*In Relay mode, you do not break down and plan the problem. Focus solely on execution.*

## Choose a mode

Decide the mode from what the user provided:

- HANDOFF_NN.md provided: **Relay mode**.
- Fresh task, no handoff: **Plan mode**.
- Both a handoff and an unrelated new task: ask which one to do. DO NOT mix them.
- Neither (unclear request): ask what the user wants.

If the task would fit comfortably in a single session, say so and just do it. Do not decompose small tasks.

## Workflow

### Plan Mode

1. If the files needed to plan are not visible, ask for them first. Do not guess the file structure.
2. Produce a plan and display it to the user. DO NOT start implementing it. Ask the user for approval or edits first. Make necessary edits, and reprompt the user until approval is obtained. Proceed only after explicit approval (Do not consider "I guess", "What if we...", "maybe" etc. as explicit approval).
3. The plan must contain:
	- **Requirement map:** every user requirement, each mapped to the subtask(s) that satisfy it. Nothing dropped.
	- **Subtasks in order**, each with: goal, size tag (S/M/L), files it reads, files it modifies/creates, what it assumes from earlier subtasks, and a completion condition.
	- **Ordering rationale:** one line on why this order.
4. Sizing heuristics:
	- Assume a limited budget unless told otherwise.
	- One coherent deliverable per subtask.
	- Roughly 5 or fewer files touched, and a modest amount of new code per subtask.
	- If a subtask is L, split it further and update the plan.
5. After approval, save the plan as `PLAN.md`, then execute subtask 1 (see Execution), and emit `HANDOFF_02.md`.

### Relay mode

DO NOT re-plan. Execute only the subtask described in the provided handoff.

1. Read the handoff. If it references files that were not provided, ask for them before starting.
2. Execute the subtask (see Execution).
3. If this is not the final subtask, provide the next handoff to the user as a clean, formatted markdown. If it is the final subtask, emit a short completion summary instead and state that the task is complete.

### Execution (both modes)

- Read and modify ONLY the files the handoff or plan lists. If you discover another file that is needed, stop and ask the user. Do not silently expand scope. If the deviation is minor and you proceed, record it in the next handoff, and alert the user in chat after completion.
- Before handing off, verify the completion condition (run the tests, check the output against acceptance criteria, or sanity-check by inspection). Clearly state what you verified and what you could not.
- If the subtask is turning out too large to finish, stop at a clean point and provide a partial handoff file covering what is done and what remains. Clearly instruct the user on any extra instructions within chat. Do not truncate silently.

## Output format

Every subtask ends in the following output, in order:

1. **Changed files:** every (and only those) file created or modified in this subtask, in full.
2. **Handoff file:** `HANDOFF_NN.md` (NN = number of the next subtask). When writing any handoff file, first read `assets/handoff-template.md` and follow its structure exactly. Keep the result under 300 words.
3. **Chat message to the user:** which files to upload to the next session. The changed files from this subtask that the next subtask needs, plus the handoff, plus `PLAN.md` that details the overall plan. Keep it concise.

## Rules

1. **Keep it concise.** Include everything a session with no memory needs to start, and nothing else. Reason: a handoff that is short but missing context fails; and one that is padded wastes the next session's budget.
2. **Scope file access.** Each subtask reads and modifies only what it lists. Reason: unlisted files bloat context and risk unintended changes.
3. **Handoff files never address the user.** User instructions go in the chat message only. Reason: the handoff is read by the next model session as its instructions, and user-directed text confuses it.
4. **Never drop a requirement.** If something cannot fit, say so in the plan and let the user decide.
5. **Confirm before building.** Plan mode never implements until the user approves.
