# CompactLogix Garage Door State Machine

## Overview
A garage door controller built as a state machine on an Allen-Bradley CompactLogix PLC, programmed in Studio 5000 and tested on a physical lab station. A single push button commands the door down and up, two TON timers (6000 ms each) model the 6-second travel time in each direction, and pilot lights show the door state: amber = moving down, green = moving up, red = stopped. A proximity sensor reverses the door to the top if it is tripped while the door is closing.

The program was designed from a state transition diagram with four states: **Door Up**, **Moving Down**, **Door Down**, and **Moving Up**.

I/O mapping:
- Green push button — `Local:1:I.Data.2`
- Proximity sensor — `Local:1:I.Data.9`
- Amber light — `Local:1:O.Data.4`
- Green light — `Local:1:O.Data.2`
- Red light — `Local:1:O.Data.0`

Internal tags:
- `button_bit` — single-scan pulse from the button (or a new sensor trip)
- `trigger_bit` — direction toggle (1 = commanded down, 0 = commanded up)
- `state_bit` — door position (1 = door down)
- `sensor_bit` — latched sensor trip
- `one_shot` — ONS storage bit
- `timer1` — TON, down travel, preset 6000 ms
- `timer2` — TON, up travel, preset 6000 ms

### Logic as built
- **Rung 0** latches `sensor_bit` when the sensor trips.
- **Rung 1** uses an ONS so a held button, or a new sensor trip, produces a single-scan pulse on `button_bit`.
- **Rung 2** toggles `trigger_bit` on each pulse (XOR seal-in), so one button gives both directions.
- **Rung 3** runs `timer1` while commanded down, the door is up, and no sensor trip is latched.
- **Rung 4** drives the amber light from `timer1.EN`.
- **Rung 5** latches `state_bit` on `timer1.DN`.
- **Rung 6** runs `timer2` when commanded up while the door is down, or when `sensor_bit` is latched.
- **Rung 7** drives the green light from `timer2.EN`.
- **Rung 8** unlatches `state_bit` and `sensor_bit` on `timer2.DN`.
- **Rung 9** turns on the red light whenever the amber and green lights are both off.

### Exercise I
A single push-button ON/OFF toggle using an ONS, which became the basis of the direction toggle in rungs 1 and 2.

### Debugging
Four issues were found and fixed on hardware:
1. **The door moved on its own at power-up.** Fixed by driving the timers from the push-button toggle.
2. **The amber light stayed on after the door stopped.** A timer's EN bit stays on as long as its rung is true. Fixed by using `state_bit` to open the down-timer rung when the door arrives.
3. **The sensor stopped the door instead of reversing it.** The up timer required `state_bit = 1`, which is never true while the door is still moving down. Fixed with a sensor branch that bypasses `state_bit`.
4. **The door went down by itself after a reversal.** `trigger_bit` was still set to "down." Fixed by feeding `sensor_bit` into the one-shot rung, so a trip also toggles `trigger_bit`.

### Testing
Tested on hardware: down and up travel with the correct lights, sensor reversal during downward travel, and the red light whenever the door is stopped.

## Planned Improvements
1. **Sensor active only while closing.** Rung 0 latches `sensor_bit` in any state, so tripping the sensor while the door is idle up, closed, or moving up also triggers a reversal cycle and toggles `trigger_bit`. For example, a trip while the door is closed opens it, and a trip while idle at the top runs a 6-second up cycle and then closes the door. Adding XIC `timer1.EN` in series with the sensor in rung 0 restricts the reversal to downward travel, matching the state diagram.
2. **Mid-travel stops.** Pressing the button during travel stops the door, but the program still treats it as fully up or fully down, so the next move runs a full 6 seconds from the wrong position. Tracking partial travel, for example by retaining the timer accumulator with an RTO, would model the real door position.

## Tech / Tools
Studio 5000, CompactLogix, RSLinx, ladder logic, ONS one-shot, TON timers, state machine design

## Screenshots
*Screenshots coming soon.*

## Status
Complete
