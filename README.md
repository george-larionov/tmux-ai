# tmux-ai

Control tmux in plain English. Press a key, type what you want ("combine panes 2 and 3 vertically", "move this pane to a new window", "undo that"), review the tmux commands Claude proposes, and run them.

```
tmux> combine panes 2 and 3
# Stack them top/bottom or side by side?
reply> side by side
join-pane -h -s %3 -t %2

Run? [Y/n/e(dit)/r(eply)]
```

Unlike AI terminal assistants that run shell commands, tmux-ai manages **tmux itself**: panes, windows, layouts and sessions. Nothing runs until you confirm.

## Requirements

- tmux 3.2+ (for `display-popup`)
- bash 4+. macOS ships bash 3.2, so run `brew install bash` there.
- `curl` and `jq`
- An [Anthropic API key](https://console.anthropic.com/). This is separate from a Claude Pro/Max subscription, and a request costs a fraction of a cent with the default model.

## Install

1. **Clone the repo** (anywhere; `~/src` is used below):

   ```sh
   git clone https://github.com/<you>/tmux-ai.git ~/src/tmux-ai
   ```

2. **Store your API key.** Open the file in an editor so the key stays out of your shell history:

   ```sh
   mkdir -p ~/.config/anthropic && chmod 700 ~/.config/anthropic
   ${EDITOR:-vi} ~/.config/anthropic/api_key   # paste the key, save
   chmod 600 ~/.config/anthropic/api_key
   ```

   tmux-ai reads `$ANTHROPIC_API_KEY` first and falls back to this file. Optionally, export the variable for other tools by adding this to `~/.zshrc` or `~/.bashrc`:

   ```sh
   [ -r ~/.config/anthropic/api_key ] && export ANTHROPIC_API_KEY="$(< ~/.config/anthropic/api_key)"
   ```

3. **Bind a key** in `~/.tmux.conf`:

   ```tmux
   bind a display-popup -E -w 70% -h 40% "$HOME/src/tmux-ai/tmux-ai '#{pane_id}'"
   ```

   Then reload tmux with `prefix` + `:` and `source-file ~/.tmux.conf`.

## Usage

Press `prefix` + `a`, type a request, and press Enter.

| Key | Where | Action |
|---|---|---|
| ↑ / ↓ | prompt | Recall past requests |
| Enter / `y` | Run? | Run the commands |
| `n` | Run? | Cancel |
| `e` | Run? | Edit the commands in `$EDITOR`, then run |
| `r` | Run? | Reply with a correction ("no, the other way") |
| Esc | anywhere | Close the popup (also cancels a pending request) |

If Claude needs clarification, it asks a question and you answer at the `reply>` prompt. Each turn sends the whole conversation along with the current layout of every session, window and pane.

The last few exchanges are also included, so follow-ups like "undo that" or "do the same in window 2" work across popups.

## Configuration

Set these environment variables in the shell that starts tmux, or with `set-environment -g` in `tmux.conf`:

| Variable | Default | Meaning |
|---|---|---|
| `TMUX_AI_MODEL` | `claude-haiku-4-5-20251001` | Model to use. Try `claude-sonnet-5` for trickier requests. |
| `TMUX_AI_HISTORY` | `10` | Past exchanges sent as context (`0` to disable) |

## Files

| Path | Contents |
|---|---|
| `~/.local/state/tmux-ai/history` | Your requests, for ↑ recall |
| `~/.local/state/tmux-ai/log.jsonl` | Requests, commands and outcomes, fed back as context |

Both files are trimmed automatically. Delete them anytime to start fresh. `$XDG_STATE_HOME` is respected.

## Troubleshooting

- **The popup flashes and closes:** run `~/src/tmux-ai/tmux-ai` directly inside tmux to see the error. It's usually a missing `jq` or API key.
- **Esc needs two presses:** you're on bash 3.2. Install a newer bash.

## License

MIT
