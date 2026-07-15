# BOM preliminare — electret2xlr (Task 3)

Stato: bozza per Task 4. Fonti: `pcb doc --package @stdlib` (generics reali, nessun
auth richiesto) + datasheet dei produttori (verificati via web, nessuno scraping
di stock LCSC — vedi Note metodologiche in fondo). **`registry-search` (pcb search
-m registry:*) non è stato utilizzabile in questa sessione: vedi Note metodologiche.**

## Tabella

| Ref | Valore | Package | Cosa usare nel .zen | Footprint | MPN (se noto) | Note |
|-----|--------|---------|----------------------|-----------|----------------|------|
| Q1 | MMBT3904 (NPN) | SOT-23 | *Non risolvibile via registry-search in questa sessione (vedi Note metodologiche).* Da autorare come `Component()` primitivo con `Symbol()` verificato dal datasheet, oppure ripetere `pcb search -m registry:components "MMBT3904"` non appena l'auth è disponibile. | SOT-23 (JEDEC TO-236AB, 3 pin, pitch 0.95mm) | MMBT3904 (multi-source: onsemi, Diodes Inc, Nexperia, Fairchild — tutti pin/footprint-compatibili) | 40V/200mA/300mW, ampiamente second-sourced |
| Q2, Q3 | MMBT3906 (PNP) | SOT-23 | Idem Q1 (registry non raggiungibile) | SOT-23 (JEDEC TO-236AB) | MMBT3906 (multi-source) | Complementare PNP di MMBT3904, stesso case |
| D1 | Zener 8.2V | SOD-123 (SOD-323 come da testo spec originale, entrambi ammessi) | `Module("@stdlib/generics/Zener.zen")` — `zener_voltage="8.2V"`, `power="500mW"`, `package="SOD-123"` | SOD-123 | BZT52C8V2 (Nexperia/Diodes Inc/onsemi MMSZ-class, 500mW ≥ 300mW richiesti). Se serve SOD-323: Nexperia BZT52-B8V2 series (package più piccolo, da confermare footprint) | Generic stdlib esiste già: nessuna dipendenza dal registry |
| C1, C2, C3 | 22µF | 1206 | `Module("@stdlib/generics/Capacitor.zen")` — `value="22uF"`, `package="1206"`, `dielectric="X5R"`, `voltage="25V"` | 1206 | — (generic, nessun MPN richiesto) | ≥16V richiesti, 25V è il grade E-series standard immediatamente sopra |
| CB2, CB3 | 22µF | 1206 (1210 ammesso se necessario) | `Module("@stdlib/generics/Capacitor.zen")` — `value="22uF"`, `package="1206"`, `dielectric="X7R"`, `voltage="50V"` | 1206 | — (generic) | X7R 50V 22µF in 1206 esiste (es. famiglie Yageo/Samsung/Murata equivalenti); se il fitting del case risultasse troppo fitto passare a `package="1210"` senza cambiare valore |
| C4 | 47µF | 1210 | `Module("@stdlib/generics/Capacitor.zen")` — `value="47uF"`, `package="1210"`, `voltage="25V"` | 1210 | — (generic) | vincolo brief: ≤1210 → 1210 è il case massimo consentito e il più comune per 47µF ceramico |
| C5 | 10µF | 0805 | `Module("@stdlib/generics/Capacitor.zen")` — `value="10uF"`, `package="0805"`, `voltage="25V"` | 0805 | — (generic) | — |
| R1 | 7.5kΩ | 1206 | `Module("@stdlib/generics/Resistor.zen")` — `value="7.5kohm"`, `package="1206"` | 1206 | — (generic) | 67mW continui in guscio chiuso → 1206 (250mW rating tipico) dà margine |
| R2 | 10kΩ | 1206 | `Module("@stdlib/generics/Resistor.zen")` — `value="10kohm"`, `package="1206"` | 1206 | — (generic) | 50mW continui, stesso margine di R1 |
| R3, R4, R6, R7 | 100kΩ | 0603 | `Module("@stdlib/generics/Resistor.zen")` — `value="100kohm"`, `package="0603"` | 0603 | — (generic) | — |
| R8 | 1kΩ | 0603 | `Module("@stdlib/generics/Resistor.zen")` — `value="1kohm"`, `package="0603"` | 0603 | — (generic) | — |
| R9 | 47kΩ | 0603 | `Module("@stdlib/generics/Resistor.zen")` — `value="47kohm"`, `package="0603"` | 0603 | — (generic) | — |
| R10 | 6.8kΩ | 0603 | `Module("@stdlib/generics/Resistor.zen")` — `value="6.8kohm"`, `package="0603"` | 0603 | — (generic) | — |
| RB2, RB3 | 47Ω | 0603 | `Module("@stdlib/generics/Resistor.zen")` — `value="47ohm"`, `package="0603"` | 0603 | — (generic) | — |
| RIN | 100Ω | 0603 | `Module("@stdlib/generics/Resistor.zen")` — `value="100ohm"`, `package="0603"` | 0603 | — (generic) | — |
| FB1 | Ferrite 600Ω@100MHz | 0603 | `Module("@stdlib/generics/FerriteBead.zen")` — `value="600ohm"`, `frequency="100MHz"`, `current="500mA"`, `package="0603"` | 0603 | BLM18AG601SN1 (Murata) — 600Ω±25%@100MHz, Irated 500mA, DCR max 0.38Ω, case 1.6×0.8×0.8mm | Generic stdlib esiste; MPN reale confermato da datasheet Murata, ≥200mA richiesti soddisfatti (500mA) |
| CIN | 220pF | 0603 | `Module("@stdlib/generics/Capacitor.zen")` — `value="220pF"`, `package="0603"`, `dielectric="C0G"` | 0603 | — (generic) | — |
| CP2, CP3 | 100pF | 0603 | `Module("@stdlib/generics/Capacitor.zen")` — `value="100pF"`, `package="0603"`, `dielectric="C0G"`, `voltage="100V"` | 0603 | — (generic) | C0G 0603 100V è reperibile (case comune per applicazioni anti-RF) |
| CSH | 1nF | 0603 | `Module("@stdlib/generics/Capacitor.zen")` — `value="1nF"`, `package="0603"`, `dielectric="C0G"`, `voltage="100V"`, `dnp=True` (se si usa NC3MXX-EMC) | 0603 | — (generic) | DNP condizionato alla scelta NC3MXX-EMC vs NC3MXX standard (vedi spec) |
| J2 | Jack 3.5mm TRS | PCB-mount, right-angle SMT | *Non risolvibile via registry-search in questa sessione.* Da autorare come `Component()` primitivo (Symbol/footprint verificati dal datasheet Same Sky sotto) o cercare in registry non appena l'auth è disponibile | vedi dimensioni sotto | **SJ-3523-SMT-TR** (Same Sky, ex CUI Devices) — TRS 3 posizioni (sleeve/tip/ring), senza switch, SMT basso profilo | Vedi sezione dedicata "J2 — dimensioni" sotto |
| J1 | XLR (pad sandwich) | — | Nessun componente: 3 pad definiti nel layout (interasse bicchierini NC3MXX 7.62mm, pad 8mm lunghi, pin1/2 fronte, pin3 retro) | n/a | n/a | Come da brief: nessuna parte da selezionare, solo layout |
| MIC1 | Piazzole capsula | THT Ø1mm, passo 2.54mm | `Module("@stdlib/generics/TestPoint.zen")` ×2, `variant="THTPad_D1.0mm_Drill0.5mm"`, posizionati a 2.54mm di passo nel layout | THTPad Ø1.0mm, drill 0.5mm | n/a (piazzole, non un connettore) | Il generic TestPoint copre esattamente Ø1mm richiesto; il passo 2.54mm è un vincolo di layout (Task 6/7), non del componente |

## J2 — dimensioni (per Task 6-7)

Fonte: datasheet ufficiale Same Sky "SJ-352X-SMT" (rev 04/15/2025), letto pagina per
pagina (disegno meccanico a pag. 2, packaging a pag. 3). MPN scelto: **SJ-3523-SMT-TR**
(variante senza switch — coerente con J2 che nella spec è un semplice TRS 3
contatti, non serve tip-switch; SJ-3524-SMT-TR è la variante con switch se mai
servisse in futuro).

- **Corpo (housing), vista frontale:** larghezza **6.0mm** — ben sotto l'11.1mm di
  larghezza scheda.
- **Corpo, lunghezza principale (housing, esclusa sporgenza boccola):** **14.5mm**
  (dimensione quotata nella vista dall'alto); la boccola sporge **2.5mm** oltre la
  faccia frontale del corpo, la coda terminali **2mm** oltre il retro → ingombro
  totale fronte-retro ≈ **17mm** (14.5 + 2.5, coda terminali esclusa dall'inviluppo
  utile).
- **Altezza (spessore sopra il PCB):** non quotata come singola cifra nel disegno
  meccanico leggibile; dalla pagina packaging (tasca del blister/reel) risulta una
  profondità tasca di **6.1mm**, che nei jack SMT low-profile di questa famiglia
  corrisponde tipicamente all'altezza reale del componente + tolleranza di
  inserimento. **Da confermare con il modello 3D/STEP ufficiale** (link
  "3D Model" nel datasheet) prima del fit finale — non ho potuto scaricare/aprire
  lo STEP in questa sessione.
- **Boccola/naso (bushing, la parte che sporge e che deve passare per il foro
  della ghiera NC3MXX):** diametro esterno **Ø5.00mm**, foro interno (accetta lo
  spinotto) **Ø3.60mm** (spinotto di riferimento Ø3.5mm, coerente col nome
  "3.5mm jack").
- **Footprint PCB consigliato (dal datasheet):** inviluppo 14.5 × 11.8mm, 2 fori
  di fissaggio meccanico Ø2.8mm e Ø2.2mm, terminali segnale su 2×Ø1.70mm.
- **Elettrico:** 12Vdc / 1A rated, resistenza di contatto 50mΩ max — ampiamente
  sufficiente per un segnale audio/bias, nessun vincolo di potenza rilevante qui.
- **Compatibilità ghiera XLR NC3MXX:** non sono riuscito a reperire in questa
  sessione la quota ufficiale del foro passacavo/ghiera del Neutrik NC3MXX (i
  datasheet Neutrik trovati via ricerca web non espongono la quota in testo
  estraibile). Il Ø5.00mm della boccola SJ-3523-SMT-TR è comunque compatibile con
  i fori passacavo XLR tipici (dell'ordine di 6–8mm); **raccomando di verificare
  con un disegno quotato Neutrik ufficiale o un fit-test STL prima di ordinare**,
  come già segnalato come "rischio aperto" nella spec di progetto
  (`docs/superpowers/specs/2026-07-15-electret2xlr-design.md`).

PDF scaricato e ispezionato pagina per pagina in questa sessione (non incluso nel
repo): `https://www.sameskydevices.com/product/resource/sj-352x-smt.pdf`.

## Note metodologiche

- **`registry-search` non utilizzabile in questa sessione.** Ogni modalità di
  `pcb search` (`registry:modules`, `registry:components`, `kicad:components`,
  `web:components`) restituisce `Error: ... Not authenticated. Run 'pcb auth
  login' to authenticate.` Ho verificato che non esiste alcun token/cache
  d'autenticazione preesistente (`~/.pcb`, variabili d'ambiente `PCB_*`/`DIODE_*`)
  e ho eseguito `pcb auth login`: il comando genera un codice e un URL
  (`https://app.diode.computer/cli-auth?code=...`) da aprire in un browser
  autenticato con un account diode.computer — un flusso OAuth interattivo che
  **non può essere completato in una sessione agente non interattiva** (nessun
  browser, nessuna credenziale diode.computer disponibili). Questo blocca sia
  `registry-search` sia, potenzialmente, `librarian` per le parti che
  richiederebbero import dal registry.
  - Impatto: le posizioni coperte da un generic `@stdlib/generics/*` (praticamente
    tutte le passive + D1 zener + FB1 ferrite + MIC1 pad) **non sono impattate**:
    per queste, come indicato nel brief del task, il generic + value + package
    *è* la parte, e ho verificato l'esistenza del generic e la validità della
    stringa `package` leggendo `pcb doc --package @stdlib` (funziona senza auth,
    usa la cache/download pubblico dello stdlib).
  - Impatto reale: Q1/Q2/Q3 (BJT SOT-23) e J2 (jack) restano senza un
    `moduleUrl`/`Symbol` di registry concreto da mettere in `Component(...)`.
    Ho comunque fornito MPN reali, footprint standard JEDEC/datasheet-verificati
    e (per J2) le quote meccaniche complete, cercate via `WebSearch`/`WebFetch`
    sui siti dei produttori (non LCSC, coerente col vincolo "niente stock
    scraping"). Task 4 ha due strade equivalenti: (a) ripetere `pcb auth login`
    con un account disponibile e poi `pcb search -m registry:components` sui
    MPN qui indicati; (b) usare la skill `librarian` per autorare/importare un
    package primitivo con `Symbol()`/footprint KiCad standard verificato dal
    datasheet (SOT-23 per i BJT è nella libreria KiCad stock).
  - Non è stata necessaria alcuna modifica di **valore** circuitale: il blocco è
    puramente di tooling/rete, non ho quindi fermato il task (per le regole del
    brief: si ferma solo se serve cambiare un valore).
- **Nessuno scraping di stock LCSC**, come da istruzione esplicita: MPN citati
  sono presi da datasheet/pagine prodotto dei produttori (Nexperia, Murata, onsemi,
  Same Sky/CUI), mai da verifiche di disponibilità/prezzo.
- Valori string per i generic (`"22uF"`, `"7.5kohm"`, ecc.) sono indicativi nel
  formato tipico Zener/diode; Task 4 dovrà confermare la sintassi esatta
  contro `pcb doc --package @stdlib/generics/Resistor.zen` (o equivalente) in
  fase di scrittura del `.zen`.
