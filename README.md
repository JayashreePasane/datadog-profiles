# datadog-profiles

# Bachmann PDU – Datadog SNMP Profiles

This repository contains Python helpers to generate **Datadog SNMP profiles** for Bachmann BN-PRO PDUs.

The goal is to keep all the  OID math and repetition in code, and auto-generate clean YAML profiles that Datadog can use directly.

---

## Overview

The repo generates these profiles:

- Category	Generator Script	Output File
  - Inlet electrical metrics	generate_inlet_profile.py	**bachmann_inlet_metrics.yaml**
  - Phase electrical metrics	generate_phase_profile.py	**bachmann_phase_metrics.yaml**
  - outlet per phase metrics	generate_outlet_per_phase_profile.py	**bachmann_outlet_metrics.yaml**
  - IO channels metrics	generate_io_profile_metrics.py	**bachmann_io_metrics.yaml**
  - Environmental sensors	generate_sensor_profile.py	**bachmann_sensors_metrics.yaml**
  - Inlet status	generate_inlet_status_profile.py	**bachmann_inlet_status.yaml**
  - Phase status	generate_phase_status_profile.py	**bachmann_phase_status.yaml**
  - Outlet status	generate_outlet_status_profile.py **bachmann_outlet_status.yaml**
  - Global PDU status generate_global_status_profile.py **bachmann_global_status.yaml**
  - Parent profile	(auto-built in main.py)	**bachmann_pdu.yaml**

Each profile is a valid Datadog SNMP profile and can be dropped into the DataDog Agent’s `snmp.d/profiles` directory.

## Where to place the generated profiles
  - all generated YAML files, place all of them under:/etc/datadog-agent/conf.d/snmp.d/profiles/ This is the supported location for custom SNMP profiles, and keeps them safe from being overwritten by Agent upgrades.
    
## Which profile to reference in snmp.d/conf.yaml
  - In your Agent’s snmp.d/conf.yaml, reference the parent profile so that the Agent uses the combined definitions. For our Bachmann PDUs, that’s **profile: bachmann_pdu**
  - This tells Datadog to apply the single parent profile, which extends all the individual metric and status profiles you’ve generated.

---

## Supported Topology

- Element	Details
  - Main PDU	unit_id = 0, tagged unit_name:main
  - Link PDUs	unit_id = 1..19, tagged unit_name:link_<n>
  - Phases per unit	L1, L2, L3
  - Outlets per phase	1..4 (per phase, per unit)
  - External sensors	Up to 10 (temp/humidity)
  - IO channels	Digital input/output
  - Status profiles	Phase, inlet, outlet, PDU-level alarms & health
- Each metric is automatically tagged using:
  - unit_id, unit_name, phase, outlet, sensor_index, channel,
  - measurement_level (inlet, phase, outlet, io, external_sensor, status)

## How to Control Number of Link PDUs

- All generator scripts use this pattern
  - units = range(0, 20)  # 0 = Main PDU, 1..19 = Link PDUs
  - To reduce number of Link PDUs, simply edit the range: in each generate_*.py file
      - **Desired Setup	Use**
        - Only main PDU (no link units)	units = range(0, 1)
        - Main + 3 Link PDUs	units = range(0, 4)
        - Main + 10 Link PDUs	units = range(0, 11)

## Files

### Generators

- `generate_inlet_profile.py`  
  Generates `bachmann_inlet_metrics.yaml` – inlet-level electrical metrics:
  - `bacCurrentMain`, `bacActivePowerLink3`, …  
  - Metric types: gauges.

- `generate_phase_profile.py`  
  Generates `bachmann_phase_metrics.yaml` – per-phase metrics per unit:
  - `bacVoltageMainL1`, `bacActivePowerLink5L3`, …

- `generate_outlet_per_phase_profile.py`  
  Generates `bachmann_outlet_metrics.yaml` – per-outlet metrics in a phase:
  - `bacCurrentMainL1Outlet0`, `bacActiveEnergyLink1L3Outlet3`, …

- `generate_io_profile_metrics.py`  
  Generates `bachmann_io_metrics.yaml` – IO channel metrics:
  - `ioOutputChannel1Main`, `ioInputChannel3Unit4`, …

- `generate_sensor_profile.py`  
  Generates `bachmann_sensors_metrics.yaml` – temperature + humidity from up to 10 sensors per unit:
  - `bacTemperatureMainSensor0`, `bacHumidityLink1Sensor0`, …

Each script prints a **complete Datadog SNMP profile** to stdout:

- `sysobjectid: 1.3.6.1.4.1.31770.*`
- `metadata.device.vendor: "bachmann"`
- `metadata.device.type: "pdu"`
- `metrics: [...]` (all SNMP symbols and tags)

### Orchestrator

python3 main.py will:

1. Run all generators from the GENERATORS list:

GENERATORS = [
("generate_inlet_profile.py", "bachmann_inlet_metrics.yaml"),
    ("generate_phase_profile.py", "bachmann_phase_metrics.yaml"),
    ("generate_outlet_per_phase_profile.py", "bachmann_outlet_metrics.yaml"),
    ("generate_io_profile_metrics.py", "bachmann_io_metrics.yaml"),
    ("generate_sensor_profile.py", "bachmann_sensors_metrics.yaml"),
    ("generate_phase_status_profile.py", "bachmann_phase_status.yaml"),
    ("generate_outlet_status_profile.py", "bachmann_outlet_status.yaml"),
    ("generate_inlet_status_profile.py", "bachmann_inlet_status.yaml"),
    ("generate_global_status_profile.py", "bachmann_global_status.yaml"),
]


2. Generate all .yaml files.

3. Create the parent profile bachmann_pdu.yaml using extends:

extends:
  - bachmann_inlet_metrics
  - bachmann_phase_metrics
  - bachmann_outlet_metrics
  - bachmann_io_metrics
  - bachmann_sensors_metrics
  - bachmann_phase_status
  - bachmann_outlet_status
  - bachmann_inlet_status
  - bachmann_global_status

---

## Requirements

- Python **3.8+** (anything reasonably recent should work)
---

## Usage

###  Generate all profiles

From the repository root:

```bash
python3 main.py
