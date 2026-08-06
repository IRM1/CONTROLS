# Compressor Receiver Pressure Control (PLC / Ladder Logic)

A beginner ladder logic project built on an Allen-Bradley SLC 500 using RSLogix 500 and Emulate 500. Keeps receiver pressure in a 90–110 psi deadband using two pressure switches, one pump, and a run indicator light.

> Training/practice project — not deployed.

## I/O List

| Address | Type   | Description                    |
|---------|--------|--------------------------------|
| I:0/0   | Input  | Low pressure switch (90 psi)   |
| I:0/1   | Input  | High pressure switch (110 psi) |
| O:0/0   | Output | Pump                           |
| O:0/1   | Output | Run indicator light            |

## Concepts

Deadband control, seal-in latching, TON timers.

## Screenshots

![Main program](docs/LAD%202-%20Main.png)
*Main program — pump/light control, timers, seal-in logic*

![I/O subroutine](docs/LAD%203-%20I%3AO.png)
*I/O subroutine — maps physical I/O to internal bits*
