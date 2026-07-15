---
name: tmux-operator
description: Operate and reuse tmux sessions, windows, panes, and terminal processes. Use when asked to inspect or capture tmux state, create terminal workspaces, reuse or launch dev servers, run or monitor tests, watchers, or logs, interact with terminal applications, or launch visible workers such as OpenCode instances.
---

# tmux Operator

Use tmux as a shared execution and observation layer. Prefer visible, recoverable processes that the user can inspect or take over.

## Operating Contract

- Inspect before acting.
- Reuse a matching process in the project's existing tmux panes before creating another server, watcher, test process, log stream, or worker.
- Use the shortest safe path. When creating a pane or window for a known command, launch the command as part of creation instead of creating a shell and typing into it afterward.
- Keep user-visible terminal output clean. Do not print completion markers or agent bookkeeping into panes by default.
- Target stable tmux IDs such as `$1` for sessions, `@3` for windows, and `%48` for panes. Never rely on names or indexes because they may be duplicated or renumbered.
- Preserve the target pane's working directory unless the user provides another directory.
- Use executable names rather than interactive shell aliases.
- Never send keys to the current `$TMUX_PANE` while this turn is running.
- Verify the target command and cwd immediately before sending input.
- Do not capture unrelated panes speculatively. Pane output may contain sensitive data.
- Do not kill panes, windows, or sessions, respawn panes, send interrupts, detach clients, or replace running processes unless explicitly requested.
- Leave created tmux objects available for inspection when work completes.

Prefer an application's dedicated control CLI or API over simulated keystrokes. Use `tmux send-keys` when terminal input is the intended interface or no better control surface exists.

## Inspect tmux

For ordinary in-session operations, use one command to verify and record the current host, server socket, tmux IDs, application, and cwd:

```bash
tmux display-message -p -t "${TMUX_PANE:?not inside tmux}" 'host=#{host} socket=#{socket_path} session=#{session_id} name=#{session_name} window=#{window_id}:#{window_index} pane=#{pane_id}:#{pane_index} command=#{pane_current_command} path=#{pane_current_path}'
```

tmux IDs identify objects only within the reported server on the reported host. If this check fails, diagnose the environment without creating anything:

```bash
tmux -V
printenv TMUX
printenv TMUX_PANE
```

If `$TMUX_PANE` is absent, pane-local operations and client switching are unavailable. Continue only when the user explicitly requested server inventory, session creation, or attachment; otherwise stop with a clear error. Do not create a detached tmux server as an implicit workaround.

Expand only to the scope needed for the request. List one window's panes first; use session-wide or server-wide inventory only when the target cannot be resolved more narrowly:

```bash
tmux list-panes -t '@3' -F 'session=#{session_id} name=#{session_name} window=#{window_id}:#{window_index} pane=#{pane_id}:#{pane_index} active=#{pane_active} dead=#{pane_dead} command=#{pane_current_command} pid=#{pane_pid} path=#{pane_current_path} title=#{pane_title} tty=#{pane_tty}'
tmux list-panes -s -t '$1' -F 'session=#{session_id} name=#{session_name} window=#{window_id}:#{window_index} pane=#{pane_id}:#{pane_index} active=#{pane_active} dead=#{pane_dead} command=#{pane_current_command} pid=#{pane_pid} path=#{pane_current_path} title=#{pane_title} tty=#{pane_tty}'
tmux list-panes -a -F 'session=#{session_id} name=#{session_name} window=#{window_id}:#{window_index} pane=#{pane_id}:#{pane_index} active=#{pane_active} dead=#{pane_dead} command=#{pane_current_command} pid=#{pane_pid} path=#{pane_current_path} title=#{pane_title} tty=#{pane_tty}'
```

Inventory sessions and windows only for workspace-level operations:

```bash
tmux list-sessions -F 'session=#{session_id} name=#{session_name} attached=#{session_attached} windows=#{session_windows}'
tmux list-windows -a -F 'session=#{session_id} window=#{window_id}:#{window_index} name=#{window_name} active=#{window_active} panes=#{window_panes}'
```

List clients only when the request concerns attached clients or detaching:

```bash
tmux list-clients -F 'client=#{client_name} session=#{session_id} tty=#{client_tty}'
```

Use the exact stable ID returned by these commands for every later operation. Names are labels for the user, not control identifiers. If more than one session, window, pane, or client could match the request, ask the user to choose.

Treat `pane_current_command` as the primary application signal, but a shell value does not prove the pane is idle: a child TUI may still control the terminal beneath a wrapper shell. If the pane previously ran a TUI or its capture shows recognizable application chrome, inspect that pane's process tree using its tty before sending shell input. Never identify an application from the pane title alone.

## Capture Output

Capture bounded recent history from one exact pane:

```bash
tmux capture-pane -p -t '%48' -S -200
```

Increase the history range only when the requested output is missing. Do not default to the entire scrollback buffer.

Capture before interacting with an existing process when its current state matters or the new output would otherwise be ambiguous. Skip the pre-capture for a recently verified idle shell running a trivial command because the echoed command identifies the output boundary. A new pane that launches its command atomically also does not need an empty pre-capture. Summarize only content relevant to the request, preserving exact errors, paths, commands, and exit statuses when they matter.

## Create Windows and Panes

Resolve the source pane and cwd first. When the command is known, launch it atomically and leave a shell available after it exits. This is the default path for a new pane, but retaining the shell forfeits the launched command's exact exit status:

```bash
tmux split-window -h -d -P -F '#{pane_id}' -t '%48' -c '/absolute/cwd' 'command_here; exec "${SHELL:-/bin/sh}"'
```

Use `-h` for a pane to the right and `-v` for a pane below. The final argument is shell source; shell-quote every interpolated path and argument. For a persistent process, the same form leaves it running and opens the shell only after it exits.

Capture the new pane ID from stdout, then take one immediate output snapshot. Monitor further only when the task has an explicit completion or readiness condition that is not yet visible.

Create an empty pane only when the user requested one or when later terminal interaction is itself the task:

```bash
tmux split-window -h -d -P -F '#{pane_id}' -t '%48' -c '/absolute/cwd'
```

Reinspect an empty pane before sending input to it.

Use a separate window for independent or background work that does not need to remain visible beside the source pane:

```bash
tmux new-window -d -P -F '#{window_id} #{pane_id}' -t '$1:' -n '<label>' -c '/absolute/cwd' 'command_here; exec "${SHELL:-/bin/sh}"'
```

Capture both IDs from stdout, then reinspect the window and its initial pane. Use names only as human-readable labels.

For an explicit exact-status requirement, keep the finished pane as a dead pane and read `pane_dead_status` using the returned ID:

```bash
pane_id=$(tmux split-window -h -d -P -F '#{pane_id}' -t '%48' -c '/absolute/cwd' 'tmux set-option -p -t "$TMUX_PANE" remain-on-exit on; exec command_here')
tmux display-message -p -t "$pane_id" 'dead=#{pane_dead} status=#{pane_dead_status}'
```

Poll the metadata only until `dead=1`; status is blank while the command is still running.

## Manage Sessions and Clients

Create a session only when explicitly requested. Preserve the requested cwd and capture all returned IDs:

```bash
tmux new-session -d -P -F '#{session_id} #{window_id} #{pane_id}' -s '<name>' -c '/absolute/cwd'
```

Do not treat a detached session as an isolation boundary. It shares the host, filesystem, credentials, network, and other resources available to its processes.

When the user explicitly requests moving a client between sessions, use the operation appropriate to the current context:

```bash
# From inside tmux
tmux switch-client -t '$1'

# From a terminal outside tmux
tmux attach-session -t '$1'
```

Before detaching, identify the exact client with `list-clients`. Detach only that client and only when explicitly requested:

```bash
tmux detach-client -t '/dev/ttys001'
```

tmux preserves sessions, windows, panes, and running processes when a client disconnects. It does not preserve a process that exits, and it does not by itself restore state after the host or tmux server restarts.

## Send Input to Existing Panes

Reserve `send-keys` for an existing pane or an interactive application. Do not create an empty pane and use `send-keys` when the command can be supplied directly to `split-window` or `new-window`.

For a pane recently verified as an idle `zsh`, `bash`, `fish`, or `sh`, run a trivial command and take a best-effort snapshot in one tmux invocation. `run-shell 'true'` yields the command queue once without a fixed delay:

```bash
tmux send-keys -t '%57' -l -- 'pwd' \; send-keys -t '%57' Enter \; run-shell 'true' \; capture-pane -p -t '%57' -S -20
```

The snapshot is not completion detection. A returned prompt shows that the shell accepts input again, but background work may still be running; missing output is inconclusive. Reinspect first only when the pane was not recently verified, may have changed applications, or is not clearly idle. Quote paths and user-provided arguments defensively. Do not inject a command into a pane running an editor, TUI, server, test process, or unknown application.

## Monitor Processes

Before starting a persistent process, look for one already running in the target project:

1. Search the current window, then the current session, expanding further only when the project may be elsewhere.
2. Match candidates by cwd and command or process tree; capture output only from likely candidates.
3. Reuse a matching healthy process as the observation and control surface, even when another user or agent created it.
4. Start another process only when no match exists, the existing process is unhealthy or configured differently, or the user explicitly requests a separate instance.

Do not infer identity from a listening port or pane title alone. Verify the pane cwd, process, and relevant readiness output.

Classify the process before waiting:

- **One-shot command:** Use an explicit completion condition, a dead pane with `pane_dead_status`, or an application-specific completion signal.
- **Server or watcher:** Wait for a specific readiness message, listening address, or healthy state; then leave it running.
- **Test watcher:** Capture the latest completed test cycle and distinguish it from the process continuing to watch.
- **Log stream:** Capture only the requested interval or event and leave the stream running.
- **Interactive application:** Use application-specific readiness and completion signals.

Capture immediately after launch. If the completion or readiness signal is absent, poll every 2-5 seconds unless the process has a naturally slower feedback cycle. Use a five-minute timeout unless the task implies another duration.

On timeout, report the current pane command and recent output. Do not interrupt the process automatically. If the pane exits, becomes dead, or changes cwd unexpectedly, stop and report the observed state.

## Interact With Terminal Applications

Before sending input to a TUI:

1. Verify the exact pane and application.
2. Capture its current state.
3. Determine how that application accepts text and submits actions.
4. Send literal text with `send-keys -l`.
5. Send control or submit keys separately.
6. Capture output to verify the action occurred.

Do not assume Enter submits across applications. Do not navigate or mutate an unfamiliar TUI by guessing keybindings.

When a dedicated skill or control CLI exists, use it instead:

- For Hunk, load `hunk-review` and use `hunk session *`; do not drive the Hunk TUI through tmux keystrokes.
- For other applications, prefer their documented session, RPC, socket, or remote-control interface when available.

## OpenCode Adapter

OpenCode may run beneath a wrapper shell, so `pane_current_command` can report `zsh` or another shell while the TUI remains active. Treat a previously known OpenCode pane or recognizable OpenCode TUI capture as OpenCode, then confirm with that pane's process tree when needed. Do not rely on the title alone.

Before launching another TUI, inventory existing panes. Reuse an OpenCode pane only when its cwd matches the requested project and reusing its conversation is appropriate.

In a new pane or window, launch OpenCode directly as the creation command. Launch it in an existing pane only after a capture and process check establish that the pane is an idle shell with no child TUI:

```bash
tmux send-keys -t '%57' -l -- 'opencode'
tmux send-keys -t '%57' Enter
```

Poll pane metadata until `pane_current_command` is `opencode`, then capture output to confirm the TUI is ready. If startup fails, report the pane output instead of retrying blindly.

OpenCode can be launched with explicit options when requested:

```bash
opencode --agent explore
opencode --session '<session-id>'
opencode --continue
```

Do not invent a session ID. Resolve sessions with `opencode session list --format json` when needed.

### Send a Prompt

This setup submits OpenCode input with `Ctrl+Y`. Return inserts a newline. Use the known submit binding directly; do not reread `tui.jsonc` on every prompt.

Send prompt text literally, then submit separately:

```bash
tmux send-keys -t '%57' -l -- 'Prompt text'
tmux send-keys -t '%57' C-y
```

For multiline prompts, send the full text literally and submit once with `C-y` at the end.

Read the active `tui.json` or `tui.jsonc` only when `C-y` does not submit, the user says the binding changed, or the target instance is known to use different configuration. Prefer an explicit `OPENCODE_TUI_CONFIG`, then project TUI config, then global TUI config. Do not assume Enter submits.

### Wait for a Response

Capture the pane before submission. After submitting:

1. Confirm output changed or OpenCode entered its active state.
2. Continue polling while the TUI shows an active indicator such as `esc interrupt`.
3. If OpenCode requests approval, permission, or other user input, classify the worker as blocked and report the exact request. Do not approve it automatically.
4. Treat the response as complete when the active indicator disappears, the input area is ready, and output is stable across two captures.
5. On timeout, report that OpenCode remains active and include recent output without interrupting it.

## Visible Worker Delegation

Use a separate tmux pane or window when the user wants visibility, persistence across client disconnects, manual takeover, or an independent long-running process. Use a pane for tightly related side-by-side work and a window for an independent background worker or a separate layout. A worker may be an OpenCode instance, another terminal agent, a test runner, a server, or a purpose-specific CLI.

Before creating a worker, define its task, cwd and allowed scope, completion or readiness condition, verification, and expected handoff. Track its hostname, server socket, stable tmux IDs, application, purpose, and state. Check for duplicates and resource contention first.

Coding workers sharing the current checkout must be read-only or explicitly limited to non-overlapping files. Do not run simultaneous write-capable workers in the same files. This skill does not create Git worktrees.

Report blocked or stale workers without answering prompts or cleaning them up automatically. Do not allow recursive workers, commits, pushes, publishing, deployment, or other external actions unless explicitly requested.

When a worker finishes, capture its result, verify any claims in the parent context, and leave its tmux objects intact.

## Common Failures

- **Not inside tmux:** For pane-local operations, report the missing tmux context. Continue only for an explicitly requested server inventory, session creation, or attachment; do not silently create a server.
- **Unknown or wrong target:** Re-list tmux objects and stop when the stable ID, hostname, or server socket cannot be resolved exactly.
- **Target is the current pane:** Do not inject keys into the running process.
- **Wrong application:** Stop rather than sending shell input to a TUI or TUI input to a shell.
- **Operation did not complete:** Report the command, expected condition, process state, and recent output without interrupting or killing it.
