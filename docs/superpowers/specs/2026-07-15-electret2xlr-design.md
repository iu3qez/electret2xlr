# electret2xlr — Design

**Data:** 2026-07-15
**Stato:** approvato

## Scopo

Adattatore attivo che collega un microfono electret (capsula a 2 terminali saldata, oppure mic da headset/lavalier via jack 3.5mm TRS) a un ingresso XLR bilanciato di una scheda audio, alimentato esclusivamente dalla phantom P48 (48V standard, 6.8kΩ per ramo). Tutto il circuito vive su un PCB montato dentro il corpo di un connettore XLR maschio Neutrik NC3MXX.

Contesto d'uso: shack radioamatoriale con trasmettitore HF collegato (indirettamente) alla stessa scheda audio → **l'immunità RF è un requisito di progetto**, non un optional.

Filosofia: KISS ma pulita e funzionale. Descrizione hardware in pcb-as-code con il toolchain `pcb`/Zener di Diode Inc.

## Origine della topologia

Il circuito adotta la topologia del progetto [tphakala/p48-pip-adapter](https://github.com/tphakala/p48-pip-adapter) (licenza CC BY-NC 4.0): il sorgente Zener è riscritto da zero, il progetto originale va citato nel README come ispirazione, l'uso resta non commerciale.

Motivazioni della scelta attiva (rispetto all'alternativa passiva impedance-balanced valutata e scartata):

1. **Rumore zener:** in una capsula a 2 terminali il nodo di bias è anche il nodo del segnale; in un design passivo il rumore di valanga dello zener finisce quasi direttamente nel segnale. Il moltiplicatore di capacità lo elimina (taglio RC < 1Hz).
2. **Impedenza d'uscita:** ~78Ω contro i ~kΩ del passivo → guida cavi lunghi senza perdite HF e con migliore immunità.
3. **Bias più alto:** ~7.5V contro 3–4V → più headroom per la capsula.
4. **RF:** la difesa non è "meno giunzioni" ma schermatura (PCB 4 strati con piani di massa interni + guscio XLR) più filtraggio esplicito.

## Circuito

Tre stadi funzionali (~15 componenti):

### Alimentazione (moltiplicatore di capacità)

- Prelievo simmetrico dai pin 2 e 3.
- Zener 8.2V (SOD-323) come riferimento grezzo.
- Filtro RC 100kΩ + 22µF: corner sotto 1Hz, elimina il rumore di valanga dello zener.
- Emitter follower NPN Q1 (MMBT3904) → rail PIP ~7.5V pulito, bypass 10µF.
- Bias capsula: 6.8kΩ dal rail PIP al nodo segnale/tip.

### Stadio audio (follower in classe A, uscita impedance-balanced)

- Q2 (MMBT3906, PNP): base al segnale della capsula, collettore su pin 2 (via rete), emettitore pilota il ramo hot. Rete di emettitore: resistenza di feed (~7.5kΩ) + bypass 22µF/50V X7R con stopper 47Ω.
- Q3 (MMBT3906): identico ma con base a massa in AC, collettore su pin 3, feed ~10kΩ → ramo cold con impedenza identica al hot ma senza audio.
- Risultato: sorgente pseudo-bilanciata ~78Ω, piatta in banda audio (−0.56dB simulati nel progetto originale), roll-off deliberato sotto ~50Hz (bypass di emettitore).
- Assorbimento ~3mA per pin, simmetrico → compatibile con ingressi bilanciati elettronici e a trasformatore.
- Condensatori di accoppiamento verso i pin XLR: 22µF/50V (siedono su ~23V DC → rating 50V obbligatorio).

### Ingresso

- **Jack 3.5mm TRS PCB-mount** sul retro: tip = segnale/bias, ring ponticellato al tip (compatibilità lavalier TRS), sleeve = GND. Le cuffiette TRRS da smartphone richiedono un adattatore esterno (scelta deliberata: circuito pulito).
- **Piazzole THT** in parallelo al jack per saldare direttamente una capsula a 2 terminali.
- **Anti-RF sull'ingresso** (il cavo headset è un'antenna): 100Ω serie + 220pF C0G direttamente sul connettore/piazzole, a protezione del FET interno della capsula (unico rivelatore AM non schermabile).

### Difesa RF aggiuntiva

- 100pF C0G da pin 2 → pin 1 e da pin 3 → pin 1, fisicamente adiacenti ai pin XLR.
- Ferrite (BLM18-class) in serie sul ramo segnale.
- 1nF tra pin 1 e piazzola guscio.
- Connettore Neutrik NC3MXX-EMC (contatto capacitivo shell→schermo) se disponibile; NC3MXX standard come ripiego.

## PCB e meccanica

Schema meccanico adottato in blocco dal p48-pip-adapter (collaudato):

- **4 strati**, strati interni = piani di massa pieni via-stitched (gabbia di Faraday attorno al front-end da µV).
- **Spessore 0.8mm — obbligatorio, non negoziabile** (1.6mm non entra tra i pin).
- Larghezza **11.1mm**; sandwich mount: il bordo del PCB si infila tra i tre pin dell'NC3MXX (interasse bicchierini 7.62mm), pad lunghi 8mm su entrambe le facce (pin 1 e 2 sul fronte, pin 3 sul retro), saldatura diretta pin→pad.
- Lunghezza: da estendere rispetto ai 35.3mm dell'originale per ospitare il jack TRS sul retro, che deve sporgere dalla ghiera posteriore. **Rischio aperto:** fit jack/ghiera da verificare su disegni quotati Neutrik + fit-test STL 3D-printabile prima di ordinare. Piano B: capsula/cavetto attraverso il pressacavo standard.
- Componenti solo sul lato top; 1206 per le resistenze di feed (dissipazione continua nel guscio chiuso, margine termico a 80°C).

## Componenti

- Tutti jellybean, **a stock LCSC** (preferire basic/preferred parts) o dallo stock personale, come da CLAUDE.md.
- **Vincolo footprint: nessun componente sotto lo 0603** (saldabilità a mano). Ammessi 0603/0805/1206, SOT-23, SOD-323.
- Transistor: MMBT3904 / MMBT3906 (SOT-23). Zener 8.2V SOD-323. Ceramici X7R 50V per i 22µF.
- Gli opamp a stock (LM8261, LM358D) sono stati valutati e scartati: non reggono 48V (servirebbe comunque il regolatore), LM358 rumoroso e con crossover, e si perderebbe il bilanciamento DC naturale dei collettori sui pin.
- I valori sono nominali: possibile bench tuning dopo il primo prototipo.

## Toolchain e struttura repo

- `pcb` (Zener) installato/eseguito via **uv**.
- Skill ufficiali diodeinc (repo pcb, cartella `skills/`): `zener-language` per scrivere i .zen, `registry-search`/`librarian` per componenti a stock LCSC, `datasheet-reader` per il fit meccanico, `spice-sim` per la verifica elettrica.
- Struttura repo:
  - `pcb.toml` — workspace Zener
  - `electret2xlr.zen` — board top-level (eventuali moduli separati per power/audio se utile)
  - `README.md` — schema, teoria di funzionamento, BOM con codici LCSC, attribuzione al p48-pip-adapter
  - `CLAUDE.md` — preferenze di progetto (già presente)

## Verifica

1. `pcb build` pulito (netlist + ERC).
2. `spice-sim`: punto di lavoro DC (rail PIP ~7.5V, assorbimento ~3mA simmetrico per pin, tensioni sui BJT sane) e risposta in frequenza (piatta in banda, roll-off <50Hz).
3. `pcb layout` → DRC KiCad zero errori; larghezza 11.1mm, spessore 0.8mm nei parametri di fabbricazione.
4. Fit meccanico: quote pad pin XLR contro datasheet Neutrik; fit-test fisico prima dell'ordine.
5. BOM: ogni parte con codice LCSC verificato a stock, o marcata "stock personale".

## Fuori scope

- Supporto TRRS/CTIA diretto (serve adattatore esterno).
- Phantom sotto 48V (P12/P24 non garantite).
- Uso commerciale (vincolo licenza CC BY-NC della topologia di riferimento).
