# CompactLogix Timed Motor Control

## Overview
A timed motor control sequence on an Allen-Bradley CompactLogix PLC, programmed in Studio 5000 and tested on a physical lab station. Pressing Start runs a 24 VDC DC motor and a green light for 5 seconds using a TON on-delay timer (preset 5000 ms), then stops them and latches a red light. Pressing Stop or E-Stop during operation does the same immediately.

I/O mapping:
- Start button (N.O.) — `Local:1:I.Data.2`
- Stop button (N.C.) — `Local:1:I.Data.1`
- E-Stop (N.C.) — `Local:1:I.Data.0`
- Motor — `Local:1:O.Data.6`
- Green light — `Local:1:O.Data.2`
- Red light — `Local:1:O.Data.0`

Internal tags:
- `system_run` — seal-in bit
- `timer` — TON, preset 5000 ms

### Logic as built
- **Rung 0** seals in `system_run` from Start. The N.C. Stop and E-Stop inputs are examined with XIC, so a broken wire stops the machine.
- **Rung 1** runs the TON while `system_run` is on. The timer resets as soon as its rung goes false.
- **Rung 2** runs the motor and green light while `system_run` is on and `timer.DN` is off, with Stop and E-Stop in series.
- **Rung 3** latches the red light on `timer.DN`, Stop, or E-Stop. Start clears it.

### Debugging
The first version sealed in the motor rung with no stop conditions of its own. Pressing Stop dropped `system_run`, which reset the timer and cleared DN. The XIO DN contact then went true again, and the seal-in held the motor on with nothing to turn it off.

**Lesson:** a TON's DN bit does not remember that the timer finished, so a seal-in needs its own stop conditions. Adding Stop and E-Stop to the motor rung fixed it.

### Testing
Tested on hardware: 5-second timed run, Stop and E-Stop during operation, and the red-light latch.

### Safety note
On production equipment, the E-Stop is hardwired through a safety relay to remove power independently of the PLC (NFPA 79 / ISO 13850). Here it is a PLC input for training purposes.

## Planned Improvements
1. **Automatic re-arm after timeout.** In this version, `system_run` stays on after the timer finishes, so the operator must press Stop and then Start to run again. Adding XIO `red_light` to the `system_run` rung makes the timeout drop `system_run` and reset the timer, so Start alone begins a new cycle, matching the rung 0 comment.
2. **Red light only on a stop during operation.** Pressing Stop while the system is idle also turns on the red light. Simply adding a `system_run` contact to the Stop branch does not work, because the PLC scans rungs top to bottom: the `system_run` rung turns off before the red light rung can check it. The fix is to move the red light rung above the `system_run` rung and put `system_run` in series with the Stop and E-Stop branch. The red light rung then reads `system_run` while it is still on and only latches for a stop during operation.

With both improvements, a timeout or stop latches the red light first, the `system_run` rung below sees it in the same scan and drops out, and Start clears the red light and re-seals `system_run` in one press.

## Tech / Tools
Studio 5000, CompactLogix, RSLinx, ladder logic, TON timer

## Screenshots
*Screenshots coming soon.*

## Status
Complete
