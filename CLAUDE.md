# electret2xlr

Adattatore microfono electret → XLR bilanciato alimentato da phantom 48V, descritto in pcb-as-code con il toolchain `pcb` / linguaggio Zener di Diode Inc (https://github.com/diodeinc/pcb).

## Preferenze componenti

- **Preferire sempre componenti già presenti a stock personale oppure a stock presso LCSC.** In fase di scelta parti, verificare la disponibilità LCSC (preferire le "basic/preferred parts" quando possibile).
- Stock personale noto (parziale): opamp LM8261, LM358D.
- **Footprint minimo 0603**: mai usare componenti più piccoli di 0603 (saldabilità a mano).

## Toolchain

- `pcb` (Zener) è un binario Rust, **non** è su PyPI: si installa con lo script ufficiale `curl -fsSL https://raw.githubusercontent.com/diodeinc/pcb/main/install.sh | bash` (finisce in `~/.local/bin`). Richiede KiCad 10.x per il layout. (`uv` è comunque disponibile per eventuale tooling Python di contorno.)
- Il repo diodeinc/pcb fornisce skill dedicate in https://github.com/diodeinc/pcb/tree/main/skills (`zener-language`, `registry-search`, `librarian`, `datasheet-reader`, `spice-sim`): usarle per scrivere .zen, cercare componenti e validare il circuito.

## Contesto d'uso

- Ambiente radioamatoriale: la scheda audio è collegata (indirettamente) a un trasmettitore HF → l'immunità RF è un requisito di progetto, non un optional.
- Filosofia KISS: circuito passivo, pochi componenti, ma pulito e funzionale.
