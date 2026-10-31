---
name: tmux
description: >-
  Control interactive terminal programs with tmux, using a shared socket and a
  separate tmux session for each agent session.
---

# tmux

Use tmux for programs that need an interactive terminal. Prefer the execution
tool's background mode for long-running, non-interactive jobs.

## Prerequisite

Check availability with `command -v tmux`. If missing, ask the user to install
tmux in the current environment. Do not install it yourself; continue tmux work
once it is available.

## Shared socket

Use `tmux -L agent` for every server operation. All agents share this named
socket under the same OS user. Keep any `TMUX_TMPDIR` setting consistent across
agents and resumed shells; let tmux manage the socket directory.

## Session ownership and naming

- Give each agent session, including subagents, its own tmux session.
- Choose a short name describing the task, such as `fix-login` or `debug-parser`.
  Use lowercase letters, digits, and hyphens without spaces, not random strings.
- If another agent owns the name, add a meaningful role or numeric suffix,
  such as `fix-login-tests` or `fix-login-2`.
- Retain the session name and pane IDs in the task context for later calls.
  Reuse the recorded session when resuming; a matching task name alone does
  not establish ownership.
- Target the owned session or pane explicitly. Keep one writer per pane and
  separate windows for independently owned sessions. Attaching does not scope
  later commands to that session.

For a new session, export the chosen task name in the controlling shell, then
create it. Replace `fix-login` with the name selected for the current task:

```bash
export SESSION='fix-login'
tmux -L agent new-session -d -s "$SESSION"
```

`-d` creates the session detached so the tool call can return. Restore the same
`SESSION` value in each new shell; `export` does not persist across independent
tool calls. When resuming an owned session, skip creation.

To attach from an interactive terminal:

```bash
tmux -L agent attach-session -t "=$SESSION"
```

The `=` prefix matches the session name exactly. Attaching is optional for
tool-driven operations.

## Cleanup

Keep sessions needed for ongoing work or user handoff. Once your own session
is no longer needed, remove it using the same recorded `SESSION` name:

```bash
tmux -L agent kill-session -t "=$SESSION"
```

Do not use `kill-server`, bulk session deletion, or socket-file removal for
routine cleanup. Leave other agents' sessions and shared server settings alone.
