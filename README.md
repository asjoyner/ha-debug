# ha-debug

Home Assistant debugging CLI for troubleshooting automations, sensors, and
light groups. Queries the HA REST API to provide diagnostic views that are
tedious to assemble manually.

## Prerequisites

- Python 3.8+ (no external dependencies — stdlib only)
- A Home Assistant instance with a long-lived access token

## Installation

### 1. Clone and symlink the CLI

```bash
git clone https://github.com/asjoyner/ha-debug.git ~/code/ha-debug
ln -sf ~/code/ha-debug/ha-debug ~/bin/ha-debug
```

### 2. Create the credential file

Both `ha-debug` and the `hass-cli` wrapper (`~/bin/ha`) read credentials from
the same file:

```bash
mkdir -p ~/.config/ha-debug
chmod 700 ~/.config/ha-debug

cat > ~/.config/ha-debug/credentials << 'EOF'
# Home Assistant credentials
HASS_TOKEN=<your-long-lived-access-token>
HASS_SERVER=http://127.0.0.1:8123
EOF

chmod 600 ~/.config/ha-debug/credentials
```

Generate a long-lived access token in HA under your profile → Security →
Long-Lived Access Tokens.

Environment variables `HASS_TOKEN` and `HASS_SERVER` take precedence over the
credential file if set.

### 3. (Optional) Install the Claude Code agent skill

This repo includes a Claude Code skill that teaches the agent when and how to
use `ha-debug` for HA troubleshooting.

```bash
mkdir -p ~/.claude/skills/ha-debug
ln -sf ~/code/ha-debug/SKILL.md ~/.claude/skills/ha-debug/SKILL.md
```

To allow the agent to run `ha-debug` without approval prompts, add these
entries to `~/.claude/settings.json` under `permissions.allow`:

```json
"Bash(ha-debug *)",
"Bash(/home/<user>/bin/ha-debug *)"
```

## Commands

| Command     | Purpose                                              |
|-------------|------------------------------------------------------|
| `timeline`  | Interleaved state history for multiple entities      |
| `flap`      | Detect flapping sensors with short-cycle analysis    |
| `trace`     | Show automation traces with action timestamps        |
| `group`     | List light group members with availability status    |
| `overlap`   | Find automations/groups controlling the same entity  |
| `logbook`   | Human-readable logbook with history API fallback     |
| `states`    | Glob-based entity search with current state          |

Run `ha-debug --help` or `ha-debug <command> --help` for full usage.

## Quick examples

```bash
# What happened to the living room lights in the last 10 minutes?
ha-debug timeline light.zg_living_room binary_sensor.presence_sensor_living_room --since 10m

# Is the presence sensor flapping?
ha-debug flap binary_sensor.presence_sensor_everything --since 1h

# Which bulbs in the group are actually responding?
ha-debug group light.zg_everything

# Are two automations fighting over the same light?
ha-debug overlap light.kitchen_walkways_in_everything

# Find all kitchen-related entities
ha-debug states 'binary_sensor.*kitchen*'
```

## Common flags

- `--since <duration>` — How far back to look. Accepts: `30s`, `10m`, `1h`, `2d`. Default varies by command.
- `--threshold <seconds>` — (`flap` only) Duration below which an on/off cycle is flagged as short. Default: 30.

## Credential file format

`~/.config/ha-debug/credentials` uses simple `KEY=VALUE` format:

```
# Comments start with #
HASS_TOKEN=eyJhbGci...
HASS_SERVER=http://127.0.0.1:8123
```

Quotes around values are optional and will be stripped if present.
