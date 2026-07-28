---
name: tmux-operator
description: Run all CLI commands in a visible persistent tmux split, or operate tmux sessions, windows, panes, and terminal workers. Use immediately when asked to set up or run commands in a split, pane, terminal next to the user, or when asked to inspect or change tmux.
---

# tmux Operator

Use tmux as a visible, persistent execution layer that the user can inspect or take over.

## Rules

- Inspect only the scope needed for the request. Pane output may contain sensitive data.
- Target stable IDs: `$1` for sessions, `@3` for windows, and `%48` for panes. Names and indexes are labels, not reliable targets.
- Preserve the target cwd and focus unless the user requests otherwise. Use `-d` when creating or moving panes and windows.
- For one known process, launch it as part of `split-window` or `new-window`. For a task whose CLI workflow must remain visible, use Visible CLI Mode instead.
- Reuse a matching healthy process unless the user explicitly requests a new instance, pane, or window.
- Verify the pane's command and cwd immediately before sending input.
- Never send keys to the current `$TMUX_PANE` during the turn.
- Prefer an application's CLI, API, or socket over simulated keystrokes. Use `send-keys -l` for literal text and send control keys separately.
- Do not kill, interrupt, respawn, detach, or replace anything unless explicitly requested.
- Leave created tmux objects available for inspection.

## Visible CLI Mode

When the user asks to set up a program or run all CLI commands in a split, pane, or visible terminal:

1. Call `visible_terminal` with `action=start` before any Bash command, including inspection commands.
2. Continue using the normal Bash tool. The plugin routes every Bash call for this OpenCode session through the persistent shell pane.
3. Do not create the split manually or use `send-keys` for routed commands.
4. Keep foreground commands attached when they need user interaction. Run a persistent server in the background only when later shell commands must use the same pane.
5. At the end of the task, call `visible_terminal` with `action=release`. This stops routing but leaves the pane and shell open.

If activation fails or the tool is unavailable, stop before running Bash and report the error. Do not silently fall back to hidden command execution.

## Inspect

Start with the current pane:

```bash
tmux display-message -p -t "${TMUX_PANE:?not inside tmux}" 'host=#{host} socket=#{socket_path} session=#{session_id} window=#{window_id} pane=#{pane_id} command=#{pane_current_command} path=#{pane_current_path}'
```

If `$TMUX_PANE` is absent, do not create a detached server unless the user explicitly requested server inventory, session creation, or attachment.

Expand from one window to one session, then the server only as needed:

```bash
tmux list-panes -t '@3' -F 'session=#{session_id} window=#{window_id} pane=#{pane_id} active=#{pane_active} dead=#{pane_dead} command=#{pane_current_command} pid=#{pane_pid} path=#{pane_current_path} state=#{@opencode_state} agent_session=#{@opencode_session_id}'
tmux list-panes -s -t '$1' -F 'session=#{session_id} window=#{window_id} pane=#{pane_id} active=#{pane_active} dead=#{pane_dead} command=#{pane_current_command} pid=#{pane_pid} path=#{pane_current_path} state=#{@opencode_state} agent_session=#{@opencode_session_id}'
tmux list-panes -a -F 'session=#{session_id} window=#{window_id} pane=#{pane_id} active=#{pane_active} dead=#{pane_dead} command=#{pane_current_command} pid=#{pane_pid} path=#{pane_current_path} state=#{@opencode_state} agent_session=#{@opencode_session_id}'
```

Use `list-sessions`, `list-windows -a`, or `list-clients` only for session, window, or client operations. Ask when more than one object matches.

Capture bounded output from one exact pane and increase the range only when necessary:

```bash
tmux capture-pane -p -t '%48' -S -200
```

## Create and Change

Create a pane or window with its program, retain the returned ID, then verify that target:

```bash
pane_id=$(tmux split-window -h -d -P -F '#{pane_id}' -t '%48' -c '/absolute/cwd' command_name arg1 arg2)
window_and_pane=$(tmux new-window -d -P -F '#{window_id} #{pane_id}' -t '$1' -n 'label' -c '/absolute/cwd' command_name arg1 arg2)
tmux display-message -p -t "$pane_id" 'pane=#{pane_id} command=#{pane_current_command} path=#{pane_current_path}'
```

Use `-h` for a pane on the right and `-v` for one below. Set requested sizing during creation with `-p` for a percentage or `-l` for an exact size. Create an empty pane only when later terminal interaction is the task.

Apply layout changes directly to stable IDs. Preserve focus with `-d` where supported:

```bash
tmux resize-pane -t '%49' -R 5
tmux swap-pane -d -s '%49' -t '%50'
tmux join-pane -d -h -s '%50' -t '%49'
tmux break-pane -d -P -F '#{window_id}' -s '%50'
tmux move-window -s '@4' -t '$1:2'
tmux select-layout -t '@3' even-horizontal
```

For a one-shot command whose exact status matters, enable `remain-on-exit` in that pane and read `pane_dead_status` after `pane_dead=1`.

## Interact and Monitor

Before sending input, verify the exact pane, cwd, process, and current screen. A shell in `pane_current_command` does not prove that no child TUI owns the terminal.

```bash
tmux send-keys -t '%57' -l -- "$text" \; send-keys -t '%57' '<submit-key>'
```

Do not guess TUI keybindings or treat an immediate capture as completion. For Hunk, load `hunk-review` and use its CLI rather than driving the TUI.

Use an explicit completion signal:

- One-shot command: process exit and exit status.
- Server or watcher: requested readiness or health output.
- Test watcher: latest completed cycle, not process exit.
- Interactive application: application-specific ready, blocked, and idle states.

Poll only while a requested readiness or completion condition is outstanding. On timeout, report the process state and recent output without interrupting it.

## OpenCode Workers

OpenCode reports authoritative pane-local metadata through the global `tmux.js` plugin:

- `@opencode_state`: `working`, `blocked`, or `idle`.
- `@opencode_session_id`: the root OpenCode session in that pane.

The plugin ignores normal subagent lifecycle events so they cannot overwrite the root state. A subagent permission or question still marks the pane `blocked` until answered, matching the herdr integration.

Launch OpenCode directly in the new pane. Use `--prompt` when the initial prompt is known, or `opencode run` when no persistent TUI is needed:

```bash
pane_id=$(tmux split-window -h -d -P -F '#{pane_id}' -t '%48' -c '/absolute/cwd' opencode --prompt "$prompt")
```

Read state without parsing the TUI:

```bash
tmux display-message -p -t "$pane_id" 'pane=#{pane_id} command=#{pane_current_command} path=#{pane_current_path} state=#{@opencode_state} session=#{@opencode_session_id}'
```

Treat `working` as active, `blocked` as waiting for user input, and `idle` as response completion. If blocked, capture the exact request but do not answer it automatically. An empty state means the pane is not instrumented; it does not mean idle.

For an existing OpenCode TUI, submit literal prompt text with the configured submit key. This setup uses `Ctrl+Y`:

```bash
tmux send-keys -t '%57' -l -- "$prompt" \; send-keys -t '%57' C-y
```

After submission, wait on `@opencode_state` rather than screen stability. Capture output only for the final response, a blocked request, or an error.
