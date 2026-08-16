# UART TX/RX (Verilog)

Transmitter and receiver modules for asynchronous serial communication, 
verified with a plain Verilog testbench.

## Modules
- `uart_tx.v` — UART transmitter (idle → start → data → stop FSM)
- `uart_rx.v` — UART receiver, 16x oversampling (idle → start → data → stop FSM)
- `uart_baud.v` — Baud rate generator; `tx_en` derived from `rx_en` for a fixed 16:1 sample ratio
- `uart_top.v` — Top-level integration of TX, RX, and baud generator
- `tb/uart_tb.v` — Verilog testbench

## Verification
- All-zeros (`0x00`), all-ones (`0xFF`), alternating bits (`0xAA`) — each sent and received correctly, `data_out` checked against input
- Back-to-back transmission (multiple bytes sent with no gap) — passing, verified with `always @(posedge rdy)` to print each received byte as it completes


## Bugs found and fixed
- **Testbench race condition:** `wait(!busy)` was checked immediately after `send()`, before `busy` had actually risen, causing a false pass with no real wait. Fixed by adding `wait(busy)` before `wait(!busy)`.
- **`rdy` not cleared between checks:** back-to-back test wasn't clearing `rdy` between receptions, so a later `wait(rdy)` could pass on stale state from a previous byte instead of confirming new data.
- **TX/RX baud drift:** `tx_en` and `rx_en` were generated from two independently-tuned free-running counters (435 cycles vs. the required 448 for an exact 16:1 ratio), causing ~13-cycle drift per bit that accumulated across a byte and corrupted later bytes in back-to-back transmission. Fixed by deriving `tx_en` directly from counting 16 `rx_en` pulses, guaranteeing an exact ratio with zero drift.
- **`rdy_clr` freezing the receive FSM:** `rdy_clr` and the FSM `case` statement were in mutually exclusive `if/else` branches, so any cycle where `rdy_clr` was asserted, the entire state machine (state, sample counter, bit counter) froze for that cycle — even if `rx_en` pulsed on the same edge. Fixed by decoupling `rdy_clr` from the FSM so both can run independently in the same cycle.


- ## OUTPUT
<img width="1547" height="387" alt="Screenshot 2026-08-16 181345" src="https://github.com/user-attachments/assets/a0920e4a-cde4-49fb-9930-e10c938086d6" />
<img width="317" height="172" alt="Screenshot 2026-08-16 181411" src="https://github.com/user-attachments/assets/51f20c4a-9b08-4bc4-83c1-50ce536efd3b" />
