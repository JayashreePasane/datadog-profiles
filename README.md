# datadog-profiles

# Bachmann PDU – Datadog SNMP Profiles

This repository contains Python helpers to generate **Datadog SNMP profiles** for Bachmann BN-PRO PDUs.

The goal is to keep all the  OID math and repetition in code, and auto-generate clean YAML profiles that Datadog can use directly.

---

## Overview

The repo generates these profiles:

- `bachmann_inlet.yaml` – inlet-level electrical metrics (per unit)
- `bachmann_phase.yaml` – per-phase electrical metrics (L1, L2, L3 per unit)
- `bachmann_outlet.yaml` – per-phase, per-outlet electrical metrics
- `bachmann_io.yaml` – IO / digital channel metrics (input/output channels)
- `bachmann_env.yaml` – environmental sensors (temperature + humidity)
- `bachmann_pdu.yaml` – **parent profile** that extends all of the above

Each profile is a valid Datadog SNMP profile and can be dropped into the DataDog Agent’s `snmp.d/profiles` directory.

---

## Supported Topology

- **Main PDU**: `unit_id = 0`, tagged as `unit_name:main`
- **Link PDUs**: `unit_id = 1..19`, tagged as `unit_name:link_<n>`
- **3 phases per unit**: `L1`, `L2`, `L3`
- **4 outlets per phase** (0–3)
- Up to **10 environmental sensors** per unit
- IO channels (input/output) per unit

All generated metrics are tagged with things like:

- `unit_id`, `unit_name`
- `measurement_level` (`inlet`, `phase`, `outlet`, `io`, `env`)
- `phase`, `outlet`, `sensor_index`, `channel` (where applicable)

---

## Files

### Generators

- `generate_inlet_profile_metrics.py`  
  Generates `bachmann_inlet.yaml` – inlet-level electrical metrics:
  - `bacCurrentMain`, `bacActivePowerUnit3`, …  
  - Metric types: gauges + counters (energy).

- `generate_phase_metrics.py`  
  Generates `bachmann_phase.yaml` – per-phase metrics per unit:
  - `bacVoltageMainL1`, `bacActivePowerUnit5L3`, …

- `generate_outlet_phase_yaml.py`  
  Generates `bachmann_outlet.yaml` – per-phase, per-outlet metrics:
  - `bacCurrentMainL1Outlet0`, `bacActiveEnergyUnit2L3Outlet3`, …

- `generate_io_profile_metrics.py`  
  Generates `bachmann_io.yaml` – IO channel metrics:
  - `ioOutputChannel1Main`, `ioInputChannel3Unit4`, …

- `generate_env_profile_metrics.py`  
  Generates `bachmann_env.yaml` – temperature + humidity from up to 10 sensors per unit:
  - `bacTemperatureMainSensor4`, `bacHumidityUnit7Sensor9`, …

Each script prints a **complete Datadog SNMP profile** to stdout:

- `sysobjectid: 1.3.6.1.4.1.31770.*`
- `metadata.device.vendor: "bachmann"`
- `metadata.device.type: "pdu"`
- `metrics: [...]` (all SNMP symbols and tags)

### Orchestrator

- `main.py`  
  Small helper that:
  1. Runs all generator scripts.
  2. Writes the individual profiles:
     - `bachmann_inlet.yaml`
     - `bachmann_phase.yaml`
     - `bachmann_outlet.yaml`
     - `bachmann_io.yaml`
     - `bachmann_env.yaml`
  3. Writes the parent profile `bachmann_pdu.yaml` that `extends` all of them.

---

## Requirements

- Python **3.8+** (anything reasonably recent should work)
---

## Usage

### 1. Generate all profiles

From the repository root:

```bash
python3 main.py
