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
- Back-to-back (consecutive-byte) transmission was attempted but not completed successfully; removed from the current testbench for further debugging later

## Bugs found and fixed
- Blocking/non-blocking mix in `uart_rx` — standardized `rdy` assignment to non-blocking (`<=`) throughout
- TX/RX baud drift — `tx_en` and `rx_en` were generated from two independently-tuned counters with a slight mismatch (435 vs. required 448 cycles for an exact 16:1 ratio), causing accumulating timing drift; fixed by deriving `tx_en` directly from counting 16 `rx_en` pulses

## Planned
- SystemVerilog/UVM testbench, once covered in coursework (ADLV elective)
- Revisit back-to-back transmission handling