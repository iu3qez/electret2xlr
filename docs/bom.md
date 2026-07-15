# BOM finale — electret2xlr

Distinta base finale, allineata al netlist autoritativo `electret2xlr.zen` (Task 4-7)
e alla verifica SPICE (`docs/sim-results.md`). Sostituisce `docs/bom-draft.md`
(bozza di Task 3, superata: conteneva ancora CB2/CB3 = 22µF, valore rivisto
in Task 5 — vedi nota sotto la tabella).

Tutti i componenti sono jellybean, ≥0603 (vincolo saldabilità a mano, vedi
`CLAUDE.md`). Nessun codice LCSC è incluso: la ricerca a registro/stock
(`registry-search`) non è risultata utilizzabile in queste sessioni (auth
OAuth interattiva non completabile in sandbox) — vedi metodologia in fondo.
I valori sono nominali; possibile bench tuning dopo il primo prototipo.

## Tabella

| Ref | Valore | Package | Cosa usare nel `.zen` | MPN (se noto) | Rating | Note |
|-----|--------|---------|------------------------|----------------|--------|------|
| Q1 | MMBT3904 (NPN) | SOT-23 | `Component()` primitivo, `Symbol("./symbols/Q_NPN_MMBT3904.kicad_sym")`, footprint `./footprints/SOT-23.kicad_mod` | MMBT3904 (onsemi, multi-source: Diodes Inc, Nexperia, Fairchild — pin/footprint-compatibili) | 40V / 200mA / 300mW | Emitter follower del cap-multiplier (VPIP) |
| Q2, Q3 | MMBT3906 (PNP) | SOT-23 | `Component()` primitivo, `Symbol("./symbols/Q_PNP_MMBT3906.kicad_sym")`, footprint `./footprints/SOT-23.kicad_mod` | MMBT3906 (onsemi, multi-source) | 40V / 200mA / 300mW | Follower appaiati hot (Q2, pin2) / cold (Q3, pin3) |
| D1 | Zener 8.2V | SOD-123 | `Module("@stdlib/generics/Zener.zen")` — `zener_voltage="8.2V"`, `power="500mW"`, `package="SOD-123"` | BZT52C8V2-class (Nexperia/Diodes Inc/onsemi, 500mW) | 500mW | Riferimento grezzo del cap-multiplier |
| C1, C2, C3 | 22µF | 1206 | `Module("@stdlib/generics/Capacitor.zen")` — `value="22uF"`, `package="1206"`, `dielectric="X5R"`, `voltage="25V"` | generic (nessun MPN richiesto) | 25V X5R | C1 bypass VPIP; C2 accoppiamento capsula→Q2B; C3 bypass Q3B a massa in AC |
| CB2, CB3 | **1µF** | 1206 | `Module("@stdlib/generics/Capacitor.zen")` — `value="1uF"`, `package="1206"`, `dielectric="X7R"`, `voltage="50V"` | generic | 50V X7R | Bypass di emettitore Q2/Q3. **Valore ratificato in Task 5** (non 22µF: con 22µF il ginocchio passa-alto cadeva sotto 1Hz — vedi `docs/sim-results.md`). Siedono su ~23V DC, rating 50V obbligatorio |
| C4 | 47µF | 1210 | `Module("@stdlib/generics/Capacitor.zen")` — `value="47uF"`, `package="1210"`, `voltage="25V"` | generic | 25V | Filtro R9/C4 del cap-multiplier (corner <1Hz) |
| C5 | 10µF | 0805 | `Module("@stdlib/generics/Capacitor.zen")` — `value="10uF"`, `package="0805"`, `voltage="25V"` | generic | 25V | Bypass rail VPIP dopo Q1 |
| R1 | 7.5kΩ | 1206 | `Module("@stdlib/generics/Resistor.zen")` — `value="7.5kohm"`, `package="1206"` | generic | ~250mW (1206) | Feed emettitore Q2 da pin 2; ~67mW continui in guscio chiuso |
| R2 | 10kΩ | 1206 | `Module("@stdlib/generics/Resistor.zen")` — `value="10kohm"`, `package="1206"` | generic | ~250mW (1206) | Feed emettitore Q3 da pin 3; ~50mW continui |
| R3, R4, R6, R7 | 100kΩ | 0603 | `Module("@stdlib/generics/Resistor.zen")` — `value="100kohm"`, `package="0603"` | generic | — | Polarizzazione base Q2 (R3/R4) e Q3 (R6/R7) |
| R8 | 1kΩ | 0603 | `Module("@stdlib/generics/Resistor.zen")` — `value="1kohm"`, `package="0603"` | generic | — | Base Q1 |
| R9 | 47kΩ | 0603 | `Module("@stdlib/generics/Resistor.zen")` — `value="47kohm"`, `package="0603"` | generic | — | Filtro RC con C4, corner <1Hz |
| R10 | 6.8kΩ | 0603 | `Module("@stdlib/generics/Resistor.zen")` — `value="6.8kohm"`, `package="0603"` | generic | — | Bias capsula da VPIP verso MICOUT/tip |
| RB2, RB3 | 47Ω | 0603 | `Module("@stdlib/generics/Resistor.zen")` — `value="47ohm"`, `package="0603"` | generic | — | Stopper in serie a CB2/CB3 |
| RIN | 100Ω | 0603 | `Module("@stdlib/generics/Resistor.zen")` — `value="100ohm"`, `package="0603"` | generic | — | Serie del filtro anti-RF a π sull'ingresso capsula |
| FB1 | Ferrite 600Ω@100MHz | 0603 | `Module("@stdlib/generics/FerriteBead.zen")` — `value="600ohm"`, `frequency="100MHz"`, `current="500mA"`, `package="0603"` | BLM18AG601SN1 (Murata) — 600Ω±25%@100MHz, 500mA, DCR max 0.38Ω | 500mA | Filtro anti-RF ingresso capsula |
| CIN | 220pF | 0603 | `Module("@stdlib/generics/Capacitor.zen")` — `value="220pF"`, `package="0603"`, `dielectric="C0G"` | generic | C0G | Filtro anti-RF ingresso capsula |
| CP2, CP3 | 100pF | 0603 | `Module("@stdlib/generics/Capacitor.zen")` — `value="100pF"`, `package="0603"`, `dielectric="C0G"`, `voltage="100V"` | generic | 100V C0G | Difesa RF pin2→GND / pin3→GND, adiacenti ai pin XLR |
| CSH | 1nF | 0603 | `Module("@stdlib/generics/Capacitor.zen")` — `value="1nF"`, `package="0603"`, `dielectric="C0G"`, `voltage="100V"` | generic | 100V C0G | Pin1 → piazzola guscio. **Popolato (`dnp=False`) per NC3MXX standard**; impostare **`dnp=True` se si monta NC3MXX-EMC** (contatto capacitivo shell→schermo già integrato nel connettore, altrimenti capacità in doppio) |
| J2 | Jack 3.5mm TRS | SMT, basso profilo | `Component()` primitivo, footprint `./footprints/Jack_3.5mm_CUI_SJ-3523-SMT_Horizontal.kicad_mod` | **SJ-3523-SMT-TR** (Same Sky, ex CUI Devices) — variante senza switch | 12Vdc / 1A, contatti 50mΩ max | Opzionale: monta il jack sul guscio solo se vuoi l'innesto 3.5mm; in alternativa capsula su MIC1/cavetto |
| MIC1 | Piazzole capsula ×2 | THT Ø1.0mm, drill 0.5mm | `Module("@stdlib/generics/TestPoint.zen")` ×2, `variant="THTPad_D1.0mm_Drill0.5mm"`, passo 2.54mm nel layout | n/a (piazzole) | — | Saldatura diretta capsula 2 terminali; escluse dal netlist SPICE solo con `--config sim_mode=true` |
| J1 | Piazzole a filo per XLR volante | footprint custom `./footprints/J1_XLR3_WirePads.kicad_mod` | `Component()` primitivo, `skip_bom=True` | n/a | 1.5kVdc (rating spinotto Neutrik) | 3 piazzole THT (pin1=GND/schermo, pin2=hot, pin3=cold, passo 3.81mm) per saldare il cavetto schermato verso uno spinotto XLR maschio volante (es. Neutrik NC3MX). Lo spinotto va acquistato a parte |

## Correzioni rispetto a `bom-draft.md`

- **CB2, CB3: 22µF → 1µF.** Era già segnalato come "rivisto" nel draft (Task 5),
  ma il draft riportava ancora il vecchio valore in tabella in alcuni punti;
  qui è il valore definitivo, allineato al netlist e a `docs/sim-results.md`.
- **CSH**: nel draft era descritto con `dnp=True` condizionato; nel netlist
  attuale (`electret2xlr.zen`) è popolato di default (`dnp=False`, per
  NC3MXX standard). La tabella sopra riflette lo stato di default reale e
  la condizione per passare a DNP (variante -EMC).
- **J1**: chiarito che è `skip_bom=True` nel netlist — non compare come riga
  d'acquisto in un BOM generato da `pcb build`, riportato qui solo per
  completezza documentale.
- Rimossi i riferimenti a LCSC/stock nella colonna dedicata: nessun codice
  LCSC è stato verificato in queste sessioni (vedi nota metodologica sotto),
  quindi non ne viene riportato nessuno per evitare di suggerire una
  verifica di stock che non è stata fatta.

## Nota metodologica

`registry-search`/`pcb search -m registry:*` non è risultato utilizzabile
nelle sessioni di questo progetto: richiede un login OAuth interattivo
(`pcb auth login`, browser + account diode.computer) non completabile in un
ambiente agente non interattivo. Per i componenti coperti da un generic
`@stdlib/generics/*` (tutte le passive, lo zener, la ferrite, i test point)
questo non ha impatto: generic + `value` + `package` *è* la parte. Per Q1/Q2/Q3
(BJT SOT-23) e J2 (jack) sono stati autorati `Component()` primitivi con
`Symbol()`/footprint vendorizzati localmente sotto `symbols/` e `footprints/`,
verificati da datasheet (JEDEC SOT-23, datasheet Same Sky SJ-352X-SMT), MPN
reali citati ma **senza verifica di stock LCSC**. Prima di un ordine reale:
ripetere `pcb auth login` con un account disponibile e verificare stock/basic-part
su LCSC per Q1/Q2/Q3/J2, o quantomeno un controllo manuale su distributori.
