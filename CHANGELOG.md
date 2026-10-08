# Changelog

Alle belangrijke wijzigingen aan dit project worden in dit bestand vastgelegd.

## [1.1.2] - 2026-10-08

### Opgelost

- voorkomt `value_json is undefined`-meldingen door geen lege retained state meer te publiceren;
- houdt MQTT-entiteiten tijdens het opstarten offline tot de eerste verse JSON-state is gepubliceerd;
- maakt MQTT Discovery-templates bestand tegen lege of niet-JSON state-berichten.

## [1.1.1] - 2026-10-02

### Toegevoegd

- volledige Engelse README naast de Nederlandse documentatie;
- Engelse voorbeeldconfiguraties;
- Engelse dashboardvariant;
- taalkeuze bovenaan beide README-bestanden.

## [1.1.0] - 2026-10-02

### Toegevoegd

- automatische herkenning van A, mA, V, mV, W, kW, MW, VA, kVA en MVA;
- optionele multipliers als override voor foutieve of ontbrekende HA-units;
- configuratieoverzicht per fase in het AppDaemon-log;
- duidelijke foutmelding wanneer een fase geen stroom- of vermogensbron heeft;
- openbare voorbeeldconfiguraties, dashboard en GitHub-testworkflow.

### Gewijzigd

- active en apparent power worden intern eerst naar W/VA genormaliseerd;
- testset uitgebreid naar automatische eenheidsconversie.

## [1.0.0] - 2026-09-26

- eerste zelfstandige versie van Phase Peak Guard;
- fasebewaking, voorspellingen, headroom, fuse stress en onbalans;
- MQTT Discovery en begrensde persistente opslag;
- uitsluitend monitoren; geen load shedding.
