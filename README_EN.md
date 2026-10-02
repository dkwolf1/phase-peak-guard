[Nederlands](README.md) | [English](README_EN.md)

# 3x25A Phase Guard for Home Assistant

An independent AppDaemon app that monitors and analyzes a three-phase household
electrical connection. It primarily monitors **current per phase** and publishes
the results as a single MQTT Discovery device in Home Assistant.

The app is monitoring and advisory only. It never controls switches and has no
dependency on a PV controller or any other AppDaemon app.

## Main features

- direct phase-current measurement with power-based fallbacks;
- import, export and neutral direction detection;
- EMA plus 1, 5 and 15-minute rolling averages;
- bounded 30, 60 and 120-second predictions;
- remaining current headroom per phase;
- continuous and daily time above configurable thresholds;
- heuristic thermal/fuse stress indicator;
- real-time and structural phase imbalance analysis;
- daily, 7-day and 30-day statistics;
- explicit `data_error` when reliable data is unavailable;
- MQTT Discovery under one Home Assistant device;
- atomic JSON storage with a configurable hard size limit.

## Requirements

- Home Assistant;
- AppDaemon with access to Home Assistant;
- the Home Assistant MQTT integration and a working broker;
- Python 3.9 or newer in the AppDaemon environment;
- three phase sources: preferably current, otherwise apparent or active power.

## Quick installation

1. Copy `phase_peak_guard.py` into your AppDaemon apps directory.
2. Copy `apps.yaml.example.en` or add its `phase_peak_guard:` block to your
   existing `apps.yaml`.
3. In Home Assistant, open **Developer Tools → States**.
4. Find the exact entity IDs for L1, L2 and L3 and replace the placeholders.
5. Set `storage_file` to a writable path inside your AppDaemon container.
6. Restart the app or AppDaemon.
7. Check the AppDaemon log, then open the MQTT device `3x25A Phase Guard`.

Minimal configuration:

```yaml
phase_peak_guard:
  module: phase_peak_guard
  class: PhasePeakGuard

  current_l1: sensor.REPLACE_ME_L1_CURRENT
  current_l2: sensor.REPLACE_ME_L2_CURRENT
  current_l3: sensor.REPLACE_ME_L3_CURRENT

  mqtt_enabled: true
  storage_file: /homeassistant/python_storage/phase_peak_guard.json
```

Values containing `REPLACE_ME` are deliberately ignored as entity IDs. The app
logs which phase does not yet have a usable current or power source.

## Selecting source entities

For each phase, the app tries the following sources in order:

1. `current_l1/l2/l3` — direct current measurement;
2. `apparent_power_l1/l2/l3` — calculated with `I = VA / V`;
3. `net_power_l1/l2/l3` — estimate based on active power;
4. separate `import_power_*` and `export_power_*` entities — active-power
   fallback and direction detection.

Voltage entities are optional. If no valid voltage is available, the app uses
`nominal_voltage`, which defaults to 230 V.

| Configuration key | Expected quantity | Required |
|---|---|---|
| `current_l1/l2/l3` | A or mA | Preferred source |
| `voltage_l1/l2/l3` | V or mV | No |
| `apparent_power_l1/l2/l3` | VA or kVA | Fallback only |
| `net_power_l1/l2/l3` | W, kW or MW | Fallback only |
| `import_power_l1/l2/l3` | W, kW or MW | Optional |
| `export_power_l1/l2/l3` | W, kW or MW | Optional |

## Automatic unit detection

Version 1.1 reads `unit_of_measurement` from Home Assistant and recognizes:

- current: `A`, `mA`;
- voltage: `V`, `mV`;
- active power: `W`, `kW`, `MW`;
- apparent power: `VA`, `kVA`, `MVA`.

If an integration does not provide a correct unit, use an explicit override:

```yaml
active_power_multiplier: 1000
```

An explicit multiplier takes precedence over automatic unit detection.

## Included examples

- `examples/direct-current.en.yaml`: recommended direct-current setup;
- `examples/power-fallback.en.yaml`: setup without current sensors;
- `apps.yaml.example.en`: fully documented configuration;
- `dashboards/phase_guard_dashboard_en.yaml`: optional English dashboard;
- the files without `.en` contain the Dutch variants.

## Important settings

| Setting | Default | Meaning |
|---|---:|---|
| `service_fuse_amp` | 25.0 | Nominal fuse rating per phase |
| `sample_interval_sec` | 5 | Sampling interval, constrained to 2–60 s |
| `ema_window_sec` | 30 | EMA time window |
| `threshold_elevated_amp` | 17.5 | Elevated load threshold |
| `threshold_high_amp` | 20.0 | High load threshold |
| `threshold_critical_amp` | 22.5 | Critical load threshold |
| `threshold_limit_amp` | 25.0 | Configured limit |
| `max_source_age_sec` | 30 | Maximum source age |
| `max_plausible_current_amp` | 80 | Plausibility limit |
| `max_storage_mb` | 5 | Hard JSON write limit |

## MQTT

The default retained topics are:

```text
home/phase_guard/state
home/phase_guard/availability
```

All discovered entities belong to one Home Assistant device. For a second
installation, configure a unique `mqtt_device_id` and `mqtt_base_topic`.

## Storage

Rolling real-time samples are bounded in memory. Daily and hourly statistics are
written atomically through a temporary file and `os.replace()`. Before writing,
the app measures the complete JSON payload. Old hourly and then daily history is
removed as needed to stay below `max_storage_mb`.

Do not publish your storage file: it contains household load history.

## Dashboard

`dashboards/phase_guard_dashboard_en.yaml` is a complete Sections view. Import it
after MQTT Discovery has created the Phase Guard entities. The configured
`/local/achtergrond.jpg` background is optional.

## Troubleshooting

### Status remains `data_error`

- verify the exact entity IDs in Developer Tools;
- verify that every state is numeric;
- check each entity's `unit_of_measurement`;
- read the `PhaseGuard configuration L1/L2/L3` messages in the AppDaemon log;
- make sure the source is not genuinely `unavailable`.

### A phase at 0 A is marked stale

Keep `shared_source_freshness: true` when all phase entities originate from the
same smart meter. A genuinely unavailable value is still rejected.

### MQTT entities do not appear

Verify that AppDaemon can call `mqtt/publish`, the MQTT integration is active,
and `mqtt_enabled: true` is configured.

## Tests

The tests do not require AppDaemon:

```bash
python -m unittest discover -s tests -v
```

They cover thresholds, predictions, outliers, stale data, unit conversion,
MQTT-only service usage and enforcement of the storage-size limit.

## Safety

This app does not replace service fuses, circuit breakers, residual-current
protection, correct cable sizing or a qualified electrician. `NORMAL` only means
that the available Home Assistant data does not indicate a high phase load. The
fuse-stress value is a heuristic indicator, not an IEC fuse-curve simulation.

## License

MIT — see `LICENSE`.
