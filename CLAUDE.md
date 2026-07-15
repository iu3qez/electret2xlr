# electret2xlr

Adattatore microfono electret → XLR bilanciato alimentato da phantom 48V, descritto in pcb-as-code con il toolchain `pcb` / linguaggio Zener di Diode Inc (https://github.com/diodeinc/pcb).

## Preferenze componenti

- **Preferire sempre componenti già presenti a stock personale oppure a stock presso LCSC.** In fase di scelta parti, verificare la disponibilità LCSC (preferire le "basic/preferred parts" quando possibile).
- **Non scrapare lcsc.com a mano** (JS-heavy, rate-limited): per la verifica strutturata di stock/basic-part usare il DB **jlcparts** (yaqwsx, SQLite scaricabile) interrogato via SQL. Per questo progetto specifico, parti tutte non critiche → conta solo il footprint corretto, non lo stock: usare `registry-search` di diode.
- Stock personale noto (parziale): opamp LM8261, LM358D.
- **Footprint minimo 0603**: mai usare componenti più piccoli di 0603 (saldabilità a mano).

## Toolchain

- `pcb` (Zener) è un binario Rust, **non** è su PyPI: si installa con lo script ufficiale `curl -fsSL https://raw.githubusercontent.com/diodeinc/pcb/main/install.sh | bash` (finisce in `~/.local/bin`). Richiede KiCad 10.x per il layout. (`uv` è comunque disponibile per eventuale tooling Python di contorno.)
- Il repo diodeinc/pcb fornisce skill dedicate in https://github.com/diodeinc/pcb/tree/main/skills (`zener-language`, `registry-search`, `librarian`, `datasheet-reader`, `spice-sim`), vendorizzate in `.claude/skills/`.
- **`pcb auth login`** apre un OAuth con callback su `localhost:<porta-random>`: via SSH serve un tunnel `ssh -L <porta>:localhost:<porta>` (la porta cambia a ogni avvio) e aprire l'URL stampato nel browser locale.
- **L'indice del registry diode non è scaricabile in questo ambiente** anche da loggati: i componenti attivi/connettori sono `Component()` primitivi su simboli KiCad (in `symbols/`) e footprint vendorizzati (in `footprints/`), non moduli del registry.

## Contesto d'uso

- Ambiente radioamatoriale: la scheda audio è collegata (indirettamente) a un trasmettitore HF → l'immunità RF è un requisito di progetto, non un optional.
- Filosofia KISS: circuito **attivo** ma minimale (cap multiplier + doppio follower PNP bilanciato in impedenza), pulito e funzionale. Topologia ispirata a github.com/tphakala/p48-pip-adapter (riscritta da zero; la CC BY-NC del progetto originale copre il suo design concreto, non la topologia).
- **Il routing PCB lo fa l'utente** in KiCad: gli step di layout consegnano una board pronta (stackup, outline, footprint, placement) e si fermano lì.

## Stato progetto (2026-07-15)

Circuito completo e verificato (`pcb build` pulito, 32 componenti; SPICE: VPIP ~7.5V, ~3mA/pin, risposta piatta in banda, taglio −3dB ~30Hz). Layout 4 strati / 0.8mm / 11.1mm con footprint sandwich J1 custom, pronto per placement+routing. Documentazione in `README.md`, BOM in `docs/bom.md`, spec/piano in `docs/superpowers/`.

Aperto: **form factor da decidere dopo un fit-test fisico** — Plan A jack posteriore ~50mm vs Plan B capsula flying-lead ~35.3mm (vedi `docs/meccanica.md`; la cavità NC3MXX non è quotata sui datasheet). CB2/CB3 ratificati a **1µF** (non 22µF). Nessun codice LCSC verificato a stock (parti non critiche).
