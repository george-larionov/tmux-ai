# tmux-ai

Control tmux in plain English. Press a key, type what you want ("combine panes 2 and 3 vertically", "move this pane to a new window", "undo that"), review the tmux commands Claude proposes, and run them.

```
tmux> combine panes 2 and 3
# Stack them top/bottom or side by side?
reply> side by side
join-pane -h -s %3 -t %2

Run? [y/n/e(dit)/r(eply)]
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
| `y` | Run? | Run the commands (other keys are ignored) |
| `n` | Run? | Cancel |
| `e` | Run? | Edit the commands in `$EDITOR`, then run |
| `r` | Run? | Reply with a correction ("no, the other way") |
| Ctrl-C | anywhere | Close the popup (also cancels a pending request). Esc works too on bash 4.3+. |

If Claude needs clarification, it asks a question and you answer at the `reply>` prompt. Each turn sends the whole conversation along with the current layout of every session, window and pane. It also sends your tmux version's own command reference (`tmux list-commands`), so the syntax matches what you have installed.

Before you see a proposal, tmux parses it without running anything (`source-file -n`), and Claude quietly fixes syntax errors. Commands then run one at a time and stop at the first failure. That error goes back to Claude with the updated layout so it can propose a fix.

The last few exchanges are also included, so follow-ups like "undo that" or "do the same in window 2" work across popups.

## Configuration

Set these environment variables in the shell that starts tmux, or with `set-environment -g` in `tmux.conf`:

| Variable | Default | Meaning |
|---|---|---|
| `TMUX_AI_MODEL` | `claude-haiku-4-5-20251001` | Model to use. Try `claude-sonnet-5` for trickier requests. |
| `TMUX_AI_HISTORY` | `10` | Past exchanges sent as context (`0` to disable) |

## Examples

Claude learns the expected style from worked examples sent with every request:

- `examples.txt` in this repo ships the defaults. Improving tmux-ai is usually just adding a block here.
- `~/.config/tmux-ai/examples.txt` holds your own conventions, which override the defaults.

Blocks are separated by blank lines. The first line is the request, and the rest is the ideal answer: tmux commands, or `# ` lines for a clarifying question. Ids are illustrative. For example:

```
make a vertical split
split-window -h -t %1

send this pane to the logs window
join-pane -s %1 -t work:logs
```

When tmux-ai gets something wrong, look up the case in `~/.local/state/tmux-ai/log.jsonl` and add the corrected version as an example.

## Files

| Path | Contents |
|---|---|
| `~/.local/state/tmux-ai/history` | Your requests, for ↑ recall |
| `~/.local/state/tmux-ai/log.jsonl` | Requests, commands and outcomes, fed back as context |

Both files are trimmed automatically. Delete them anytime to start fresh. `$XDG_STATE_HOME` is respected.

## Troubleshooting

- **The popup flashes and closes:** run `~/src/tmux-ai/tmux-ai` directly inside tmux to see the error. It's usually a missing `jq` or API key.
- **Esc doesn't close the popup:** use Ctrl-C. Esc needs bash 4.3+ (`brew install bash` on macOS).
- **Too many wrong commands:** set `TMUX_AI_MODEL=claude-sonnet-5`.

## License

MIT
