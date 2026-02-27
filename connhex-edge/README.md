# Connhex Edge for Home Assistant

![Supports aarch64 Architecture][aarch64-shield]
![Supports amd64 Architecture][amd64-shield]
![Supports armv7 Architecture][armv7-shield]

Bridges Home Assistant entities to [Connhex Cloud](https://connhex.com) via Connhex Edge. Monitors state changes, batches them into SenML format, and publishes to the Connhex IoT infrastructure.

## Requirements

- Home Assistant OS (HAOS) or Home Assistant Supervised
- A Connhex Cloud account and infrastructure host (e.g. `compiuta.connhex.dev`)

## Installation

1. In Home Assistant, go to **Settings → Add-ons → Add-on Store**
2. Click the **⋮** menu (top right) and select **Repositories**
3. Add the following URL and click **Add**:
   ```
   https://github.com/compiuta-origin/connhex-hassio-addons
   ```
4. Find **Connhex Edge for Home Assistant** in the store and click **Install**

## Configuration

After installation, go to the **Configuration** tab of the add-on and fill in the following options:

| Option           | Required | Description                                                            | Default |
| ---------------- | -------- | ---------------------------------------------------------------------- | ------- |
| `connhex_host`   | Yes      | Connhex Cloud infrastructure host (e.g. `compiuta.connhex.dev`)        | —       |
| `batch_interval` | No       | How often events are batched and sent (e.g. `30s`, `1m`)               | `30s`   |
| `log_level`      | No       | Logging verbosity: `debug`, `info`, `warn`, `error`                    | `info`  |
| `filter_include` | No       | List of HA entity IDs to monitor. If empty, all entities are monitored. Wildcards supported. | —  |
| `filter_exclude` | No       | List of HA entity IDs to always ignore. Wildcards supported.                                 | —  |

### Entity filters

- If `filter_include` is empty, **all** entities are monitored
- `filter_exclude` always takes precedence — excluded entities are ignored even if they appear in `filter_include`
- Both fields support `*` as a wildcard matching any sequence of characters

| Pattern | Matches | Does not match |
|---|---|---|
| `sensor.*` | `sensor.temperature`, `sensor.humidity` | `binary_sensor.motion` |
| `*.temperature` | `sensor.temperature`, `input_number.temperature` | `sensor.humidity` |
| `binary_sensor.*_motion` | `binary_sensor.kitchen_motion`, `binary_sensor.hall_motion` | `binary_sensor.kitchen_door` |

Example:

```yaml
filter_include:
  - sensor.*
  - light.living_room
filter_exclude:
  - sensor.internal_*
```

## First run

On first start, the add-on will register with the Connhex Cloud infrastructure using the provided `connhex_host`. The device will appear as pending in [Connhex Control](https://connhex.com/connhex-control) and must be approved before data starts flowing.

## Authentication

The add-on authenticates with Home Assistant automatically using the Supervisor token — no manual token creation is required.

[aarch64-shield]: https://img.shields.io/badge/aarch64-yes-green.svg
[amd64-shield]: https://img.shields.io/badge/amd64-yes-green.svg
[armv7-shield]: https://img.shields.io/badge/armv7-yes-green.svg
