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

## Configurazione (organizzata in sezioni)

Per ridurre la confusione, i parametri sono raggruppati in **sezioni collassabili**.
All'apertura vedi solo i **3 essenziali**; il resto è nascosto finché non ti serve:

| In vista subito | Sezioni collassabili (aprile solo se ti servono) |
|---|---|
| Sensore di movimento · Sensore lux · Luci target | 🛰️ Sensori aggiuntivi · ⚙️ Comportamento generale · 🌗 Giorno/Notte · 💡 Accensione & Luminosità · 🌙 Spegnimento · 🎨 Temperatura colore · 🎬 Modalità scene |

**Setup minimo per partire:** imposta i 3 essenziali e lascia tutto il resto ai valori di default.
Le luci si accendono al movimento (se il lux è sotto soglia) e si spengono dopo il timeout di assenza.

---

## Spegnimento quando c'è abbastanza luce naturale

Con **"Spegni quando c'è abbastanza luce naturale"** attivo (sezione 🌙 Spegnimento):

- Quando il sensore lux **sale sopra** la `Soglia lux spegnimento` per il tempo di stabilità
  configurato, le luci si spengono **anche se c'è presenza** — di giorno c'è già abbastanza
  luce diurna, quindi si risparmia energia.
- Se poi torna il buio e sei in stanza, con `Abilita accensione automatica al calo di lux`
  attivo le luci si **riaccendono da sole**.

> ⚠️ **Isteresi obbligatoria.** Imposta `Soglia lux spegnimento` **≥** `Soglia lux giorno`.
> Serve una banda morta tra la soglia che accende e quella che spegne, altrimenti le luci
> oscillano on/off. Esempio sicuro: giorno = 155 lx, spegnimento = 250 lx.

Questo sostituisce automazioni esterne tipo "spegni la sala se c'è il sole": gestisci la luce
in un solo posto, evitando che due automazioni si contendano la stessa lampada.

---

## Troubleshooting

### Le luci NON si accendono quando dovrebbero
1. **Controllo lux troppo restrittivo.** Con "Abilita controllo lux in accensione" attivo, la luce
   si accende solo se `lux < soglia`. In sezione **💡 Accensione**, alza `Soglia lux giorno` (o
   `Soglia lux notte`) — se è più bassa del lux reale in stanza, la luce non parte mai.
2. **Sensore lux "unavailable".** Se il tuo sensore va spesso offline, il valore di fallback
   `Lux fallback` (default 999) blocca l'accensione. Se ti capita, **abbassalo** sotto la soglia
   giorno (es. 0) così un sensore assente non impedisce l'accensione.
3. **Profilo notte inatteso.** Se `is_night` risulta vero quando non te lo aspetti (es. lux_based
   con soglia notte troppo alta), viene usata la `Soglia lux notte`, di solito più bassa. Verifica
   la modalità di rilevamento notte in **🌗 Giorno/Notte**.
4. **Debounce movimento.** Con `Debounce movimento ON` > 0, il PIR deve restare ON per quel tempo:
   con sensori a impulso breve il trigger può non scattare mai. Tienilo a 0 salvo necessità.

### Le luci RESTANO accese / non si spengono
1. **mmWave "incollato" su ON.** I radar (LD2410 & simili) danno spesso falsi positivi: finché
   rilevano "presenza" la luce resta accesa. Assicurati che **"Abilita timeout massimo mmWave"**
   sia attivo (sezione 🛰️) — senza timeout la luce resta accesa a tempo indeterminato.
2. **Timeout di assenza troppo lungo.** In **🌙 Spegnimento**, `Timeout assenza giorno/notte`
   è il tempo dopo l'ultimo movimento prima di spegnere. Abbassalo se lo spegnimento tarda.
3. **Movimento sempre rilevato.** Se il PIR resta ON (riflessi, tende in movimento, animali),
   il countdown di spegnimento non parte mai. Verifica lo storico del `binary_sensor`.
4. **Secondo sensore open space.** Con due sensori di movimento lo spegnimento richiede che
   **entrambi** siano liberi (logica AND). Se uno resta ON, la luce non si spegne.

### Troppe opzioni
Ignora le sezioni collassate: servono solo per le funzioni avanzate (circadiano, adattivo,
pre-spegnimento, scene). Con i soli 3 parametri essenziali il blueprint funziona già.

---

## Funzionalità principali

### Sensori supportati
| Sensore | Tipo | Obbligatorio |
|---|---|---|
| PIR / presenza | `binary_sensor` | ✅ Sì |
| mmWave / radar (es. LD2410) | `binary_sensor` | No (opzionale) |
| Luminosità (lux) | `sensor` | ✅ Sì |
| Sensore crepuscolare esterno (lux) | `sensor` | No (opzionale, consigliato con `lux_based`) |
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

- Puoi impostare un **sensore crepuscolare dedicato** per decidere quando è notte,
  separato dal sensore lux usato per l'accensione/spegnimento

- **Nessun orario da configurare**: il sistema si adatta automaticamente a stagioni e meteo
- **Reazione al meteo**: se arrivano nuvole temporalesche, entra in profilo notte anche di giorno
- **Tramonto reale**: non serve un offset per il sole, reagisce al lux effettivo

Parametro chiave: **Soglia lux per profilo notte** (default 50 lx). Se lux < soglia → profilo notte.
Se valorizzi **Sensore crepuscolare esterno**, il confronto viene fatto su quel sensore;
altrimenti viene usato il sensore lux principale.

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

### 🆕 Luminosità adattiva al lux
Con **"Abilita luminosità adattiva al lux"** attivo:

- La luminosità si calcola dinamicamente in base al rapporto lux/soglia del profilo attivo
- **Buio totale (lux ≈ 0)** → luminosità al massimo del profilo (es. 85% giorno, 25% notte)
- **Lux vicino alla soglia** → luminosità al minimo configurato (default 20%)
- Interpolazione lineare continua: la luce "segue" il calo di luce naturale
- Utile in ambienti con luce variabile (finestre, tramonto graduale, nuvole)
- Funziona solo con modalità **"Controllo luminosità"** (non con le scene)
- Richiede che il controllo lux sia abilitato e il sensore lux sia valido

> **Esempio:** soglia giorno = 200 lx, luminosità max = 85%, minimo = 20%.
> Con lux = 100 (metà soglia): luminosità = 20% + (85%-20%) × (1 - 0.5) = **52%**
> Con lux = 10 (quasi buio): luminosità ≈ **83%**

---

### 🆕 Temperatura colore circadiana (transizione continua)

Con **"Abilita temperatura colore circadiana"** attivo (richiede anche "Abilita controllo temperatura colore"):

- La temperatura colore non passa bruscamente da 4000K a 2700K, ma **varia in modo continuo
  e graduale** in base all'ora del giorno, all'elevazione del sole o al lux esterno
- Perfetto per ambienti domestici, camere da letto e uffici dove si vuole un'illuminazione
  naturale e rilassante che accompagni il ritmo circadiano
- Tre modalità di interpolazione, una per ciascun metodo di rilevamento notte:

| Modalità | Logica di interpolazione |
|---|---|
| **Orario** | Transizione lineare centrata su inizio/fine notte, durata configurabile |
| **Sole** | Segue l'elevazione solare: ≥15° → freddo, tra -6° e 15° → interpolato, ≤-6° → caldo |
| **Lux esterno** | Interpola tra soglia lux giorno (bianco freddo) e soglia lux notte (bianco caldo) |

#### Modalità oraria — esempio con `circadian_transition_minutes: 60`

```
Ore 21:00          22:00        22:30        23:00         ...
    │                │            │            │
    └──── 4000 K ────┤ 4000→2700K │← 2700 K──────────────
                     │   (30min)  │(30min)     │
                 inizio trans.  night_start fine trans.
```

Specularmente al mattino, la transizione inversa avviene centrata su `night_end_time`.

#### Modalità sole — curva basata sull'elevazione

```
Elevazione solare:   > 15°   15° → -6°    < -6°
Temperatura:          4000 K  interpolata  2700 K
                      (giorno)  (alba/tramonto) (notte)
```

#### Modalità lux — risposta al lux esterno in tempo reale

```
lux ≥ soglia_giorno (es. 200 lx)  → 4000 K (bianco freddo)
lux ≤ soglia_notte  (es. 50 lx)   → 2700 K (bianco caldo)
lux tra le due soglie              → interpolazione lineare
```

> **Aggiornamento in tempo reale:** la temperatura viene ricalcolata ogni volta che
> la luce si accende (a seguito di rilevamento movimento o trigger lux). Per aggiornare
> la temperatura in modo continuo mentre sei già in stanza, puoi creare una seconda
> automazione separata che chiama `light.turn_on` con solo `color_temp_kelvin` ogni
> 10–15 minuti, condizionata a: presenza rilevata + luci accese.

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
- **Doppio controllo mmWave**: il sensore viene verificato sia prima della fase di pre-dimming
  sia subito dopo — se il radar torna attivo durante il dimming (senza che il PIR si riattivi),
  il sistema aspetta nuovamente che la presenza si liberi prima di spegnere

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
enable_adaptive_brightness: true    # luminosità adattiva: più buio = più luce
adaptive_brightness_min_pct: 20     # minimo 20% anche con lux quasi a soglia
enable_pre_off_dim: true            # avviso visivo prima dello spegnimento
pre_off_dim_brightness_pct: 10
pre_off_dim_duration_seconds: 15
off_transition_seconds: 3
enable_color_temp: true
day_color_temp_kelvin: 4000         # bianco neutro di giorno
night_color_temp_kelvin: 2700       # bianco caldo di notte
enable_circadian_color_temp: true   # transizione continua (no salto brusco giorno/notte)
# Con lux_based, la transizione segue il lux in tempo reale → nessun parametro di durata
```

---

## Note sul campo `source_url`

Il campo `source_url` nel blueprint punta all'URL raw del file su GitHub.
Questo permette a Home Assistant di rilevare e proporre aggiornamenti automatici
del blueprint quando vengono rilasciate nuove versioni.
