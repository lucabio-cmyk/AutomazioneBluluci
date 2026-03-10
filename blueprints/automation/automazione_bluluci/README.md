# Import in Home Assistant da GitHub (fix errore YAML)

Se in Home Assistant vedi l'errore:

```text
mapping values are not allowed here
line 35, column 28: --tab-size-preference: 4;
```

stai quasi sicuramente importando la **pagina HTML di GitHub** (URL `github.com/.../blob/...`) invece del file YAML raw.

## URL corretto da usare in Home Assistant

Usa sempre il link `raw.githubusercontent.com`, ad esempio:

```text
https://raw.githubusercontent.com/<USER_O_ORG>/AutomazioneBluluci/main/blueprints/automation/automazione_bluluci/pir_lux_day_night.yaml
```

Sostituisci `<USER_O_ORG>` con il tuo utente/organizzazione GitHub.

## Nota sul blueprint

Nel file blueprint il campo `source_url` deve puntare allo stesso URL raw pubblico del file, così Home Assistant può gestire correttamente anche gli aggiornamenti.
