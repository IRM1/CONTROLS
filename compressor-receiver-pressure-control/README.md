# Compressor Receiver Pressure Control (PLC / Ladder Logic)

A beginner ladder logic project built on an Allen-Bradley SLC 500 using RSLogix 500 and Emulate 500. Keeps receiver pressure in a 90–110 psi deadband using two pressure switches, one pump, and a run indicator light.

> Training/practice project — not deployed.

## I/O List

| Address | Type   | Description              |
|---------|--------|--------------------------|
| I:0/0   | Input  | Low pressure switch (90 psi)  |
| I:0/1   | Input  | High pressure switch (110 psi) |
| O:0/0   | Output | Pump                     |
| O:0/1   | Output | Run indicator light      |

## Concepts

Deadband control, seal-in latching, TON timers.

## Screenshots

Ladder logic screenshots are in the [`docs/`](docs/) folder.
