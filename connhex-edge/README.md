# Connhex Edge for Home Assistant

![Supports aarch64 Architecture][aarch64-shield]
![Supports amd64 Architecture][amd64-shield]
![Supports armv7 Architecture][armv7-shield]

Bidirectional bridge between Home Assistant and [Connhex Cloud](https://connhex.com) via Connhex Edge. Monitors state changes, batches them into SenML format, and publishes to the Connhex IoT infrastructure. Also receives commands from Connhex Cloud to control Home Assistant devices remotely.

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

| Option               | Required | Description                                                                                                           | Default |
| -------------------- | -------- | --------------------------------------------------------------------------------------------------------------------- | ------- |
| `connhex_host`       | Yes      | Connhex Cloud infrastructure host (e.g. `compiuta.connhex.dev`)                                                       | —       |
| `batch_interval`     | No       | How often events are batched and sent (e.g. `30s`, `1m`)                                                              | `30s`   |
| `sync_interval`      | No       | How often entities are polled from HA to reconcile state (min: `60s`)                                                 | `60s`   |
| `log_level`          | No       | Logging verbosity: `debug`, `info`, `warn`, `error`                                                                   | `info`  |
| `filter_include`     | No       | List of HA entity IDs to monitor. If empty, all entities are monitored. Wildcards supported.                          | —       |
| `filter_exclude`     | No       | List of HA entity IDs to always ignore. Wildcards supported.                                                          | —       |
| `commands_allowlist` | No       | List of entity patterns that Connhex Cloud is allowed to control. Wildcards supported.                                | `*`     |
| `attributes_include` | No       | List of entity patterns whose attributes (e.g. brightness, color_temp) are forwarded to Connhex. Wildcards supported. | —       |

### Entity filters

- If `filter_include` is empty, **all** entities are monitored
- `filter_exclude` always takes precedence — excluded entities are ignored even if they appear in `filter_include`
- Both fields support `*` as a wildcard matching any sequence of characters

| Pattern                  | Matches                                                     | Does not match               |
| ------------------------ | ----------------------------------------------------------- | ---------------------------- |
| `sensor.*`               | `sensor.temperature`, `sensor.humidity`                     | `binary_sensor.motion`       |
| `*.temperature`          | `sensor.temperature`, `input_number.temperature`            | `sensor.humidity`            |
| `binary_sensor.*_motion` | `binary_sensor.kitchen_motion`, `binary_sensor.hall_motion` | `binary_sensor.kitchen_door` |

Example:

```yaml
filter_include:
  - sensor.*
  - light.living_room
filter_exclude:
  - sensor.internal_*
```

### Commands allowlist

The `commands_allowlist` controls which Home Assistant entities Connhex Cloud is allowed to control remotely. By default it is set to `*` (allow all). The same wildcard patterns used for entity filtering apply here.

- Set to `["*"]` to allow controlling any entity (default)
- Set to specific patterns to restrict control (e.g., `["light.*", "switch.*"]`)
- Set to an empty list `[]` to block all commands

Example — only allow controlling lights and switches:

```yaml
commands_allowlist:
  - "light.*"
  - "switch.*"
```

Commands targeting entities not in the allowlist are blocked and logged. Commands without an `entity_id` (domain-only services like `persistent_notification.create`) bypass the allowlist check.

### Entity attributes

Home Assistant entities carry attributes alongside their primary state (e.g. a light's `brightness` and `color_temp`). By default only the primary state is forwarded. Use `attributes_include` to opt in entities whose attributes should also be exported.

For each matching entity, every attribute is published to Connhex as an independent data record with a composite name, e.g. `ha:light:living_room:brightness`. Scalar attributes (numbers, booleans, strings) are sent as typed SenML values; non-scalar attributes (lists like `rgb_color`, nested objects) are JSON-encoded and delivered as opaque bytes so they can be preserved end-to-end.

Example — export attributes for all lights and a specific climate entity:

```yaml
attributes_include:
  - "light.*"
  - "climate.living_room"
```

If `attributes_include` is empty (default), no attributes are forwarded.

### Sending commands from Connhex Control

Commands can be sent from Connhex Control via the **Controls** tab of the connectable detail page, using the **Send command** panel.

The **Command** field must be set to `call_service` — this is the only command type currently supported. The **Payload** field contains a JSON object describing the Home Assistant service call:

| Field          | Type   | Required | Description                                                      |
| -------------- | ------ | -------- | ---------------------------------------------------------------- |
| `domain`       | string | yes      | The HA service domain (e.g., `light`, `switch`, `input_boolean`) |
| `service`      | string | yes      | The service to call (e.g., `turn_on`, `turn_off`, `toggle`)      |
| `entity_id`    | string | no       | The target entity. Omit for domain-only services.                |
| `service_data` | object | no       | Additional parameters for the service call.                      |

**Example — turn on a light with brightness:**

- **Service**: `ha-adapter`
- **Command**: `call_service`
- **Payload**:

```json
{
  "domain": "light",
  "service": "turn_on",
  "entity_id": "light.living_room",
  "service_data": { "brightness": 255 }
}
```

**Example — send a persistent notification (no entity_id needed):**

- **Service**: `ha-adapter`
- **Command**: `call_service`
- **Payload**:

```json
{
  "domain": "persistent_notification",
  "service": "create",
  "service_data": { "message": "Hello from Connhex!", "title": "Connhex Test" }
}
```

**Example — toggle a switch:**

- **Service**: `ha-adapter`
- **Command**: `call_service`
- **Payload**:

```json
{
  "domain": "input_boolean",
  "service": "toggle",
  "entity_id": "input_boolean.test_switch"
}
```

## First run

On first start, the add-on will register with the Connhex Cloud infrastructure using the provided `connhex_host`. The device will appear as pending in [Connhex Control](https://connhex.com/connhex-control) and must be approved before data starts flowing.

## Authentication

The add-on authenticates with Home Assistant automatically using the Supervisor token — no manual token creation is required.

[aarch64-shield]: https://img.shields.io/badge/aarch64-yes-green.svg
[amd64-shield]: https://img.shields.io/badge/amd64-yes-green.svg
[armv7-shield]: https://img.shields.io/badge/armv7-yes-green.svg
