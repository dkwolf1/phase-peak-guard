[Nederlands](README.md) | [English](README_EN.md)

# 3x25A Phase Guard for Home Assistant

De actuele Nederlandse view staat in
[`dashboards/phase_guard_dashboard.yaml`](dashboards/phase_guard_dashboard.yaml).
Alle `phase_guard_*`-entiteiten in de view komen van deze app via MQTT Discovery.
Vul zelf drie fasebronnen in `apps.yaml` in; de view heeft verder geen eigen
entiteiten of extra kaarten nodig. De volledige entiteitenhandleiding staat in
de [PV Energy Suite-dashboarddocumentatie](https://github.com/dkwolf1/PV-blueprint/blob/main/dashboards/README.md).

Zelfstandige AppDaemon-app voor het bewaken en analyseren van een driefasige
huisaansluiting. De app bewaakt primair de **stroom per fase** en publiceert de
resultaten als één MQTT Discovery-device in Home Assistant.

De app monitort en adviseert uitsluitend. Er worden geen switches bediend en er
is geen koppeling met een PV-regelaar of andere AppDaemon-app.

De app kan als zelfstandig onderdeel van de PV Energy Suite worden gebruikt.
Zijn MQTT-headroomsensoren zijn beschikbaar voor dashboards. Zonder betrouwbare
fase-toewijzing van apparaten gebruikt de verbruiksplanner die headroom niet
als automatische startvoorwaarde; de guard blijft volledig onafhankelijk.

## Belangrijkste functies

- directe stroommeting per fase, met vermogensfallback;
- import-, export- en neutraalrichting;
- EMA en gemiddelden over 1, 5 en 15 minuten;
- begrensde voorspellingen over 30, 60 en 120 seconden;
- headroom tot de ingestelde hoofdzekering;
- tijd boven verhoogde, hoge, kritieke en limietwaarden;
- heuristische thermische/zekeringbelasting;
- actuele en structurele fase-onbalans;
- dag-, 7-daagse en 30-daagse statistiek;
- expliciete `data_error` bij onvoldoende betrouwbare data;
- MQTT Discovery onder één Home Assistant-device;
- atomaire JSON-opslag met een configureerbare maximumgrootte.

## Vereisten

- Home Assistant;
- AppDaemon met toegang tot Home Assistant;
- de Home Assistant MQTT-integratie en een werkende broker;
- Python 3.9 of nieuwer in de AppDaemon-omgeving;
- drie fasebronnen: bij voorkeur stroom, anders apparent of active power.

## Snel installeren

1. Kopieer `phase_peak_guard.py` naar de map waarin AppDaemon je apps laadt.
2. Kopieer `apps.yaml.example` naar je AppDaemon-configuratie of neem alleen het
   blok `phase_peak_guard:` over.
3. Open in Home Assistant **Ontwikkelaarstools → Statussen**.
4. Zoek de stroomentiteiten voor L1, L2 en L3 en vul hun exacte entity-ID's in.
5. Kies voor `storage_file` een schrijfbare locatie in je AppDaemon-container.
6. Herstart de app of AppDaemon.
7. Controleer het AppDaemon-log en daarna het MQTT-device `3x25A Phase Guard`.

Minimale configuratie:

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

Waarden met `REPLACE_ME` worden bewust niet als echte entity-ID gebruikt. De app
meldt in het log welke fase nog geen bruikbare bron heeft.

## Welke entiteiten moet ik invullen?

De app probeert per fase deze volgorde:

1. `current_l1/l2/l3` — directe stroommeting;
2. `apparent_power_l1/l2/l3` — berekening `I = VA / V`;
3. `net_power_l1/l2/l3` — schatting uit actief vermogen;
4. afzonderlijke `import_power_*` en `export_power_*` — actieve fallback en
   richtingsbepaling.

Voltage is optioneel. Zonder geldige spanningssensor gebruikt de app
`nominal_voltage`, standaard 230 V.

| Configuratiesleutel | Verwachte grootheid | Verplicht |
|---|---|---|
| `current_l1/l2/l3` | A of mA | Voorkeursroute |
| `voltage_l1/l2/l3` | V of mV | Nee |
| `apparent_power_l1/l2/l3` | VA of kVA | Alleen als fallback |
| `net_power_l1/l2/l3` | W, kW of MW | Alleen als fallback |
| `import_power_l1/l2/l3` | W, kW of MW | Optioneel |
| `export_power_l1/l2/l3` | W, kW of MW | Optioneel |

## Automatische eenheidsherkenning

Versie 1.1 leest `unit_of_measurement` uit Home Assistant en ondersteunt:

- stroom: `A`, `mA`;
- spanning: `V`, `mV`;
- actief vermogen: `W`, `kW`, `MW`;
- apparent power: `VA`, `kVA`, `MVA`.

Als een integratie geen of een verkeerde unit publiceert, kan een multiplier
handmatig worden ingesteld:

```yaml
active_power_multiplier: 1000
```

Hiermee wordt bijvoorbeeld een kW-waarde naar W omgerekend. Een expliciete
multiplier heeft voorrang op automatische detectie.

## Voorbeelden

- `examples/direct-current.yaml`: aanbevolen configuratie met drie
  stroomsensoren;
- `examples/power-fallback.yaml`: configuratie zonder directe stroomsensoren;
- `apps.yaml.example`: volledig gedocumenteerde configuratie;
- `dashboards/phase_guard_dashboard.yaml`: optioneel Lovelace-dashboard.

## Belangrijke instellingen

| Instelling | Standaard | Betekenis |
|---|---:|---|
| `service_fuse_amp` | 25.0 | Nominale fasezekering |
| `sample_interval_sec` | 5 | Meetinterval, begrensd op 2–60 s |
| `ema_window_sec` | 30 | Tijdvenster van de EMA |
| `threshold_elevated_amp` | 17.5 | Verhoogde belasting |
| `threshold_high_amp` | 20.0 | Hoge belasting |
| `threshold_critical_amp` | 22.5 | Kritieke belasting |
| `threshold_limit_amp` | 25.0 | Ingestelde limiet |
| `max_source_age_sec` | 30 | Maximale bronleeftijd |
| `max_plausible_current_amp` | 80 | Plausibiliteitsgrens |
| `max_storage_mb` | 5 | Harde schrijflimiet voor JSON-opslag |

## MQTT

Standaard worden gebruikt:

```text
home/phase_guard/state
home/phase_guard/availability
```

De state is één retained JSON-bericht. Alle ontdekte entiteiten horen bij
hetzelfde Home Assistant-device. Gebruik voor een tweede installatie een unieke
`mqtt_device_id` en `mqtt_base_topic`.

## Opslag

Rolling realtime-samples zijn begrensd in RAM. Dag- en uurstatistiek wordt
atomair opgeslagen via een tijdelijk bestand en `os.replace()`. Voor het schrijven
wordt de complete JSON-payload gemeten. Oude uur- en daarna daghistorie wordt
verwijderd wanneer dat nodig is om onder `max_storage_mb` te blijven.

Publiceer het opslagbestand niet: het bevat belastinghistorie van de woning.

## Dashboard

`dashboards/phase_guard_dashboard.yaml` is een complete Sections-view. Plak de
view in een handmatig dashboard nadat MQTT Discovery de Phase Guard-entiteiten
heeft aangemaakt. De gebruikte achtergrond `/local/achtergrond.jpg` is optioneel.

## Problemen oplossen

### Status blijft `data_error`

- controleer de entity-ID's in Ontwikkelaarstools;
- controleer of de state numeriek is;
- controleer `unit_of_measurement`;
- lees de regels `PhaseGuard configuratie L1/L2/L3` in het AppDaemon-log;
- controleer of de sensor niet werkelijk `unavailable` is.

### Een fase met 0 A wordt stale genoemd

Laat `shared_source_freshness: true` staan wanneer alle fase-entiteiten van
dezelfde slimme meter komen. Een echte `unavailable`-waarde wordt nog steeds
afgekeurd.

### Geen MQTT-entiteiten

Controleer of AppDaemon `mqtt/publish` via Home Assistant mag aanroepen en of de
MQTT-integratie actief is. Controleer ook of `mqtt_enabled: true` staat.

## Testen

Zonder AppDaemon-installatie:

```bash
python -m unittest discover -s tests -v
```

De tests simuleren AppDaemon en controleren onder andere grenzen, voorspellingen,
outliers, stale data, eenheidsconversie, MQTT-only servicegebruik en opslaglogica.

## Veiligheid

Deze app is geen vervanging voor hoofdzekeringen, automaten, aardlekbeveiliging,
kabeldimensionering of een beoordeling door een elektricien. `NORMAL` betekent
alleen dat de beschikbare Home Assistant-data geen hoge fasebelasting aangeeft.
De zekeringbelasting is een heuristische indicator en geen IEC-zekeringcurve.

## Licentie

MIT — zie `LICENSE`.
