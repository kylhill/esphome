# ESPHome Device Configuration Repository

This repo contains ESPHome firmware configurations for smart home devices integrated with Home Assistant.

## ESPHome Commands

```bash
esphome validate <device>.yaml   # Check config for errors
esphome compile <device>.yaml    # Build firmware
esphome upload <device>.yaml     # Flash device OTA
esphome logs <device>.yaml       # Stream serial/network logs
```

## Architecture

Device configs live at the root (e.g. `mj-s01-1.yaml`). They compose behavior using a two-layer template system:

**Layer 1 – Root device config** sets substitutions (`device_name`, `friendly_name`, `area`, `static_ip`) and pulls in two packages: `templates/common.yaml` (shared WiFi/API/OTA config) and a device-type template.

**Layer 2 – Device-type templates** (in `templates/`) pull in a hardware base template and one or more component templates from `templates/components/`. They also define the default behavior by extending stub scripts.

```
mj-s01-1.yaml
  └── packages:
        common: templates/common.yaml          # WiFi, API, OTA, uptime sensor
        device: templates/mj-s01-light-auto.yaml
                  └── packages:
                        parent: mj-s01-base.yaml   # GPIO pins, LED globals, click scripts
                        light:  components/mj-s01-light.yaml  # relay_light entity
```

**Device families:**

| Prefix | Hardware | Notes |
|---|---|---|
| `mj-s01-*` | Martin Jerry MJ-S01 (single-pole) | ESP8266, red+blue LEDs, multi-click |
| `mj-st02-*` | Martin Jerry MJ-ST02 (3-way) | Tuya MCU over UART |
| `athom-mini-*` | Athom Mini Switch | ESP8285, single LED |
| `s31-*` | Sonoff S31 | ESP8266, CSE7766 power monitoring |
| `shelly-1-*` | Shelly 1 | ESP8266, external switch input |
| `shelly-em` | Shelly EM | ADE7953 dual-phase power monitoring |

## Key Conventions

**Required substitutions** — every root device config must define all four:
```yaml
substitutions:
  device_name: mj-s01-1          # must match filename (without .yaml)
  friendly_name: "Room SW 1"     # shown in Home Assistant
  area: "Living Room"            # Home Assistant area
  static_ip: 192.168.7.xx        # 192.168.7.x subnet; check existing files to avoid conflicts
```

**Script extension pattern** — base templates define empty stub scripts (`on_single`, `on_double`, `on_long`). Device templates and root configs use `!extend` to inject behavior without overwriting:
```yaml
script:
  - id: !extend on_single
    then:
      - light.toggle: relay_light
```

**Main entities use `name: None`** — the entity name in Home Assistant is derived from the device's `friendly_name`. Don't give the primary relay/fan/cover a string name.

**`unused/` directory** — retired or spare device configs are moved here rather than deleted. They are not compiled or deployed.

**`secrets.yaml` is gitignored** — it holds `wifi_ssid`, `wifi_password`, `ap_password`, `api_key`, and `ota_password`. Use `templates/secrets.yaml` as a reference for the expected keys. Never commit the real file.

**Sensor filters** — use `delta` + `throttle_average` with `or:` to balance responsiveness vs. MQTT/HA noise. See `shelly-em.yaml` and `templates/components/cse7726.yaml` for examples.

**Adding a new device:**
1. Copy the closest existing root config, set a new `device_name`/`static_ip`.
2. Pick the appropriate template from `templates/` (or create a new one following the base + component pattern).
3. Extend the relevant click scripts to wire up the primary entity action.


<!-- headroom:rtk-instructions -->
# RTK (Rust Token Killer) - Token-Optimized Commands

When running shell commands, **always prefix with `rtk`**. This reduces context
usage by 60-90% with zero behavior change. If rtk has no filter for a command,
it passes through unchanged — so it is always safe to use.

## Key Commands
```bash
# Git (59-80% savings)
rtk git status          rtk git diff            rtk git log

# Files & Search (60-75% savings)
rtk ls <path>           rtk read <file>         rtk grep <pattern>
rtk find <pattern>      rtk diff <file>

# Test (90-99% savings) — shows failures only
rtk pytest tests/       rtk cargo test          rtk test <cmd>

# Build & Lint (80-90% savings) — shows errors only
rtk tsc                 rtk lint                rtk cargo build
rtk prettier --check    rtk mypy                rtk ruff check

# Analysis (70-90% savings)
rtk err <cmd>           rtk log <file>          rtk json <file>
rtk summary <cmd>       rtk deps                rtk env

# GitHub (26-87% savings)
rtk gh pr view <n>      rtk gh run list         rtk gh issue list

# Infrastructure (85% savings)
rtk docker ps           rtk kubectl get         rtk docker logs <c>

# Package managers (70-90% savings)
rtk pip list            rtk pnpm install        rtk npm run <script>
```

## Rules
- In command chains, prefix each segment: `rtk git add . && rtk git commit -m "msg"`
- For debugging, use raw command without rtk prefix
- `rtk proxy <cmd>` runs command without filtering but tracks usage
<!-- /headroom:rtk-instructions -->
