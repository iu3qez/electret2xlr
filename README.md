# electret2xlr

Adattatore attivo che alimenta un microfono electret dalla phantom power P48 e ne porta il segnale a un ingresso XLR bilanciato, con tutta l'elettronica alloggiata dentro il corpo di un connettore Neutrik NC3MXX.

## Perché

Un microfono electret ha bisogno di qualche volt di bias e presenta un'uscita sbilanciata ad alta impedenza; un ingresso XLR "vero" si aspetta invece una sorgente bilanciata bassa impedenza alimentata a 48V/6.8kΩ per ramo (P48). electret2xlr fa da ponte fra i due mondi: prende i 48V phantom dai pin 2/3 dell'XLR, li riduce a un bias pulito per la capsula, e rimanda indietro un'uscita pseudo-bilanciata a bassa impedenza sugli stessi due pin — senza alcuna alimentazione esterna.

Contesto d'uso: shack radioamatoriale con un trasmettitore HF collegato (indirettamente) alla stessa scheda audio. L'immunità RF è quindi un requisito di progetto fin dall'inizio, non un'aggiunta a posteriori.

## Teoria di funzionamento

Il circuito è organizzato in tre stadi (~15 componenti attivi/passivi oltre ai connettori):

### 1. Alimentazione (moltiplicatore di capacità)

I 48V phantom vengono prelevati simmetricamente dai pin 2 e 3 dell'XLR. Uno zener D1 da 8.2V (SOD-123) dà un riferimento di tensione grezzo; il filtro RC R9 (47kΩ) + C4 (47µF) porta il corner sotto 1Hz, eliminando il rumore di valanga dello zener prima che raggiunga il segnale. Un emitter follower NPN (Q1, MMBT3904) bufferizza il riferimento filtrato producendo il rail di alimentazione pulito **VPIP ≈ 7.5V** (bypass C5 10µF), da cui è derivato anche il bias della capsula tramite R10 (6.8kΩ) verso il nodo segnale/tip.

### 2. Stadio audio (follower in classe A, uscita impedance-balanced)

Due emitter follower PNP appaiati (MMBT3906) generano l'uscita bilanciata:

- **Q2 (ramo "caldo")**: base accoppiata in AC al segnale della capsula (C2, 22µF), collettore a massa, emettitore alimentato da pin 2 attraverso R1 (7.5kΩ) — la corrente di segnale modula l'assorbimento su pin 2. CB2 (1µF/50V X7R) + RB2 (47Ω stopper) bypassano R1 alle frequenze audio, abbassando l'impedenza d'uscita vista dal pin.
- **Q3 (ramo "freddo")**: identico a Q2 ma con la base messa a massa in AC (C3), emettitore alimentato da pin 3 via R2 (10kΩ), bypass gemello CB3+RB3. Nessun segnale sulla base: il ramo freddo replica solo l'impedenza del ramo caldo, senza audio.

Il risultato è una sorgente pseudo-bilanciata a **~78Ω** (valore di progetto ereditato dalla topologia di riferimento, non ricalcolato nella verifica SPICE di questo progetto), piatta in banda audio, con un roll-off passa-alto deliberato introdotto dal bypass di emettitore (non un difetto: serve a togliere di mezzo rumore e derive sub-audio). Assorbimento ~3mA per pin, simmetrico entro il 5% — compatibile sia con ingressi bilanciati elettronici sia a trasformatore.

CB2/CB3 sono impostati a **1µF** (non 22µF): un valore più alto avrebbe spostato il ginocchio passa-alto sotto 1Hz, vanificando il roll-off voluto attorno ai 30Hz — vedi `docs/sim-results.md` per la derivazione.

### 3. Ingresso capsula/jack

- **Piazzole THT** (MIC1) in parallelo, per saldare direttamente una capsula electret a 2 terminali.
- **Jack 3.5mm TRS** opzionale (J2, PCB-mount): tip = segnale/bias, ring ponticellato al tip (compatibilità capsule/lavalier a 2 terminali su connettore TRS), sleeve = GND. Le cuffiette **TRRS da smartphone non sono supportate direttamente** — serve un adattatore esterno TRRS→TRS (scelta deliberata, per non complicare il circuito con la commutazione di un quarto contatto).
- **Filtro anti-RF a π** sull'ingresso (CIN 220pF C0G + FB1 ferrite 600Ω@100MHz + RIN 100Ω): il cavo verso la capsula/jack è la principale antenna del sistema, quindi è la prima linea di difesa contro l'ingresso HF che altrimenti si rivelerebbe sul FET interno della capsula (unico punto non schermabile).

### Difesa RF aggiuntiva

Oltre al filtro d'ingresso: CP2/CP3 (100pF C0G, pin2→GND e pin3→GND, adiacenti ai pin XLR), un condensatore CSH (1nF) fra pin1 e la piazzola guscio, e un PCB a 4 strati con piani di massa interni via-stitched che fa da gabbia di Faraday attorno al front-end a bassissimo segnale. CSH è pensato per il connettore **NC3MXX standard**; se si monta la variante **NC3MXX-EMC** (contatto capacitivo shell→schermo già integrato), CSH va marcato **DNP** per non duplicare la capacità.

## Figure chiave (verificate via SPICE)

Da `docs/sim-results.md` (ngspice via `pcb sim`, criteri di accettazione della spec):

| Grandezza | Valore simulato | Criterio |
|---|---|---|
| V(VPIP) | 7.49 V | 7.0–8.0 V |
| V(MICOUT) | 4.09 V | 3–5 V |
| Corrente per pin (P2 / P3) | 3.00 mA / 3.13 mA | 2.5–3.5 mA |
| Sbilanciamento P2 vs P3 | 4.4 % | < 10 % |
| Piattezza 100Hz–20kHz | 0.38 dB | ±1 dB |
| Roll-off basso (−3dB) | ≈ 30.6 Hz | tra 20 e 60 Hz |

Nota: la fase dell'uscita differenziale (P2−P3) a 1kHz risulta ≈ −179° (sostanzialmente invertita rispetto al segno convenzionale atteso) — comportamento noto e condiviso con il progetto di origine (issue #6 di p48-pip-adapter), non corretto in questa revisione: un eventuale scambio hot/cold è una verifica da banco, non da tavolino. Dettagli in `docs/sim-results.md`.

## Setup su una nuova macchina

Prerequisiti:

- **`pcb`** (toolchain Zener di diode) — binario Rust, **non** su PyPI:
  ```bash
  curl -fsSL https://raw.githubusercontent.com/diodeinc/pcb/main/install.sh | bash
  export PATH="$HOME/.local/bin:$PATH"   # aggiungilo al tuo profilo shell
  pcb --version                          # atteso: pcbc 0.4.x
  ```
- **KiCad 10.x** (serve solo per `pcb layout` / editor). Verifica con `kicad-cli version`. Opzioni di installazione, in ordine di comodità:
  - **AppImage ufficiale** dalla pagina download di KiCad (nessun sudo, non tocca il pacchetto di sistema) — consigliata se hai già una KiCad 9 installata;
  - **Flatpak**: `flatpak install flathub org.kicad.KiCad` (di solito già all'ultima release);
  - pacchetto della distro, se offre già la 10.x.
- `git` per clonare il repo. `ngspice` **solo** se vuoi rieseguire la verifica SPICE (facoltativo).

Poi:

```bash
git clone https://github.com/iu3qez/electret2xlr && cd electret2xlr
pcb build electret2xlr.zen    # atteso: 32 componenti, nessun errore
```

> **Non serve `pcb auth login` né accesso al registry online.** Footprint e simboli sono vendorizzati nel repo (`footprints/`, `symbols/`), quindi un clone pulito builda offline. Il login diode servirebbe solo per attingere a componenti dal registry, cosa che questo progetto non fa.

## Build

Toolchain: [`pcb`](https://github.com/diodeinc/pcb) (Zener), versione usata in questo progetto `pcbc 0.4.7`, installato con lo script ufficiale (**non** da PyPI):

```bash
curl -fsSL https://raw.githubusercontent.com/diodeinc/pcb/main/install.sh | bash
```

Serve inoltre **KiCad 10.x** per l'editor di layout. I footprint e i simboli usati (SOT-23, jack TRS, XLR3, le piazzole a filo custom `J1_XLR3_WirePads` per lo spinotto XLR volante) non sono presi dal registry online — che non è raggiungibile in ambienti sandbox senza rete/auth — ma **vendorizzati localmente** in `footprints/` e `symbols/`.

```bash
pcb build electret2xlr.zen   # genera netlist + ERC
pcb layout electret2xlr.zen  # apre/aggiorna il layout in KiCad
```

Il file `layout/layout.kicad_pcb` importato riflette board a 4 strati, 1.6mm di spessore, con outline provvisorio 30×40mm (guscio esterno — vedi §Fattore di forma). **Placement e routing sono lasciati intenzionalmente all'utente in KiCad**: i footprint importati sono ancora alle coordinate di import del netlist, non è stato eseguito alcun autorouting né posizionamento definitivo. La DRC su questo stato riporta i 51 net non connessi (da instradare) e alcune violazioni di sovrapposizione dovute a `J1` auto-piazzato sopra il gruppo C4/Q2/J2: si azzerano spostando i componenti in fase di placement.

Per la verifica SPICE (facoltativa, richiede un binario `ngspice` raggiungibile):

```bash
NGSPICE=/path/to/ngspice pcb sim electret2xlr.zen \
  --config sim_mode=true --setup testbench/setup_dc_ac.cir -v
```

`sim_mode=true` esclude dal netlist SPICE i due test point (TP MIC1_1/MIC1_2), privi di modello SPICE proprio nello stdlib; in build normale (`pcb build`, senza `--config`) restano popolati come piazzole reali.

## Fattore di forma: guscio esterno schermato

Il montaggio dentro il corpo del connettore XLR (sandwich in-connettore, come il `p48-pip-adapter`) è stato **abbandonato**: la cavità dell'NC3MXX è troppo piccola per piazzarci tutti i 32 componenti. La scheda vive invece in un **guscio esterno di lamina di rame**, che fa da gabbia di Faraday, collegata alla scheda audio con un **cavetto schermato che termina in uno spinotto XLR maschio volante** (es. Neutrik NC3MX).

Conseguenze di progetto:

- La board **non è più vincolata a 11.1mm**: outline comodo (default provvisorio 30×40mm nel layout, da adattare al tuo guscio), componenti su **entrambi i lati**.
- **Spessore standard 1.6mm** (non più 0.8mm: quello serviva solo a infilarsi tra i pin dell'XLR).
- **4 strati mantenuti** con i piani di massa interni: doppia schermatura, guscio di rame + piani interni.
- Il guscio di rame è lo **schermo**: va **collegato a massa (GND, cioè pin 1)** in almeno un punto — tipicamente saldando un lembo del rame a una piazzola GND della board. Il connettore `J1` nel netlist sono le 3 piazzole THT a filo (`footprints/J1_XLR3_WirePads.kicad_mod`) verso il cavetto: pin1=GND/schermo, pin2=hot, pin3=cold.

## Montaggio e cablaggio XLR

1. Saldare i tre fili del cavetto schermato alle piazzole THT `J1`: **pin1=GND** (allo stesso capo va anche la calza del cavetto e il lembo del guscio di rame), **pin2=hot (P2)**, **pin3=cold (P3)**.
2. All'altro capo del cavetto, cablare lo spinotto XLR maschio volante (NC3MX) rispettando lo standard: **pin1=schermo/GND, pin2=hot (+), pin3=cold (−)**.
3. Chiudere la board nel guscio di lamina di rame, bondando il rame a GND. Verificare che il rame non tocchi piazzole a tensione (isolare con nastro/kapton dove serve).

**Nodo SHELL:** il pad di contatto col guscio di rame (nodo `SHELL`, dove arriva CSH) è una feature di rame da disegnare **a mano nel layout KiCad** e da collegare a GND — non è un componente della netlist. `pcb build` segnala perciò un warning ERC di net a singolo pin per `SHELL`: è atteso e corretto, non un errore.

## Cablaggio capsula / jack

- **Capsula a 2 terminali saldata diretta**: sulle piazzole THT MIC1 (passo 2.54mm), segnale/bias su un pad, GND sull'altro — nessuna polarità critica lato pad (la capsula stessa ha verso, va rispettato secondo il suo datasheet). Il cavetto della capsula esce dal guscio schermato.
- **Jack 3.5mm TRS (J2)**, se montato sul guscio: **tip = segnale/bias**, **ring** ponticellato al tip (compatibilità con capsule TRS a 2 terminali che usano ring per il secondo contatto), **sleeve = GND**.
- **Cuffiette/headset TRRS da smartphone non sono utilizzabili direttamente**: serve un adattatore esterno TRRS→TRS. Non è supportato un quarto contatto (es. commutazione mic/altoparlante) su questo jack.

## Attribuzione e licenza

La topologia circuitale è ispirata al progetto [tphakala/p48-pip-adapter](https://github.com/tphakala/p48-pip-adapter). La licenza CC BY-NC 4.0 di quel progetto copre il suo *design concreto* (layout, gerber, documentazione), non la topologia in sé — un'idea circuitale funzionale ampiamente pubblicata e non proteggibile come tale. Sorgente Zener, layout e documentazione di questo repository sono scritti da zero: nessun vincolo di licenza è quindi ereditato. L'attribuzione qui è una cortesia verso il progetto originale, non un obbligo legale.

## Documenti di riferimento

- `docs/superpowers/specs/2026-07-15-electret2xlr-design.md` — spec di progetto completa.
- `docs/sim-results.md` — verifica SPICE (DC + AC).
- `docs/meccanica.md` — verifica del fit meccanico nel NC3MXX e decisione Piano A/B.
- `docs/bom.md` — distinta base finale.
