# Task 5 — Verifica SPICE

Simulazione ngspice del netlist `electret2xlr.zen` (Task 4): punto di lavoro DC
e risposta AC differenziale P2→(P2−P3), rispetto ai criteri del brief.

## Setup di simulazione

### Toolchain

- `pcb` (pcbc 0.4.7) — nessun cambiamento necessario.
- ngspice: **non presente come binario** nel sandbox (`which ngspice` → non
  trovato), solo la libreria `libngspice0` di sistema. `pcb sim` invoca pero'
  il binario `ngspice` come processo esterno (`crates/pcb-sim/src/ngspice.rs`),
  non la libreria via FFI. Risolto **senza root**: `apt-get download ngspice`
  (fetch del `.deb`, non richiede privilegi) + `dpkg-deb -x` per estrarre
  `usr/bin/ngspice` in `~/.local/bin/ngspice` (tutte le librerie condivise
  richieste erano gia' presenti). Il binario e' stato usato impostando la
  variabile d'ambiente `NGSPICE=~/.local/bin/ngspice` richiesta da
  `pcb sim` per un percorso non standard. Nessuna modifica di sistema, nessun
  pacchetto installato con `apt install`.

### Modelli SPICE mancanti

Il primo `pcb sim --setup "..."` di controllo ha confermato l'ipotesi del
brief: il registry diode non e' raggiungibile in questo sandbox, quindi Q1,
Q2, Q3 (autorati come `Component()` primitivi da simboli KiCad) non avevano
`spice_model`. In piu', anche J1 (XLR3), J2 (jack 3.5mm) e i due `TestPoint`
(TP1/TP2) risultavano privi di modello (il tool richiede un modello SPICE per
*ogni* componente del netlist, connettori/testpoint inclusi).

Modelli aggiunti direttamente in `electret2xlr.zen`:

- **Q1 (MMBT3904, NPN)** e **Q2/Q3 (MMBT3906, PNP)**: parametri SPICE
  jellybean ben noti (dataset Fairchild/onsemi per 2N3904/2N3906,
  elettricamente equivalenti al die MMBT390x in SOT-23), in
  `spice/Transistors.lib`, subcircuit `Q_NPN_MMBT3904` / `Q_PNP_MMBT3906`.
- **D1 (zener 8.2V)**: gia' coperto dallo stdlib (`Zener.zen` espone
  `spice_model` con `BV` dal parametro `zener_voltage`) — nessuna modifica
  necessaria.
- **J1, J2 (connettori)**: modello "pass-through" vuoto
  (`spice/Connector.lib`, subcircuit `CONN3`, nessun elemento interno). Sono
  parti puramente meccaniche; nel testbench lo stimolo (fantasma P48, segnale
  capsula) e' iniettato esattamente dove questi connettori farebbero da
  interfaccia (P2/P3 per J1, MICIN per J2), quindi un modello vuoto è corretto
  e non altera nessuna delle grandezze verificate.
- **TP1, TP2 (TestPoint stdlib)**: lo stdlib `TestPoint.zen` non espone un
  parametro `spice_model`. Soluzione: nuovo config booleano
  `sim_mode = config(bool, default=False, optional=True)` in cima al file;
  `TestPoint(..., dnp=sim_mode)`. `dnp` e' un kwarg universale che il
  framework Zener inietta in qualunque `Module()`/`Component()` a prescindere
  dal suo `config()` dichiarato (verificato leggendo
  `pcb-zen-core/src/lang/module.rs`), quindi funziona senza modificare lo
  stdlib. Con `sim_mode=False` (default, `pcb build` normale) i test point
  restano popolati come sempre; solo `pcb sim --config sim_mode=true` li
  esclude dal netlist SPICE. **Verificato** che `pcb build electret2xlr.zen`
  (senza `--config`) produce ancora tutti e 32 i componenti, incluse TP1/TP2.

### Sorgenti di stimolo (in `testbench/setup_dc_ac.cir`, passato con `--setup`)

- **Fantasma P48**: `V48` 48V DC, `R_P2`/`R_P3` 6.8kΩ verso P2/P3 (come da
  brief).
- **Capsula electret**: sorgente di corrente `I_MIC` su MICIN, `DC 0.5m`
  (sink, per convenzione SPICE `I n+ n- val` estrae corrente da n+) `AC 1`
  (ampiezza unitaria per l'analisi `.ac`, il valore assoluto è arbitrario:
  conta solo la forma della risposta in dB).
- **Bleed SHELL→GND (1GΩ, solo testbench)**: SHELL è collegato solo a CSH
  verso GND — sulla board reale il nodo prosegue meccanicamente nella calotta
  del connettore XLR (non è un pin logico di J1). Senza J1 nel netlist di
  simulazione resterebbe un nodo isolato dietro un condensatore ideale
  (matrice singolare in analisi DC — errore `check node xc11.mid2`, dove
  `xc11` è l'istanza SPICE di CSH). Un resistore da 1GΩ elimina l'artefatto
  senza alterare nessun criterio (non è nel netlist di produzione).

Comando usato:

```
NGSPICE=~/.local/bin/ngspice pcb sim electret2xlr.zen \
  --config sim_mode=true --setup testbench/setup_dc_ac.cir -v
```

Nota sul tool: `pcb sim <path relativo>` fallisce con
"No such file or directory (os error 2)" durante lo spawn di ngspice, perché
`zen_path.parent()` su un nome file senza directory (es. `electret2xlr.zen`)
ritorna `Some("")` invece di `None`, e `current_dir("")` fa fallire lo spawn.
Usare sempre un percorso assoluto (o con almeno una directory) per `pcb sim`.

## Risultati DC (operating point)

| Grandezza | Valore simulato | Criterio | Esito |
|---|---|---|---|
| V(VPIP) | 7.486 V | 7.0 – 8.0 V | **PASS** |
| V(MICOUT) | 4.086 V | 3 – 5 V | **PASS** |
| I da P2 (attraverso R_P2 6.8k) | 2.996 mA | 2.5 – 3.5 mA | **PASS** |
| I da P3 (attraverso R_P3 6.8k) | 3.131 mA | 2.5 – 3.5 mA | **PASS** |
| Sbilanciamento I(P2) vs I(P3) | 4.4 % | < 10 % | **PASS** |
| V(VREF) (informativo) | 8.134 V | — | zener 8.2V − caduta R9, coerente |
| V(P2) / V(P3) (informativo) | 27.63 V / 26.71 V | — | 48V − I·6.8k, coerente |

## Risultati AC (risposta differenziale P2−P3)

Sweep `.ac dec 50 0.01 100k`, riferimento a 1kHz = 75.41 dB.

| Frequenza | Guadagno (dB) | Nota |
|---|---|---|
| 20 Hz | 69.90 | ancora in salita, sotto il ginocchio |
| 30 Hz | 72.30 | ≈ punto −3dB |
| 31 Hz | 72.47 | |
| 60 Hz | 74.95 | |
| 100 Hz | 75.65 | inizio banda piatta richiesta |
| 1 kHz | 75.41 | riferimento |
| 5 kHz | 75.39 | |
| 10 kHz | 75.36 | |
| 20 kHz | 75.27 | fine banda piatta richiesta |

| Criterio | Misura | Esito |
|---|---|---|
| Piattezza 100Hz–20kHz (±1dB) | max−min = 75.65−75.27 = **0.38 dB** | **PASS** |
| −3dB inferiore tra 20 e 60Hz | crossing (75.41−3=72.41dB) interpolato tra 30Hz (72.30) e 31Hz (72.47) → **≈ 30.6 Hz** | **PASS** |
| Fase (nota, non criterio pass/fail) | φ(P2−P3) @1kHz = **−179.1°** (~180°, invertente) | vedi nota sotto |

Dati completi del barrido esportati in `testbench/output/ac_response.txt`
(351 punti, 0.01Hz–100kHz, `wrdata`).

### Nota sulla fase (issue #6 del progetto p48 originale)

La fase misurata a 1kHz è ≈ −179°, cioè l'uscita differenziale P2−P3 è
sostanzialmente **invertita** rispetto al segno convenzionale P2−P3 atteso
(equivalente a dire che il segnale reale è in fase su P3−P2). Questo è
esattamente il comportamento segnalato nell'issue #6 del progetto originale
p48-pip-adapter citato nel brief: la capsula pilota lo stadio "caldo" (Q2→P2)
mentre lo stadio "freddo" (Q3→P3) è di solo riferimento AC-a-massa; a seconda
di come si definisce il segno differenziale, il risultato può apparire
invertito. **Non è stato corretto in questo task**: l'eventuale rimedio
(scambiare gli accoppiamenti C2/C3, o più precisamente la topologia
hot/cold) è una decisione da verificare al banco, non a tavolino — come
indicato esplicitmente dal brief.

## Cambio di valore: CB2/CB3 da 22µF a 1µF

**Motivo:** con i valori originali (CB2=CB3=22µF, dal Task 3/4), la
simulazione AC iniziale mostrava un ginocchio passa-alto molto più basso del
richiesto: la risposta P2−P3 era già piatta entro ~0.5dB fino a 5Hz, con il
vero punto −3dB (rispetto al riferimento 1kHz) situato tra 1 e 2 Hz — ben al
di sotto della finestra 20–60Hz richiesta dal criterio di accettazione.

**Causa elettrica:** la rete R1(7.5k) in parallelo con CB2+RB2(47Ω) tra
l'emettitore di Q2 e P2 (idem R2/CB3/RB3 sul lato Q3/P3) forma un partitore
di tensione verso il carico R_P2/R_P3 (6.8k) la cui impedenza equivalente
transita da R1 (bassa frequenza, condensatore aperto) a R1‖RB2 (alta
frequenza) con ginocchio inferiore a f₁ ≈ 1/(2π·CB2·(R1+RB2)). Con
CB2=22µF, f₁ ≈ 0.96Hz — troppo basso.

**Correzione:** CB2 e CB3 ridotti insieme (per mantenere il bilanciamento tra
i due inseguitori appaiati) da 22µF a **1µF** (X7R, 1206, 50V — package/
dielettrico invariati, valore molto comune, nessun problema di reperibilità).
Questo sposta f₁ a ≈ 1µF/22µF × 0.96Hz ≈ 21Hz nominale; il valore
effettivamente misurato in simulazione (con tutto il resto del circuito
incluso) è ≈30.6Hz, centrato comodamente nella finestra 20–60Hz richiesta,
senza intaccare la piattezza 100Hz–20kHz (0.38dB, ben entro ±1dB) né il punto
di lavoro DC (CB2/CB3 sono in serie, bloccano la DC: correnti P2/P3 e V(VPIP)
identiche prima e dopo il cambio).

**File modificati:** `electret2xlr.zen` (CB2, CB3: `22uF` → `1uF`),
`docs/bom-draft.md` (nota di coerenza sul valore).

## Riepilogo criteri

| # | Criterio | Esito |
|---|---|---|
| 1 | V(VPIP) 7.0–8.0V | PASS (7.486V) |
| 2 | I(P2), I(P3) 2.5–3.5mA, sbil. <10% | PASS (2.996mA / 3.131mA, 4.4%) |
| 3 | V(MICOUT) 3–5V | PASS (4.086V) |
| 4 | AC piatta ±1dB 100Hz–20kHz | PASS (0.38dB) |
| 5 | −3dB inferiore tra 20 e 60Hz | PASS (≈30.6Hz, dopo correzione CB2/CB3) |

Tutti i criteri sono soddisfatti dopo la correzione di CB2/CB3. Nessun altro
valore del netlist è stato modificato.

## File di supporto creati

- `spice/Transistors.lib` — modelli Q_NPN_MMBT3904 / Q_PNP_MMBT3906.
- `spice/Connector.lib` — modello pass-through CONN3 per J1/J2.
- `testbench/setup_dc_ac.cir` — sorgenti e analisi ngspice (`--setup`).
- `testbench/output/ac_response.txt` — dump numerico del barrido AC completo.
