## 🛗 Elevator Controller in VHDL (FPGA)

A 7-floor (0–6) elevator controller written in **VHDL** as a Moore finite state machine. It was designed and simulated in **Xilinx Vivado** as a team project (3 students) for the *Digital Systems Design* course at the University of the Peloponnese (May 2025).

### Features

- Internal cabin buttons (floor select) plus **Up** and **Down** call buttons on every floor
- Pending-request registers, so several requests can be queued at once
- Direction-aware service: keep going in the current direction, stop for matching requests, reverse when nothing is left ahead
- Door control with a timed open/close cycle, and the elevator does not move while the door is open
- Outputs for current floor, moving up, moving down and door open
- Synchronous `reset` returns to floor 0, door closed, no pending requests

### Interface

| Signal | Dir | Width | Description |
|--------|-----|-------|-------------|
| `clk` | in | 1 | System clock |
| `reset` | in | 1 | Reset to the initial state |
| `internal_req` | in | 3 | Floor chosen inside the cabin (0–6) |
| `internal_valid` | in | 1 | Marks `internal_req` as a valid request |
| `ext_up_req` | in | 7 | One bit per floor: "going up" call |
| `ext_down_req` | in | 7 | One bit per floor: "going down" call |
| `current_floor` | out | 3 | Current floor |
| `moving_up` / `moving_down` | out | 1 | Direction indicators |
| `door_open` | out | 1 | `'1'` while the door is open |

### State machine

`IDLE` → `MOVING_UP_STATE` / `MOVING_DOWN_STATE` → `DOOR_OPENING` → `DOOR_CLOSING` → `IDLE`

The door timer is a constant (`DOOR_TIMER_MAX = 10` clock cycles), and the outputs depend only on the current state, as a Moore machine requires.

### Testbench

`ele3_tb.vhd` drives five scenarios and prints the floor, state and door status to the console after each one:

1. Internal request for floor 3
2. External "up" call from floor 1
3. External "down" call from floor 5
4. Multiple simultaneous requests from different floors
5. Direction change with "down" requests from floors 6 and 4

### Run it

**Vivado:** create a project with `ele3.vhd` as the design source and `ele3_tb.vhd` as the simulation source. Set `elevator_controller_tb` as the top module and run *Behavioral Simulation*.

**GHDL (open source):**

```bash
ghdl -a --std=08 ele3.vhd ele3_tb.vhd
ghdl -e --std=08 elevator_controller_tb
ghdl -r --std=08 elevator_controller_tb --stop-time=200us
```

### Known issues

- The elevator overshoots by one floor. In Test 1 it is asked for floor 3 and stops at floor 4.
- Test 2 stops with an out-of-range error. The floor counter is a 3-bit value and can reach 7, but the request vectors only cover floors 0–6.
- Requests are cleared once served and there is no priority logic between up and down calls.

Ideas for improvement: bounds-check the floor counter, compare the next floor rather than the current one when deciding to stop, and add floor sensors to model the real position of the cabin and door.

### Repository contents

- `ele3.vhd`: elevator controller
- `ele3_tb.vhd`: testbench
- `docs/`: project report (Greek) and presentation
