---
name: ha-debug
description: >-
  Use ha-debug CLI to troubleshoot Home Assistant automations, sensors, lights,
  and entity state. TRIGGER when: user asks about HA entity behavior, automation
  issues, flapping sensors, light group problems, or "why did X turn on/off".
  DO NOT TRIGGER for: HA config editing, dashboard changes, or general HA setup.
---

# ha-debug: Home Assistant Debugging CLI

> **Purpose:** Quickly diagnose HA automation issues, sensor flapping, light group overlaps, and entity state without writing raw API calls.

## Setup

The `ha-debug` tool is installed at `~/bin/ha-debug` and auto-loads credentials from `~/.config/ha-debug/credentials`. No environment setup needed.

**IMPORTANT:** Always call it directly via its full path to avoid approval prompts:

```bash
/home/asjoyner/bin/ha-debug <subcommand> [args]
```

## Available Commands

### `timeline` — Interleaved state history for multiple entities

Best for: Understanding the sequence of events across related entities (e.g. a sensor and its light).

```bash
/home/asjoyner/bin/ha-debug timeline light.zg_everything binary_sensor.presence_sensor_everything --since 10m
/home/asjoyner/bin/ha-debug timeline 'binary_sensor.*kitchen*' --since 1h
```

### `flap` — Detect flapping sensors

Best for: Diagnosing sensors that toggle on/off rapidly (presence sensors, motion sensors).

```bash
/home/asjoyner/bin/ha-debug flap binary_sensor.presence_sensor_everything --since 30m
/home/asjoyner/bin/ha-debug flap binary_sensor.presence_sensor_everything --since 1h --threshold 30
```

### `trace` — Show automation trace with action timestamps

Best for: Understanding what an automation did and when (which actions fired, errors).

```bash
/home/asjoyner/bin/ha-debug trace automation.motion_activated_light_everything
/home/asjoyner/bin/ha-debug trace motion_activated_light_kitchen
```

### `group` — List light group members with state

Best for: Seeing which bulbs in a Zigbee group are on, off, or unavailable.

```bash
/home/asjoyner/bin/ha-debug group light.zg_everything
/home/asjoyner/bin/ha-debug group light.zg_kitchen_walkways
```

### `overlap` — Find automations controlling the same entity

Best for: Diagnosing race conditions where two automations fight over the same light/device.

```bash
/home/asjoyner/bin/ha-debug overlap light.zg_everything
/home/asjoyner/bin/ha-debug overlap light.kitchen_walkways_in_everything
```

### `logbook` — Human-readable logbook entries

Best for: Quick check of what happened to an entity recently.

```bash
/home/asjoyner/bin/ha-debug logbook light.zg_everything --since 10m
/home/asjoyner/bin/ha-debug logbook binary_sensor.presence_sensor_everything --since 1h
```

### `states` — List entities matching a pattern

Best for: Discovering entity IDs and their current state. Supports glob patterns.

```bash
/home/asjoyner/bin/ha-debug states '*everything*'
/home/asjoyner/bin/ha-debug states 'binary_sensor.*kitchen*'
/home/asjoyner/bin/ha-debug states 'light.zg_*'
```

## Diagnostic Workflow

When the user reports an automation issue (e.g. "lights turned on then off"):

1. **Identify entities:** Use `states` to find relevant entity IDs
2. **Build timeline:** Use `timeline` with the sensor + light entities over the relevant window
3. **Check for flapping:** Use `flap` on the triggering sensor
4. **Check group membership:** Use `group` to see which bulbs responded
5. **Check for conflicts:** Use `overlap` to find competing automations
6. **Check automation trace:** Use `trace` to see the automation's action sequence

## Common Flags

- `--since <duration>` — How far back to look. Accepts: `30s`, `10m`, `1h`, `2d`
- `--threshold <seconds>` — (flap only) Duration below which a cycle is flagged as short

## Notes

- The tool reads `HASS_TOKEN` and `HASS_SERVER` from `~/.config/ha-debug/credentials`
- Environment variables `HASS_TOKEN` / `HASS_SERVER` take precedence if set
- Entity patterns use glob syntax: `*` matches anything, `?` matches one character
- Zigbee group member detection works by name matching (e.g. `light.zg_everything` finds `light.everything_room_*`)
- The `overlap` command reads `/var/lib/hass/config/automations.yaml` directly for full automation config inspection
