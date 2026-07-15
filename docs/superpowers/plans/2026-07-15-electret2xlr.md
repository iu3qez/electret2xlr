# electret2xlr Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** PCB (pcb-as-code Zener) di un adattatore attivo electret → XLR alimentato da phantom P48, montato dentro un connettore Neutrik NC3MXX, con ingresso jack TRS 3.5mm + piazzole capsula e hardening RF.

**Architecture:** Topologia p48-pip-adapter: moltiplicatore di capacità (zener 8.2V + RC <1Hz + follower NPN) → rail PIP ~7.5V; due follower PNP gemelli (hot con segnale su pin 2, cold a base AC-grounded su pin 3) → uscita impedance-balanced ~78Ω con assorbimento simmetrico ~3mA/pin. Aggiunte nostre: jack TRS + piazzole capsula con filtro π anti-RF in ingresso, 100pF sui pin XLR. PCB 4 strati 0.8mm, 11.1mm di larghezza, sandwich-mount tra i pin dell'XLR.

**Tech Stack:** `pcb` CLI (Zener/Starlark) di Diode Inc + KiCad 10.x + skill diodeinc (`zener-language`, `registry-search`, `librarian`, `datasheet-reader`, `spice-sim`) + ngspice per la simulazione.

**Spec di riferimento:** `docs/superpowers/specs/2026-07-15-electret2xlr-design.md` (approvata). Netlist di riferimento: `netlist.py` del repo https://github.com/tphakala/p48-pip-adapter (da consultare in lettura; NON copiarne layout/file: CC BY-NC).

## Global Constraints

- **Footprint minimo 0603** — mai componenti più piccoli; ammessi 0603/0805/1206/1210, SOT-23, SOD-323/SOD-123.
- **Componenti**: per questo progetto (passivi + transistor jellybean, tutte parti non critiche) il vincolo è il **footprint corretto**, non lo stock. Usare `registry-search` di diode per avere parti con footprint pronti per il `.zen`. La verifica di stock LCSC in tempo reale non è richiesta oggi (in futuro: DB jlcparts).
- **PCB: 4 strati, spessore 0.8mm (non negoziabile), larghezza 11.1mm**, strati interni = piani GND pieni via-stitched.
- **Alimentazione: solo phantom P48** (48V, 6.8kΩ/ramo); assorbimento simmetrico sui pin 2 e 3.
- **CB2/CB3 rating ≥50V X7R** (siedono su ~23V DC). CP2/CP3 (100pF sui pin XLR) rating ≥100V C0G (siedono su 48V).
- Commit frequenti, messaggi in italiano che spiegano il perché, **niente riga Co-Authored-By**.
- Il "test" di ogni modifica al sorgente è `pcb build` pulito; per il layout è la DRC KiCad a zero errori.

---

### Task 1: Toolchain e skill diodeinc

**Files:**
- Create: `.claude/skills/zener-language/`, `.claude/skills/registry-search/`, `.claude/skills/librarian/`, `.claude/skills/datasheet-reader/`, `.claude/skills/spice-sim/` (copiati dal repo diodeinc/pcb)

**Interfaces:**
- Produces: comando `pcb` funzionante nel PATH; `kicad-cli` 10.x disponibile; skill consultabili in `.claude/skills/` per tutti i task successivi.

- [ ] **Step 1: Installa il CLI `pcb`** (non è su PyPI — verificato: è un binario Rust)

```bash
curl -fsSL https://raw.githubusercontent.com/diodeinc/pcb/main/install.sh | bash
export PATH="$HOME/.local/bin:$PATH"
pcb --version
```

Expected: versione stampata senza errori (es. `pcb x.y.z`).

- [ ] **Step 2: Verifica/installa KiCad 10.x** (richiesto da `pcb layout`)

```bash
kicad-cli version
```

Expected: `10.x`. Se assente o più vecchio: installare (Debian/Ubuntu: `sudo apt install kicad` se il repo ha la 10.x, altrimenti flatpak `flatpak install org.kicad.KiCad`). Se serve intervento dell'utente (sudo/password), fermarsi e chiederlo.

- [ ] **Step 3: Installa le skill diodeinc nel progetto**

```bash
git clone --depth 1 https://github.com/diodeinc/pcb /tmp/diode-pcb
mkdir -p .claude/skills
cp -r /tmp/diode-pcb/skills/* .claude/skills/
ls .claude/skills
```

Expected: `datasheet-reader  librarian  registry-search  spice-sim  zener-language`.

- [ ] **Step 4: Leggi la skill del linguaggio** — leggere per intero `.claude/skills/zener-language/SKILL.md` (e i file collegati). Le firme API usate nei task successivi (Module, Component, Net, Board, generics) vanno verificate contro questa skill e `pcb doc --package @stdlib`; in caso di divergenza fa fede la skill, non questo piano.

- [ ] **Step 5: Commit**

```bash
git add .claude/skills
git commit -m "Installa le skill diodeinc per il toolchain pcb/Zener"
```

---

### Task 2: Workspace Zener minimale

**Files:**
- Create: `pcb.toml`
- Create: `electret2xlr.zen` (stub di board vuota)

**Interfaces:**
- Produces: workspace che `pcb build` valida; `electret2xlr.zen` è il top-level che il Task 4 riempirà.

- [ ] **Step 1: Prova lo scaffolding ufficiale**

```bash
pcb new board electret2xlr github.com/sfabris/electret2xlr
```

Se il comando vuole creare una directory nuova invece di popolare il repo corrente, spostare i file generati nella root del repo. Se il comando non esiste in questa versione, creare i file a mano (Step 2).

- [ ] **Step 2 (fallback manuale): crea `pcb.toml`**

```toml
[workspace]
repository = "github.com/sfabris/electret2xlr"
pcb-version = "0.4"

[board]
name = "electret2xlr"
path = "electret2xlr.zen"
description = "Adattatore attivo electret -> XLR bilanciato alimentato da phantom P48, montato dentro un Neutrik NC3MXX."
```

e `electret2xlr.zen` stub (adattare la firma di `Board` a quanto dice la skill zener-language):

```zen
Board(name="electret2xlr", layout_path="layout/electret2xlr", layers=4)
```

- [ ] **Step 3: Valida**

```bash
pcb sync && pcb build electret2xlr.zen
```

Expected: build senza errori.

- [ ] **Step 4: Commit**

```bash
git add pcb.toml electret2xlr.zen
git commit -m "Scaffolding workspace Zener con board 4 strati vuota"
```

---

### Task 3: Selezione componenti a stock LCSC

**Files:**
- Create: `docs/bom-draft.md`

**Interfaces:**
- Produces: tabella con, per ogni ref, **MPN + package + footprint (dal registry diode) + simbolo/parte da usare nel `.zen`**; il Task 4 usa questi identificativi nei `Component(...)` / generics. Il codice LCSC è opzionale (annotarlo se il registry lo espone, ma non è un requisito).

Usare le skill `registry-search` e `librarian` di diode: l'obiettivo è ottenere per ogni posizione una parte con **footprint corretto** già utilizzabile nel `.zen`. Lo stock LCSC NON è un criterio di accettazione per questo progetto (parti tutte non critiche). Requisiti per posizione (valori dalla spec; candidati indicati dove noti):

| Ref | Parte | Requisiti | Candidato |
|-----|-------|-----------|-----------|
| Q1 | MMBT3904 | NPN SOT-23 | LCSC C20526 (basic) |
| Q2, Q3 | MMBT3906 | PNP SOT-23 | LCSC C8492 |
| D1 | Zener 8.2V | SOD-123/SOD-323, ≥300mW | BZT52C8V2 |
| C1, C2, C3 | 22µF | X5R/X7R ≥16V, 1206/1210 | — |
| CB2, CB3 | 22µF | **X7R ≥50V**, 1206/1210 (se introvabile in 1206, 1210 è ammesso) | — |
| C4 | 47µF | ≥16V, ≤1210 | — |
| C5 | 10µF | ≥16V, 0805 | — |
| R1 | 7.5kΩ | **1206** (67mW continui) | — |
| R2 | 10kΩ | **1206** (50mW continui) | — |
| R3, R4, R6, R7 | 100kΩ | 0603 | — |
| R8 | 1kΩ | 0603 | — |
| R9 | 47kΩ | 0603 | — |
| R10 | 6.8kΩ | 0603 | — |
| RB2, RB3 | 47Ω | 0603 | — |
| RIN | 100Ω | 0603 | — |
| FB1 | Ferrite | 600Ω@100MHz, 0603, ≥200mA | BLM18AG601SN1 o equiv. LCSC |
| CIN | 220pF | C0G 0603 | — |
| CP2, CP3 | 100pF | **C0G ≥100V** 0603 | — |
| CSH | 1nF | C0G ≥100V 0603, **DNP** se si usa NC3MXX-EMC | — |
| J2 | Jack 3.5mm TRS | PCB-mount, corpo ≤11mm di larghezza, boccola compatibile con foro ghiera XLR | PJ-3133 / SJ-3523 / PJ-392: scegliere col datasheet |
| J1 | XLR (pad) | nessun componente: 3 pad sandwich definiti nel layout | — |
| MIC1 | Piazzole capsula | 2 pad THT Ø1mm passo 2.54mm | — |

- [ ] **Step 1:** per ogni riga, cercare con `registry-search` una parte diode con il footprint corretto; annotare MPN, package, footprint e l'identificativo da usare nel `.zen` (Module/Symbol) in `docs/bom-draft.md`. Codice LCSC solo se il registry lo espone (colonna opzionale).
- [ ] **Step 2:** per J2 individuare un jack 3.5mm TRS PCB-mount adatto e, se il datasheet è reperibile (`datasheet-reader` o web), annotare le dimensioni del corpo e della boccola (servono ai Task 6-7). Se non trovi un jack nel registry, annota comunque un MPN reale plausibile con le sue quote.
- [ ] **Step 3:** se per una posizione il registry non ha il footprint richiesto, annotare l'alternativa (footprint equivalente ≥0603) e proseguire; segnalare nel report solo se serve cambiare un *valore* del circuito (in quel caso fermarsi).
- [ ] **Step 4: Commit**

```bash
git add docs/bom-draft.md
git commit -m "BOM preliminare con componenti verificati a stock LCSC"
```

---

### Task 4: Netlist Zener completa

**Files:**
- Modify: `electret2xlr.zen`

**Interfaces:**
- Consumes: MPN/LCSC da `docs/bom-draft.md`; sintassi verificata dalla skill `zener-language`.
- Produces: board completa che passa `pcb build`; nomi di net stabili usati da sim e layout: `GND, P2, P3, VPIP, VREF, Q1B, MICIN, MICF, MICOUT, Q2B, Q2E, Q3B, Q3E, NB2, NB3, SHELL`.

Netlist da implementare (riferimento p48 + aggiunte RF; i pin BJT seguono la piedinatura SOT-23 MMBT390x: 1=B, 2=E, 3=C):

| Net | Connessioni | Funzione |
|-----|-------------|----------|
| GND | J1-1, MIC1-2, J2-SLEEVE, D1-A, C4-2, C5-2, C1-2, C3-2, R4-2, R7-2, Q2-C, Q3-C, CIN-2, CP2-2, CP3-2, CSH-2 | massa |
| P2 | J1-2, R1-1, RB2-2, CP2-1 | XLR pin 2 (hot) |
| P3 | J1-3, R9-1, Q1-C, R2-1, RB3-2, CP3-1 | XLR pin 3 (cold + alimentazione regolatore) |
| VPIP | Q1-E, C5-1, C1-1, R10-1, R3-1, R6-1 | rail PIP ~7.5V |
| VREF | R9-2, D1-1, C4-1, R8-1 | riferimento zener filtrato |
| Q1B | R8-2, Q1-B | base regolatore |
| MICIN | J2-TIP, J2-RING (ponticello), MIC1-1, CIN-1, FB1-1 | nodo capsula/jack, con C anti-RF |
| MICF | FB1-2, RIN-1 | dopo la ferrite |
| MICOUT | RIN-2, R10-2, C2-1 | bias capsula + prelievo segnale |
| Q2B | R3-2, R4-1, C2-2, Q2-B | base hot |
| Q2E | R1-2, Q2-E, CB2-1 | emettitore hot |
| Q3B | R6-2, R7-1, C3-1, Q3-B | base cold (AC a massa) |
| Q3E | R2-2, Q3-E, CB3-1 | emettitore cold |
| NB2 | CB2-2, RB2-1 | bypass hot |
| NB3 | CB3-2, RB3-1 | bypass cold |
| SHELL | pad guscio, CSH-1 | schermo (CSH DNP con NC3MXX-EMC) |

Nota zener D1: il catodo va su VREF (nodo positivo, clampa a +8.2V), l'anodo a GND. Verificare la convenzione dei pin del simbolo scelto.

- [ ] **Step 1: scrivi `electret2xlr.zen`.** Struttura di riferimento (adattare le firme alla skill; i `Part`/simboli vengono dalla BOM del Task 3):

```zen
Resistor = Module("@stdlib/generics/Resistor.zen")
Capacitor = Module("@stdlib/generics/Capacitor.zen")
FerriteBead = Module("@stdlib/generics/FerriteBead.zen")  # se non esiste nel stdlib: Component con MPN LCSC

GND = Net("GND")
P2 = Net("P2")
P3 = Net("P3")
VPIP = Net("VPIP")
VREF = Net("VREF")
Q1B = Net("Q1B")
MICIN = Net("MICIN")
MICF = Net("MICF")
MICOUT = Net("MICOUT")
Q2B = Net("Q2B")
Q2E = Net("Q2E")
Q3B = Net("Q3B")
Q3E = Net("Q3E")
NB2 = Net("NB2")
NB3 = Net("NB3")
SHELL = Net("SHELL")

# --- alimentazione (cap multiplier) ---
Resistor(name="R9", value="47kohm", package="0603", P1=P3, P2=VREF)
Component(name="D1", symbol=Symbol(...zener BZT52C8V2...), pins={"K": VREF, "A": GND})
Capacitor(name="C4", value="47uF", package="1210", P1=VREF, P2=GND)
Resistor(name="R8", value="1kohm", package="0603", P1=VREF, P2=Q1B)
Component(name="Q1", symbol=Symbol(...MMBT3904...), pins={"B": Q1B, "E": VPIP, "C": P3})
Capacitor(name="C5", value="10uF", package="0805", P1=VPIP, P2=GND)
Capacitor(name="C1", value="22uF", package="1206", P1=VPIP, P2=GND)

# --- ingresso capsula/jack + anti-RF ---
Component(name="J2", symbol=Symbol(...jack TRS...), pins={"T": MICIN, "R": MICIN, "S": GND})
Component(name="MIC1", symbol=Symbol(...header 1x02...), pins={"1": MICIN, "2": GND})
Capacitor(name="CIN", value="220pF", package="0603", P1=MICIN, P2=GND)
FerriteBead(name="FB1", package="0603", P1=MICIN, P2=MICF)
Resistor(name="RIN", value="100ohm", package="0603", P1=MICF, P2=MICOUT)
Resistor(name="R10", value="6.8kohm", package="0603", P1=VPIP, P2=MICOUT)

# --- stadio audio ---
Capacitor(name="C2", value="22uF", package="1206", P1=MICOUT, P2=Q2B)
Resistor(name="R3", value="100kohm", package="0603", P1=VPIP, P2=Q2B)
Resistor(name="R4", value="100kohm", package="0603", P1=Q2B, P2=GND)
Component(name="Q2", symbol=Symbol(...MMBT3906...), pins={"B": Q2B, "E": Q2E, "C": GND})
Resistor(name="R1", value="7.5kohm", package="1206", P1=P2, P2=Q2E)
Capacitor(name="CB2", value="22uF", package="1206", P1=Q2E, P2=NB2)   # 50V X7R
Resistor(name="RB2", value="47ohm", package="0603", P1=NB2, P2=P2)

Resistor(name="R6", value="100kohm", package="0603", P1=VPIP, P2=Q3B)
Resistor(name="R7", value="100kohm", package="0603", P1=Q3B, P2=GND)
Capacitor(name="C3", value="22uF", package="1206", P1=Q3B, P2=GND)
Component(name="Q3", symbol=Symbol(...MMBT3906...), pins={"B": Q3B, "E": Q3E, "C": GND})
Resistor(name="R2", value="10kohm", package="1206", P1=P3, P2=Q3E)
Capacitor(name="CB3", value="22uF", package="1206", P1=Q3E, P2=NB3)   # 50V X7R
Resistor(name="RB3", value="47ohm", package="0603", P1=NB3, P2=P3)

# --- XLR e difesa RF ai pin ---
Component(name="J1", symbol=Symbol(...XLR3 / pad sandwich...), pins={"1": GND, "2": P2, "3": P3})
Capacitor(name="CP2", value="100pF", package="0603", P1=P2, P2=GND)   # C0G 100V, adiacente al pin
Capacitor(name="CP3", value="100pF", package="0603", P1=P3, P2=GND)   # C0G 100V, adiacente al pin
Capacitor(name="CSH", value="1nF", package="0603", P1=SHELL, P2=GND)  # DNP con NC3MXX-EMC

Board(name="electret2xlr", layout_path="layout/electret2xlr", layers=4)
```

Per J1 e MIC1 usare simboli/footprint generici o custom secondo quanto suggerisce la skill `librarian` (il footprint sandwich di J1 viene comunque rifinito nel Task 6).

- [ ] **Step 2: Build (test)**

```bash
pcb build electret2xlr.zen
```

Expected al primo giro: possibili errori su simboli/firme → correggerli con `librarian`/`pcb doc --package @stdlib`. Iterare fino a build pulito.

- [ ] **Step 3: Verifica incrociata della netlist** — confrontare a mano ogni riga della tabella net sopra con il sorgente (o con l'output netlist di `pcb build`): ogni pin nella tabella deve comparire, nessun pin in più.

- [ ] **Step 4: Commit**

```bash
git add electret2xlr.zen pcb.toml
git commit -m "Netlist completa: cap multiplier, follower PNP bilanciati, ingresso jack/capsula con filtro anti-RF"
```

---

### Task 5: Verifica SPICE

**Files:**
- Create: `docs/sim-results.md` (+ eventuali file di sim generati dalla skill)

**Interfaces:**
- Consumes: netlist del Task 4 (net `VPIP`, `P2`, `P3`, `MICOUT`).
- Produces: verifica documentata del punto di lavoro e della risposta; eventuali correzioni di valori committate.

- [ ] **Step 1:** usare la skill `spice-sim` per costruire la simulazione della board. Modello della sorgente: generatore phantom 48V con 6.8kΩ verso P2 e P3; capsula modellata come sorgente di corrente AC (o JFET semplice) sul nodo MICIN con 0.5mA di bias DC. Il repo p48 ha una cartella `sim/` consultabile come riferimento di metodo.
- [ ] **Step 2: DC operating point.** Criteri di accettazione:
  - `V(VPIP)` tra 7.0 e 8.0V
  - corrente da P2 e da P3 ciascuna tra 2.5 e 3.5mA, sbilanciamento <10%
  - `V(MICOUT)` tra 3 e 5V (bias capsula sano)
- [ ] **Step 3: AC.** Criteri: risposta P2→(P2−P3) piatta ±1dB tra 100Hz e 20kHz; −3dB inferiore tra 20 e 60Hz. Nota dal progetto originale (issue #6): la fase può risultare invertente — annotarlo; l'eventuale rimedio è scambiare gli accoppiamenti C2/C3, decisione da prendere col bench test, non ora.
- [ ] **Step 4:** scrivere risultati e grafici/numeri in `docs/sim-results.md`. Se un criterio fallisce, correggere i valori nel `.zen`, rilanciare `pcb build` + sim, documentare il cambio.
- [ ] **Step 5: Commit**

```bash
git add docs/sim-results.md electret2xlr.zen
git commit -m "Verifica SPICE: punto di lavoro DC e risposta AC nei criteri di spec"
```

---

### Task 6: Layout 4 strati per NC3MXX

**Files:**
- Create/Modify: `layout/electret2xlr/` (file KiCad generati da `pcb layout` e poi editati)

**Interfaces:**
- Consumes: netlist Task 4, dimensioni jack da `docs/bom-draft.md`.
- Produces: `.kicad_pcb` con DRC a zero errori, pronto per gerber.

Geometria vincolante (dal progetto di riferimento, collaudata):

- Outline: larghezza **11.1mm**; lunghezza = 35.3mm + ingombro jack (target ≤45mm, da confermare col Task 7).
- Spessore **0.8mm**, 4 strati; In1/In2 = piani GND pieni, via-stitched ai pour esterni e al pad di pin 1.
- Pad XLR "sandwich" (origine = bordo lato XLR): pin 1 sul **fronte** a x=1.90mm; pin 2 sul **fronte** a x=9.52mm; pin 3 sul **retro** a x=5.71mm. Pad lunghi **8mm**, che si estendono ~3mm oltre la punta del pin. Interasse bicchierini NC3MXX: 7.62mm.
- Punta dei pin XLR: **≥5mm di clearance** dal primo componente.
- CP2/CP3 fisicamente adiacenti ai pad XLR; CIN/FB1/RIN adiacenti al jack/piazzole MIC1; componenti solo sul lato top (a parte il pad di pin 3).
- Jack J2 sul bordo posteriore, boccola oltre l'outline.

- [ ] **Step 1:** `pcb layout electret2xlr.zen` → genera il progetto KiCad in `layout/electret2xlr/`.
- [ ] **Step 2:** Board Setup in KiCad (o editando il file): 4 strati con In1/In2 piani, spessore 0.8mm; regole: clearance 0.125mm, track min 0.15mm, via 0.6/0.3mm.
- [ ] **Step 3:** disegnare outline e pad sandwich alle coordinate sopra (footprint custom per J1: pad SMD 8mm su F.Cu per pin 1-2, su B.Cu per pin 3).
- [ ] **Step 4:** piazzare per blocchi funzionali nell'ordine del segnale (jack → filtro RF → bias/audio → alimentazione → pad XLR), rispettando la clearance di 5mm dalle punte dei pin. I courtyard dei 1206/1210 potrebbero sovrapporsi: warning accettabile, corti no.
- [ ] **Step 5:** routing; GND per via ai piani interni; stitching lungo il perimetro.
- [ ] **Step 6: DRC (test)** — `kicad-cli pcb drc layout/electret2xlr/electret2xlr.kicad_pcb` (o da GUI). Expected: **0 errori** (warning courtyard tollerati). Iterare fino a pulito.
- [ ] **Step 7: Commit**

```bash
git add layout/
git commit -m "Layout 4 strati 0.8mm per montaggio sandwich in NC3MXX, DRC pulita"
```

---

### Task 7: Verifica fit meccanico

**Files:**
- Create: `docs/meccanica.md`

**Interfaces:**
- Consumes: layout Task 6, datasheet jack (Task 3).
- Produces: conferma quotata che il PCB entra nell'NC3MXX e decisione definitiva sul montaggio del jack; eventuale correzione dell'outline.

- [ ] **Step 1:** scaricare il disegno quotato Neutrik NC3MXX (e NC3MXX-EMC) e, con la skill `datasheet-reader`, estrarre: diametro e lunghezza utile della cavità interna, diametro del foro della boccola posteriore, quote dell'insert.
- [ ] **Step 2:** confrontare con il layout: larghezza 11.1mm × spessore assemblato (0.8mm + componenti ~2.5mm) dentro la cavità; lunghezza totale ≤ lunghezza utile; boccola del jack vs foro posteriore (il bushing NC3MXX si può forare/sostituire: annotare il diametro richiesto).
- [ ] **Step 3:** se qualcosa non entra: accorciare/riposizionare (tornare al Task 6), oppure attivare il piano B della spec (capsula su cavetto attraverso il pressacavo standard) — **decisione da sottoporre all'utente**.
- [ ] **Step 4:** documentare tutte le quote e la decisione in `docs/meccanica.md`. Consigliare nel documento la stampa di un fit-test (anche solo il PCB nudo ordinato in anticipo, o una sagoma cartoncino 11.1×L mm) prima del montaggio finale.
- [ ] **Step 5: Commit**

```bash
git add docs/meccanica.md
git commit -m "Verifica quotata del fit nel NC3MXX e decisione sul montaggio del jack"
```

---

### Task 8: README e BOM finale

**Files:**
- Create: `README.md`
- Create: `docs/bom.md` (BOM finale; sostituisce concettualmente `bom-draft.md`, che si può eliminare)

**Interfaces:**
- Consumes: tutto quanto sopra.
- Produces: documentazione completa del progetto.

- [ ] **Step 1: `README.md`** con: scopo (una riga), schema a blocchi/teoria di funzionamento (riassunto della sezione Circuito della spec, con i tre stadi), istruzioni di build (`pcb build`, `pcb layout`), istruzioni di montaggio nell'NC3MXX (sequenza di saldatura sandwich: pre-stagnare pad pin 3 e bicchierino, appoggiare su pin 1-2, ponte su pin 3 per ultimo), cablaggio jack (TRS: tip=segnale, sleeve=GND; TRRS non supportato senza adattatore), nota di attribuzione di cortesia a tphakala/p48-pip-adapter con il chiarimento licenza della spec.
- [ ] **Step 2: `docs/bom.md`** — tabella finale ref / valore / package / MPN / codice LCSC / rating / note (DNP per CSH se NC3MXX-EMC), ricontrollando che ogni codice sia ancora a stock.
- [ ] **Step 3:** rileggere README e spec insieme: nessuna contraddizione (valori, quote, pinout).
- [ ] **Step 4: Commit**

```bash
git add README.md docs/bom.md
git rm docs/bom-draft.md
git commit -m "Documentazione finale: README con teoria e montaggio, BOM con codici LCSC"
```
