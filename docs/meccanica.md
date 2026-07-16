> **DECISIONE (2026-07-16) — montaggio in-connettore ABBANDONATO.** La prova di
> placement ha confermato che nell'NC3MXX (cavità non quotata sui datasheet) **non
> ci stanno tutti i 32 componenti**. Il progetto è passato a **scheda in guscio
> esterno di lamina di rame** (gabbia di Faraday) collegata alla scheda audio con un
> **cavetto schermato + spinotto XLR maschio volante** (NC3MX). Di conseguenza:
> board non più vincolata a 11.1mm (outline 50×40mm), **spessore 1.6mm**
> standard (non più 0.8mm), 4 strati mantenuti, `J1` = 3 piazzole THT a filo
> (`footprints/J1_XLR3_WirePads.kicad_mod`). L'analisi qui sotto sul fit dentro
> l'NC3MXX (Piano A/B, footprint sandwich) è **superata** e conservata solo come
> storico. Vedi README §"Fattore di forma: guscio esterno schermato".

# Verifica fit meccanico — electret2xlr dentro il Neutrik NC3MXX (Task 7) — STORICO/SUPERATO

Stato: **superato dalla decisione del 2026-07-16** (vedi riquadro sopra). Testo
originale conservato per storico.

## 1. Quote scheda e jack (dati di partenza, da Task 3/5/6)

- **PCB:** larghezza **11.1mm**, spessore **0.8mm**, 4 strati. Outline attuale in
  `layout/layout.kicad_pcb` (Edge.Cuts): rettangolo `(50,50)–(61.1,95)` →
  **11.1 × 45mm** (confermato leggendo il file; corrisponde al valore
  "provvisorio" citato nel commit di Task 6 `0528dec`).
- **Altezza assemblata:** 0.8mm PCB + massimo rilievo dei componenti lato top
  (~2.5mm, 1206/1210 e SOT-23) → **~3.3mm** di inviluppo verticale.
- **Sandwich XLR (J1):** i tre pin del NC3MXX (interasse bicchierini **7.62mm**)
  impegnano pad da **8mm** sul bordo scheda (pin1/2 fronte, pin3 retro), come da
  Task 6. Nessun connettore fisico: è il bordo PCB stesso a fare da lama nel
  sandwich.
- **Jack J2 — CUI/Same Sky SJ-3523-SMT-TR** (fonte: datasheet ufficiale
  "SJ-352X-SMT" rev 04/15/2025, letto in Task 3):
  - corpo (housing) **6.0mm × 14.5mm** (larghezza × lunghezza principale);
  - boccola/naso sporgente **2.5mm** oltre la faccia frontale del corpo →
    inviluppo utile fronte-retro **≈17mm** (coda terminali 2mm esclusa
    dall'inviluppo utile, sporge solo sul retro per la saldatura);
  - boccola: diametro esterno **Ø5.00mm**, foro interno **Ø3.60mm**;
  - altezza sopra PCB: **non quotata come cifra singola** nel disegno
    meccanico leggibile del datasheet; il valore desunto dalla tasca di
    imballo (packaging) è **6.1mm**, preso come stima conservativa **da
    confermare con lo STEP/3D ufficiale** (non scaricato/aperto finora).
  - footprint PCB consigliato dal datasheet: inviluppo **14.5 × 11.8mm**.
- **Precedente diretto — p48-pip-adapter** (stessa famiglia di connettore,
  stesso schema sandwich, verificato via GitHub
  `github.com/tphakala/p48-pip-adapter`, README): PCB **11.1 × 35.3 × 0.8mm**,
  *senza* jack (capsula saldata direttamente ai pad, PUI Audio AOM-5024). Note
  del progetto originale, citate testualmente:
  - "The thin edge has to fit into the gap between the three Neutrik pins; a
    standard 1.6mm board is far too thick" (conferma indipendente del vincolo
    0.8mm già adottato qui);
  - clearance dalla punta del pin al componente più vicino **≥5mm**, e per il
    pin3 la clearance del pad è scesa fino a **~0.5–0.7mm** (tolleranza molto
    stretta, tanto da richiedere pre-stagnatura e tecnica di saldatura
    dedicata);
  - il progetto include un **fit-test 3D-stampabile** prima di produrre, che è
    esattamente la pratica che raccomandiamo di ripetere qui (§6).

## 2. Quote ufficiali Neutrik NC3MXX reperite in questa sessione

Fonte primaria: datasheet ufficiale Neutrik, scaricato in questa sessione da
`https://www.neutrik.com/en/product/nc3mxx.pdf` (3 pagine, letto integralmente
pagina per pagina) e incrociato col catalogo generale
`https://www.neutrik.com/media/10098/download/01 NEUTRIK Section XLR
Connectors PG EN 202202-V23_opt.pdf` (sezione serie XX, riga "Cable O.D.
range").

Quote **effettivamente presenti in forma testuale/machine-readable**:

| Quota | Valore | Fonte |
|---|---|---|
| Cable O.D. accettato dal passacavo/ghiera posteriore | **3.5 – 8.0mm** | Datasheet NC3MXX pag. 2 ("Mechanical" → "Cable O.D."), confermato dal catalogo XLR generale (riga 500/571) |
| Wire size max | 2.5mm² / AWG 14 | Datasheet pag. 2 |
| Materiali/costruzione | shell zama nichelata, insert PA66, boot poliuretano | Datasheet pag. 3 |
| Rated voltage / dielectric strength | <50V / 1.5kVdc | Datasheet pag. 2 |

**Quote NON reperite in forma machine-readable in questa sessione** (dichiarato
esplicitamente, come richiesto dal brief — nessun numero è stato inventato):

- **Lunghezza/diametro utile della cavità interna per un PCB.** Il datasheet
  PDF ufficiale (2 varianti scaricate e ispezionate, più il catalogo XLR
  generale di 30 pagine passato a `pdftotext`) contiene solo tabelle
  elettriche/meccaniche generiche (vedi tabella sopra): **non contiene alcun
  disegno quotato in testo estraibile** — i disegni meccanici con quote sono
  presenti solo come immagini vettoriali nei file DXF/STEP ufficiali (link "3D
  Model"/"Drawing" sulla pagina prodotto Neutrik), che in questa sandbox non
  sono stati scaricabili/apribili (nessun visualizzatore CAD disponibile).
- **Diametro del foro/ghiera passacavo posteriore come cifra dimensionale
  diretta** (distinta dal range "Cable O.D." accettato). Il dato più vicino e
  ufficialmente quotato è il **Cable O.D. 3.5–8.0mm** sopra: è la specifica
  con cui il progettista Neutrik garantisce che il passacavo/boot/chuck a
  serraggio accetti cavi in quel range di diametro esterno — un proxy
  ragionevole ma non identico al "diametro del foro nudo".

Non ho inventato cifre di lunghezza cavità: dove serviva un numero e non era
disponibile, ho usato il **precedente reale p48-pip-adapter** (stesso
connettore, board 35.3mm fisicamente verificata) come riferimento empirico —
vedi §3.

## 3. Confronto fit

### 3.1 Larghezza e spessore assemblato — OK

- 11.1mm di larghezza scheda: **ampiamente sotto** qualunque lettura plausibile
  del diametro interno utile di uno shell XLR di questa classe (il corpo
  esterno stesso è nell'ordine di ~24mm di diametro per connettori cable-mount
  di questa famiglia — dato di conoscenza generale di settore, non citato da
  una fonte machine-readable Neutrik in questa sessione, quindi trattato come
  indicativo). Il precedente p48-pip-adapter usa la stessa larghezza 11.1mm
  nello stesso connettore e **è un prodotto reale, con fit verificato
  fisicamente dall'autore**: la larghezza non è un problema.
- Spessore assemblato ~3.3mm (0.8 PCB + ~2.5 componenti): stesso discorso, il
  precedente ha componenti equivalenti (SOT-23, 1206/1210) nello stesso guscio.
  **Nessun rischio noto.**

### 3.2 Lunghezza — punto critico, non confermabile a tavolino

Il precedente p48-pip-adapter (stesso connettore, nessun jack) ha impegnato
**35.3mm** di lunghezza per: zona sandwich pin XLR + tutto il circuito attivo
(BJT, zener, capacità, resistenze — un set di componenti molto simile al
nostro), lasciando solo **~5mm** di clearance dalla punta pin al componente più
vicino, e clearance ancora più risicata (~0.5–0.7mm) sul pad del pin3. Questo è
un'indicazione forte che **35mm circa è già vicino al limite pratico** di
lunghezza utilizzabile in questo connettore per un PCB analogo, verificato
empiricamente da chi ha stampato un fit-test 3D reale.

Il nostro circuito (`electret2xlr.zen`) replica lo stesso set di componenti del
precedente **più**:
- J2 (jack TRS SJ-3523-SMT-TR): inviluppo utile **~17mm** fronte-retro (14.5mm
  corpo + 2.5mm boccola sporgente);
- MIC1 (piazzole capsula, in parallelo al jack): ingombro trascurabile (2 THT
  pad Ø1mm passo 2.54mm).

Se il jack viene semplicemente accodato al layout del precedente senza
sovrapposizioni, la lunghezza necessaria sale a grandezza **35.3 + ~14.5 ≈
50mm** (il corpo del jack, non la sola boccola: la boccola dei 2.5mm sporge
*oltre* il bordo scheda dentro il passacavo, quindi non consuma ulteriore
lunghezza di PCB). L'outline attuale in `layout.kicad_pcb` è **45mm**, quindi
**circa 5mm più corto** di questa stima "somma diretta" — ma il vero margine
dipende dal placement effettivo (Task 6 ha volutamente lasciato
placement/routing all'utente: i footprint nel file sono ancora alle
coordinate di import del netlist, non posizionati sul board reale), quindi
45mm potrebbe bastare se c'è sovrapposizione/ottimizzazione nel piazzamento, o
potrebbe essere già insufficiente.

**Non è possibile risolvere questo punto solo da datasheet**: manca la quota
Neutrik della cavità interna utile (§2), e il precedente stesso mostra che il
margine reale in questo connettore è ridotto anche senza jack. Serve un
fit-test fisico (§6) prima di congelare la lunghezza.

**Raccomandazione lunghezza finale:** portare l'outline a **50mm** (rispetto
ai 45mm attuali) come target di lavoro per il placement in Task 6/KiCad, per
dare margine al jack pieno, **ma trattare 50mm come "da verificare fisicamente
prima di ordinare"**, non come quota garantita — vedi Piano B sotto se il
fit-test dimostra che lo shell non ospita 50mm.

### 3.3 Boccola jack (Ø5.00mm) vs passacavo/ghiera posteriore NC3MXX

- Il datasheet Neutrik quota il **Cable O.D. accettato dal passacavo/chuck a
  **3.5 – 8.0mm**. La boccola del jack SJ-3523-SMT-TR ha diametro esterno
  **Ø5.00mm**, quindi **rientra numericamente nel range dichiarato** per il
  passaggio dietro la ghiera/boot.
- Attenzione: quel range è pensato per un **cavo rotondo flessibile** stretto
  dal chuck a serraggio, non per un boss plastico rigido di forma leggermente
  diversa (il naso del jack è cilindrico ma con un piccolo gradino/flangia).
  Un Ø5mm rigido dovrebbe passare, ma la tenuta/serraggio del chuck non è
  garantita identica a quella su un cavo — **da verificare fisicamente**, non
  solo per numero.
- Nessuna necessità di forare il bushing standard: 5.00mm è già dentro il
  range 3.5–8.0mm, quindi **non serve sostituire/allargare il passacavo** per
  questa parte — il punto aperto resta solo la lunghezza (§3.2), non il
  diametro.

## 4. Riepilogo tabellare fit

| Verifica | Esito | Confidenza |
|---|---|---|
| Larghezza 11.1mm nella cavità | OK | Alta (precedente reale verificato) |
| Spessore assemblato ~3.3mm | OK | Alta (precedente reale verificato) |
| Lunghezza 45mm attuale (senza jack ottimizzato) | **Dubbio/probabilmente corto** | Bassa — nessuna quota Neutrik di cavità disponibile |
| Lunghezza 50mm raccomandata (con jack pieno) | Da verificare fisicamente | Bassa, stessa ragione |
| Boccola jack Ø5.00mm nel passacavo (range dichiarato 3.5–8.0mm) | Numericamente compatibile | Media (dato ufficiale Neutrik, ma per cavo non per boss rigido) |

## 5. Decisione sul montaggio del jack — DA CONFERMARE CON L'UTENTE

Non è possibile, sulla sola base dei datasheet, garantire che l'inviluppo
completo del jack J2 (≈17mm utili + il resto del circuito, board totale
~50mm) entri nella cavità interna del NC3MXX: la quota di cavità non è
pubblicata da Neutrik in forma machine-readable, e il precedente reale
(p48-pip-adapter) suggerisce che il margine disponibile in questo connettore è
già stretto **senza** alcun jack (35.3mm quasi al limite). Questo è un
genuino bivio, coerente con quanto la spec di progetto aveva già segnalato
come "rischio aperto":

**Piano A — jack J2 sul bordo posteriore della scheda (schema attuale)**
- Pro: mic 3.5mm TRS collegabile direttamente, nessun cavo volante, coerente
  col resto del progetto già autorato (`electret2xlr.zen`, footprint
  `Jack_3.5mm_CUI_SJ-3523-SMT_Horizontal` già presente).
- Contro: richiede board ~50mm (vs 35.3mm del precedente verificato),
  boccola Ø5mm da far sporgere nel passacavo esistente (numericamente
  compatibile ma non verificato fisicamente), **rischio concreto di non
  entrare nello shell** senza conferma da fit-test.

**Piano B — capsula su cavetto flying-lead attraverso il pressacavo standard
(piano B già previsto nella spec)**
- Il jack J2 esce dal PCB; la capsula/mic esterno si collega via un breve
  spezzone di cavo che esce dal pressacavo/boot standard del NC3MXX (lo
  stesso percorso già usato per il cavo microfonico in un connettore XLR
  cable-mount "normale").
- Pro: la board torna dimensionalmente al precedente **verificato fisicamente**
  (~35.3mm, stesso schema pin-sandwich, stesso ingombro dei componenti):
  **rischio di fit sostanzialmente azzerato**, perché è lo stesso schema già
  collaudato dal progetto sorgente.
- Contro: si perde la comodità "spina 3.5mm diretta nel corpo XLR"; l'utente
  deve saldare/assemblare un piccolo pigtail con jack esterno o terminazione
  a piacere; JJ2 (footprint/Component già scritto in `electret2xlr.zen`)
  andrebbe rimosso o reso opzionale/DNP e sostituito da un'uscita su cavetto.

**Raccomandazione:** dato che il precedente reale mostra margini già stretti
senza jack, e che non esiste una quota Neutrik ufficiale che confermi lo
spazio per un jack intero, **il Piano B è l'opzione a rischio meccanico più
basso**. Il Piano A resta preferibile se il fit-test fisico (§6) lo conferma,
perché offre più comodità d'uso. **Questa scelta va confermata dall'utente**
prima di congelare Task 6 (placement/routing) in via definitiva — segnalato
come DONE_WITH_CONCERNS in uscita da questo task.

## 6. Fit-test fisico raccomandato (prima di ordinare)

Indipendentemente dal piano scelto, **non ordinare PCB/connettori prima di
un test fisico**:

1. **Sagoma cartoncino 11.1 × L mm** (L = 45mm attuale, 50mm raccomandato per
   Piano A, o 35.3mm per Piano B), tagliata a spessore reale se possibile
   (impilare 2 strati di cartoncino ≈0.8mm), inserita a mano nello shell
   NC3MXX aperto (senza il retro/boot montato) per verificare visivamente lo
   spazio libero fino al boot/pressacavo.
2. In alternativa, più affidabile: **ordinare solo il PCB nudo** (senza
   componenti) nella lunghezza scelta e verificarne l'inserimento reale nel
   connettore comprato a parte, prima di ordinare l'intera BOM.
3. Se si sceglie Piano A, verificare *anche* che la boccola del jack (o un
   cilindretto di prova Ø5mm × ~5mm) passi effettivamente attraverso il
   pressacavo/boot montato, non solo attraverso lo shell aperto.
4. Ripetere l'esercizio del progetto originale p48-pip-adapter: un modello
   3D-stampabile del solo ingombro scheda+jack, infilato nello shell reale,
   resta il modo più economico per chiudere questo rischio senza attendere
   una fornitura PCB.

## 7. Incertezze aperte (riepilogo)

- **Quota ufficiale Neutrik della lunghezza/cavità interna utile**: non
  reperita in forma machine-readable in questa sessione (datasheet PDF e
  catalogo XLR generale ispezionati per intero, nessun disegno quotato in
  testo estraibile; i file DXF/STEP ufficiali non sono stati aperti, nessun
  visualizzatore CAD disponibile in sandbox). Uso di un precedente reale
  (p48-pip-adapter, 35.3mm) come miglior proxy disponibile, esplicitamente
  segnalato come tale.
- **Diametro effettivo del foro/ghiera nuda** (a differenza del range
  "Cable O.D." 3.5–8.0mm accettato dal chuck): non quotato separatamente da
  Neutrik in forma testuale.
- **Altezza reale del jack SJ-3523-SMT-TR sopra il PCB**: stimata a 6.1mm da
  una quota di imballo (packaging), non da un disegno meccanico dedicato —
  ereditata da Task 3, non rivalutabile senza lo STEP ufficiale.
- **Placement reale dei componenti**: Task 6 ha lasciato piazzamento e
  routing all'utente (i footprint sono ancora alle coordinate di import del
  netlist in `layout.kicad_pcb`, fuori dall'outline 11.1×45mm); la stima di
  lunghezza in questo documento (§3.2) è quindi basata su ingombri sommati,
  non su un placement verificato in KiCad.
