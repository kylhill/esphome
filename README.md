# esphome

## Purpose

ESPHome firmware configurations for smart home devices integrated
with Home Assistant.

Includes device-specific configurations and reusable templates for
switches, relays, lights, fans, and power monitoring devices.

## Getting started

Install the ESPHome CLI and create an untracked `secrets.yaml`
containing the required Wi-Fi and API credentials.

Review the network settings in `templates/common.yaml` and the
device-specific configuration before deployment.

Validate and build a device configuration:

```sh
esphome validate mj-s01-1.yaml
esphome compile mj-s01-1.yaml
```

Upload firmware or inspect device logs:

```sh
esphome upload mj-s01-1.yaml
esphome logs mj-s01-1.yaml
```

## Documentation

- [Shared device templates](templates/)
- [Architecture and development conventions](AGENTS.md)
- [ESPHome documentation](https://esphome.io/)
- [License](LICENSE)