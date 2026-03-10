# Auto Luci PIR + mmWave + Lux (Giorno/Notte) — Robusto

Blueprint professionale per Home Assistant: accensione/spegnimento luci basato su presenza PIR,
sensore mmWave opzionale e sensore di luminosità (lux), con profili giorno/notte intelligenti.

---

## Importazione in Home Assistant

Usa sempre il link **raw** di GitHub (non la pagina HTML):

```text
https://raw.githubusercontent.com/lucabio-cmyk/AutomazioneBluluci/main/blueprints/automation/automazione_bluluci/pir_lux_day_night.yaml
```

Se in Home Assistant vedi l'errore:

```text
mapping values are not allowed here
line 35, column 28: --tab-size-preference: 4;
```

stai importando la pagina HTML di GitHub (URL `github.com/.../blob/...`) invece del file YAML raw.
Usa sempre `raw.githubusercontent.com`.

---

## Funzionalità principali

### Sensori supportati
| Sensore | Tipo | Obbligatorio |
|---|---|---|
| PIR / presenza | `binary_sensor` | ✅ Sì |
| mmWave / radar (es. LD2410) | `binary_sensor` | No (opzionale) |
| Luminosità (lux) | `sensor` | ✅ Sì |
| Sole (`sun.sun`) | entità HA | Solo in modalità sole |

### Profili giorno/notte
Il blueprint include due profili distinti (luminosità, timeout, temperatura colore) e tre modalità
per determinare quando è "notte":

| Modalità | Descrizione |
|---|---|
| **Orario** (`time_based`) | Fascia oraria fissa configurabile (es. 22:30–06:30) |
| **Sole** (`sun_based`) | Tramonto/alba con offset in minuti |
| **Lux esterno** (`lux_based`) | 🆕 Usa il sensore lux direttamente — ideale per sensori crepuscolari |

#### Modalità lux esterno (crepuscolare) — consigliata con sensori di lux esterni
Se hai un sensore lux esterno (es. sul tetto, sul balcone o integrato nella stazione meteo),
la modalità `lux_based` è la scelta migliore:

- **Nessun orario da configurare**: il sistema si adatta automaticamente a stagioni e meteo
- **Reazione al meteo**: se arrivano nuvole temporalesche, entra in profilo notte anche di giorno
- **Tramonto reale**: non serve un offset per il sole, reagisce al lux effettivo

Parametro chiave: **Soglia lux per profilo notte** (default 50 lx). Se lux < soglia → profilo notte.

---

### 🆕 Accensione automatica al calo di lux (trigger crepuscolare)
Con **"Abilita accensione automatica al calo di lux"** attivo:

- Quando il sensore lux scende sotto la soglia giorno per il tempo di stabilità configurato
  (default 60 s), e la presenza è già rilevata dal PIR → le luci si accendono da sole
- Ideale per: essere seduti sul divano e non dover alzarsi quando cala il sole
- Il parametro **"Stabilità lux"** evita accensioni spurie per nuvole passeggere o ombre momentanee

> **Nota tecnica (mode: restart):** il trigger lux può interrompere un conto alla rovescia di
> spegnimento. Il blueprint include un gestore di sicurezza che ripristina lo spegnimento
> automatico se la stanza è vuota al momento del trigger. Il timeout di sicurezza è
> `min(timeout_attivo, 60 s)`.

---

### 🆕 Pre-spegnimento con dissolvenza (avviso visivo)
Con **"Abilita pre-spegnimento"** attivo:

1. Alla scadenza del timeout di assenza, la luce si abbassa alla luminosità configurata
   (default 10%) con transizione morbida
2. Dopo il tempo di avviso (default 15 s), la luce si spegne completamente
3. Se durante la fase di dimming viene rilevato nuovo movimento → le luci tornano alla piena
   intensità e il timer riparte normalmente

Utile in: bagno, studio, sala riunioni — evita spegnimenti improvvisi inaspettati.

---

### 🆕 Transizione morbida allo spegnimento
Il parametro **"Transizione allo spegnimento (s)"** aggiunge una dissolvenza graduale quando
le luci si spengono (anche nella fase pre-spegnimento). Default: 2 secondi.

---

### Sensore mmWave (radar)
Il sensore mmWave impedisce lo spegnimento automatico anche quando la persona è ferma
(es. seduta a leggere, guardare la TV) e il PIR non rileva più movimento.

- **Timeout mmWave**: timeout massimo di sicurezza per evitare che le luci restino accese
  indefinitamente in caso di falsi positivi del radar
- Opzione per disabilitare il timeout (luci accese finché il radar rileva presenza)

---

### Override manuale
Collegando un `input_boolean` come helper di override:
- Se l'helper è ON → logica automatica sospesa
- Opzione "Mantieni spegnimento auto durante override": anche in override, l'automazione
  può ancora spegnere le luci quando non c'è presenza

---

## Configurazione consigliata con sensore crepuscolare esterno

```yaml
# Esempio configurazione ottimale per sensore lux esterno
night_detection_mode: lux_based
lux_night_detection_threshold: 50   # lux: soglia crepuscolo (adattare al proprio sensore)
day_lux_threshold: 200              # lux: non accendere se c'è abbastanza luce diurna
night_lux_threshold: 15             # lux: di notte, accendi solo se è davvero buio
enable_lux_trigger: true            # accensione automatica al tramonto con presenza
lux_trigger_stability_seconds: 60  # il calo deve durare almeno 60s
enable_pre_off_dim: true            # avviso visivo prima dello spegnimento
pre_off_dim_brightness_pct: 10
pre_off_dim_duration_seconds: 15
off_transition_seconds: 3
enable_color_temp: true
day_color_temp_kelvin: 4000         # bianco neutro di giorno
night_color_temp_kelvin: 2700       # bianco caldo di notte
```

---

## Note sul campo `source_url`

Il campo `source_url` nel blueprint punta all'URL raw del file su GitHub.
Questo permette a Home Assistant di rilevare e proporre aggiornamenti automatici
del blueprint quando vengono rilasciate nuove versioni.
