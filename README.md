# device_status

A [Home Assistant](https://www.home-assistant.io/) custom component that reports whether network devices (computers, routers, servers, ESP8266 sensors, smart TVs and so on) are reachable. Each configured host becomes a sensor whose state is `online` or `offline`, based on a single [`fping`](https://fping.org/) probe.

> **Note:** this is a legacy YAML `sensor` platform (manifest version 0.9.0, first written in 2018). It has not been confirmed against current Home Assistant releases. <!-- TODO: verify compatibility with current Home Assistant -->

## How it works

* On every update, the sensor runs `fping -C1 -q <host> 2>&1 | grep -v '-' | wc -l` through the shell.
* The state is `online` when that pipeline counts exactly one output line without a `-` in it (a reply prints `<host> : <ms>`, a lost probe prints `<host> : -`), and `offline` otherwise. See [Known limitations](#known-limitations) for host names that contain a hyphen.
* The icon follows the state: `mdi:arrow-up-bold-circle-outline` when online, `mdi:arrow-down-bold-circle-outline` when offline.
* Entities are named `sensor.device_status_<device_id>`, where `<device_id>` is the key you use under `devices`.

## Requirements

* Home Assistant with support for custom components.
* The `fping` binary on the `PATH` of the Home Assistant process.

On every startup the component runs `apk add fping` when `/usr/sbin/fping` is missing, and separately runs `apt inftall fping -y` (a typo, so it always fails) when `/usr/bin/fping` is missing. On Debian-based systems, install it yourself:

```bash
sudo apt-get install fping --yes
```

## Installation

1. Copy the `custom_components/device_status/` folder into your Home Assistant configuration directory, so that you have `<config>/custom_components/device_status/sensor.py`.
2. Add the configuration below to `configuration.yaml`.
3. Restart Home Assistant.

## Configuration

```yaml
sensor:
  - platform: device_status
    scan_interval: 10
    devices:
      internet_connection:
        host: 8.8.8.8
        name: "Internet Connection"
      nas:
        host: 192.168.1.10
        name: "NAS"
```

| Key | Required | Default | Description |
|-----|----------|---------|-------------|
| `scan_interval` | No | `10` (seconds) | How often every device is probed. |
| `devices` | Yes | — | Map of device IDs (slugs) to device settings. |
| `devices.<id>.host` | Yes | — | Host name or IP address passed to `fping`. |
| `devices.<id>.name` | No | — | Friendly name of the sensor. |
| `devices.<id>.icon` | No | `mdi:desktop-classic` | Accepted by the schema, but ignored: the icon always follows the online/offline state. |

## Known limitations

* **Shell command built from the config.** The `host` value is inserted into a shell command without quoting, so only put trusted host names or IP addresses in the configuration.
* **Host names with a hyphen always show `offline`.** The `grep -v '-'` filter drops any output line containing a hyphen, including a successful reply from a host such as `my-nas.lan`. Use an IP address or a host name without hyphens.
* **Unresolvable host names show `offline`** without any error in the logs.
* **One probe per update.** A single lost ping marks the device `offline` until the next scan.
* **Manifest requirement.** `manifest.json` lists `fping` under `requirements`, which Home Assistant installs as a Python package from PyPI; the component actually needs the `fping` system binary.

## Credits

Written by Tomer Klein, with thanks to [Tomer Figenblat](https://github.com/TomerFi) for all the help.

## License

[Apache License 2.0](License)
