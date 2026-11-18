# APC Smart-UPS High-Precision SNMP Package for Home Assistant

High-precision Home Assistant YAML package and Lovelace dashboard card for **APC Smart-UPS** (`SMT` / `SMX` / `SRT` series with AP9630, AP9631, AP9640, or AP9641 Network Management Cards) over SNMPv1.

Most community APC SNMP templates rely solely on the legacy integer branches of the APC PowerNet MIB, truncating voltages, currents, load percentages, and temperatures to whole integers and estimating wattage from load percentage. This configuration taps into the **APC PowerNet High-Precision (`upsHighPrec*`)** registers to expose **0.1-unit decimal resolution**, **true measured Active Power (`W`) & Apparent Power (`VA`)**, **cumulative Energy (`kWh`)** ready for the Home Assistant Energy Dashboard, and **battery lifecycle metadata**.

## Repository Contents

- [`apc_smart_ups.yaml`](apc_smart_ups.yaml) — Home Assistant package with 20 high-precision SNMP sensors
- [`apc_smart_ups_card.yaml`](apc_smart_ups_card.yaml) — Lovelace `entities` dashboard card YAML

## Features

- **0.1-unit precision** for Input Voltage (`V`), Output Voltage (`V`), Battery DC Voltage (`V`), Output Frequency (`Hz`), Output Current (`A`), Load (`%`), Battery Charge (`%`), and Battery Temperature (`°C`)
- **True Active Power (`W`) & Apparent Power (`VA`)** read directly from the UPS DSP rather than estimated from load percentage
- **Energy Dashboard integration**: exposes cumulative output energy (`kWh`, `0.01 kWh` resolution) with `state_class: total_increasing`
- **Full operating mode & transfer reason decoding** (`On Line`, `On Battery`, `On Smart Boost`, `On Smart Trim`, `Bypass`, `Brownout`, `UPS self test`, etc.)
- **Battery lifecycle tracking**: battery health, replacement SKU (e.g., `RBC55`), installation date, and self-test diagnostics

## Prerequisites

1. **Enable SNMPv1 on your APC Network Management Card:**
   - Log in to the APC NMC web interface.
   - Navigate to **Configuration → Network → SNMPv1 → Access** and check **Enable**.
   - Under **Access Control**, configure a read-only community string (avoid keeping the default `public` if your firewall uses IDS/IPS, as Suricata-based systems such as UniFi IPS flag cross-subnet `public` SNMP traffic).
   - Ensure the NMC has a valid **Default Gateway** configured if Home Assistant resides on a different VLAN/subnet.

2. **Enable Home Assistant Packages:**
   In your `/config/configuration.yaml`, ensure `packages` is enabled under `homeassistant:`:
   ```yaml
   homeassistant:
     packages: !include_dir_named packages
   ```

## Installation

1. **Create the package file:**
   Create `/config/packages/apc_smart_ups.yaml` (create the `/config/packages` directory first if it does not exist) and paste the contents of [`apc_smart_ups.yaml`](apc_smart_ups.yaml) into it.

2. **Configure UPS IP & SNMP Community:**
   Replace `192.168.1.101` with your APC Network Management Card's IP address and `public` with your SNMPv1 read-only community string across the sensor definitions.

3. **Validate & Restart Home Assistant:**
   - Go to **Developer Tools → YAML** and click **Check Configuration**.
   - Perform a full **Restart Home Assistant** (a simple YAML reload is not enough on initial setup because the legacy `snmp` platform requires a full restart to register new entities).

4. **Add the Lovelace Dashboard Card:**
   Open your dashboard, click **Edit Dashboard → + Add Card → Manual**, and paste the contents of [`apc_smart_ups_card.yaml`](apc_smart_ups_card.yaml).

## Included Sensors & OID Reference

All OIDs below belong to the **APC PowerNet MIB** (`1.3.6.1.4.1.318`).

| Entity ID | Description | Unit | OID | Scaling |
| :--- | :--- | :---: | :--- | :---: |
| `sensor.apc_ups_status` | Operating Status | — | `1.3.6.1.4.1.318.1.1.1.4.1.1.0` | Value map |
| `sensor.apc_ups_comm_status` | UPS-to-NMC Communication | — | `1.3.6.1.4.1.318.1.1.1.8.1.0` | Value map |
| `sensor.apc_ups_battery_soc` | Battery State of Charge | `%` | `1.3.6.1.4.1.318.1.1.1.2.3.1.0` | `÷ 10` |
| `sensor.apc_ups_runtime_remaining` | Estimated Runtime Remaining | text | `1.3.6.1.4.1.318.1.1.1.2.2.3.0` | Timeticks |
| `sensor.apc_ups_output_power` | Output Active Power | `W` | `1.3.6.1.4.1.318.1.1.1.4.2.8.0` | `1:1` |
| `sensor.apc_ups_apparent_power` | Output Apparent Power | `VA` | `1.3.6.1.4.1.318.1.1.1.4.2.9.0` | `1:1` |
| `sensor.apc_ups_load_percentage` | Output Load Percentage | `%` | `1.3.6.1.4.1.318.1.1.1.4.3.3.0` | `÷ 10` |
| `sensor.apc_ups_output_current` | Output Current | `A` | `1.3.6.1.4.1.318.1.1.1.4.3.4.0` | `÷ 10` |
| `sensor.apc_ups_energy_total` | Cumulative Output Energy | `kWh` | `1.3.6.1.4.1.318.1.1.1.4.3.6.0` | `÷ 100` |
| `sensor.apc_ups_input_voltage` | Input Voltage | `V` | `1.3.6.1.4.1.318.1.1.1.3.3.1.0` | `÷ 10` |
| `sensor.apc_ups_output_voltage` | Output Voltage | `V` | `1.3.6.1.4.1.318.1.1.1.4.3.1.0` | `÷ 10` |
| `sensor.apc_ups_output_frequency` | Output Frequency | `Hz` | `1.3.6.1.4.1.318.1.1.1.4.3.2.0` | `÷ 10` |
| `sensor.apc_ups_battery_voltage` | Battery DC Voltage | `V` | `1.3.6.1.4.1.318.1.1.1.2.3.4.0` | `÷ 10` |
| `sensor.apc_ups_battery_temperature` | Battery Temperature | `°C` | `1.3.6.1.4.1.318.1.1.1.2.3.2.0` | `÷ 10` |
| `sensor.apc_ups_battery_status` | Battery Health Status | — | `1.3.6.1.4.1.318.1.1.1.2.2.4.0` | Value map |
| `sensor.apc_ups_battery_installed_date` | Battery Replacement Date | — | `1.3.6.1.4.1.318.1.1.1.2.1.3.0` | String |
| `sensor.apc_ups_battery_sku` | Battery Cartridge SKU | — | `1.3.6.1.4.1.318.1.1.1.2.2.19.0` | String |
| `sensor.apc_ups_last_transfer_reason` | Last Transfer to Battery Reason | — | `1.3.6.1.4.1.318.1.1.1.3.2.5.0` | Value map |
| `sensor.apc_ups_last_self_test_result` | Last Self-Test Result | — | `1.3.6.1.4.1.318.1.1.1.7.2.3.0` | Value map |
| `sensor.apc_ups_last_self_test_date` | Last Self-Test Date | — | `1.3.6.1.4.1.318.1.1.1.7.2.4.0` | String |

### Status Value Mapping (`sensor.apc_ups_status`)

| Raw Value | Decoded State | Raw Value | Decoded State |
| :---: | :--- | :---: | :--- |
| `1` | `Unknown` | `7` | `Off` |
| `2` | `On Line` | `8` | `Rebooting` |
| `3` | `On Battery` | `9` | `Switched Bypass` |
| `4` | `On Smart Boost` | `10` | `Hardware Failure Bypass` |
| `5` | `Timed Sleeping` | `11` | `Sleeping Until Power Returns` |
| `6` | `Software Bypass` | `12` | `On Smart Trim` |

## Troubleshooting

- **Sensors show `unknown` across VLANs / subnets:**
  - Verify **UDP port 161** is allowed from your Home Assistant host to the APC NMC IP (and return traffic is permitted).
  - If using **UniFi OS / Suricata IDS/IPS**, cross-subnet SNMP queries using the default `public` community string trigger signature `GPL SNMP public access udp (SID 2101411)` and get silently dropped. Configure a custom community string (e.g., `apc_mon`) on both the NMC and in YAML.
- **High-Precision OIDs (`...2.3.*`, `...3.3.*`, `...4.3.*`) return `unknown` on older Smart-UPS models:**
  - Older white/beige Smart-UPS (`SUA` series) or older NMC firmware revisions may only expose integer registers. If `...3.3.1.0` or `...4.3.1.0` returns `unknown`, fall back to the integer OIDs (`...3.2.1.0` for Input Voltage, `...4.2.1.0` for Output Voltage, `...4.2.3.0` for Load %, `...2.2.2.0` for Battery Temp) and remove `/ 10` from their `value_template`.

## License

MIT
