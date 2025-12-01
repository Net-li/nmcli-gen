🔥 New tool: **nmcli-gen**  
Generate fully-idempotent NetworkManager profiles from a simple YAML specification.
No manual nmcli work. No guesswork. One YAML → one shell script.

This repository contains:

A clean public description of what the tool does
YAML examples you can freely inspect
A link to the commercial version on Gumroad (which contains the generator script)
👉 The actual Python generator is not published here.

---

## What it solves

Hand-editing `.nmconnection` files or re-creating NetworkManager profiles by hand is:

- error-prone,
- not reproducible,
- hard to version-control.

`nmcli-gen` lets you:

- keep your Wi-Fi / Ethernet config in **YAML**,  
- generate an **idempotent apply script** (`nmcli` calls only),  
- commit YAML + script to git,  
- re-apply safely on any compatible Linux machine.

---

## High-level features

- ✅ Generate a `bash` script from a YAML spec
- ✅ Wi-Fi and Ethernet profiles
- ✅ Autoconnect on/off and `autoconnect-retries` handling (with a safe fallback if `-1` is not supported)
- ✅ IPv4 / IPv6:
  - `auto`, `manual`, `disabled`
  - static address, gateway, DNS, routes
- ✅ Policy block:
  - `connection.metered`
  - IPv6 privacy (`off|prefer|only`)
  - per-family `route-metric` (IPv4/IPv6)
- ✅ Wi-Fi options:
  - SSID, PSK
  - interface name
  - optional MAC policy (cloned / random / permanent)
  - optional scan-time MAC randomization
  - Wi-Fi powersave
  - `bgscan` vs `scan-interval` (mutually exclusive, with warnings)
- ✅ `--dry-run` mode (the generated script only echoes `nmcli` commands)
- ✅ `--no-recreate` mode (do not delete existing profiles, modify/create in-place)
- ✅ Emits warnings as comments at the end of the script

---

🧩 Why YAML?
Because YAML is cleaner than:
  - long nmcli con modify ... strings
  - half-broken .nmconnection templates
  - trial-and-error every time you ship a Wi-Fi profile to an embedded device

## YAML snippet example

connections:
  - id: eth-dhcp
    type: ethernet
    iface: enp0s3

    ipv4:
      method: auto

    policy:
      metered: no
      route_metric:
        ipv4: 200

Another Wi-Fi example:

connections:
  - id: wifi-psk
    type: wifi
    iface: wlan0
    autoconnect: true

    wifi:
      ssid: "MySSID"
      psk: "s3cr3t"
      powersave: 0
      scan_random: true

    ipv4:
      method: auto
    ipv6:
      method: disabled

Generated script (short excerpt)

The tool generates a shell script similar to:

#!/bin/sh
set -eu
DRY_RUN=0

do() {
  if [ "$DRY_RUN" = "1" ]; then
    echo "$@"
  else
    "$@"
  fi
}
do nmcli con delete "eth-dhcp" || true
do nmcli con add type ethernet ifname "enp0s3" con-name "eth-dhcp"
do nmcli con modify "eth-dhcp" connection.autoconnect yes
do nmcli con modify "eth-dhcp" ipv4.method auto
do nmcli con modify "eth-dhcp" ipv4.route-metric 200
do nmcli con up "eth-dhcp" || true

How to get nmcli-gen

The full Python generator, with all options and examples, is available on Gumroad:

👉 https://netli.gumroad.com/l/nmcligen

You get:

nmcli-gen.py (Python 3 script)
example YAML files (Ethernet + Wi-Fi)
usage notes
ready-to-run generated scripts


🧭 Roadmap (public)

These are not included in v1, but planned:
                                Feature	Status
EAP / 802.1X	                      planned
WPA-Enterprise	                    planned
Hotspot/AP mode	                    planned
VLAN profiles	                      planned
Connection monitoring / diagnostics	planned
