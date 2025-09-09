On reset the design initializes: current_floor=0, door_open=1, moving=0, state=IDLE, next_floor=0.

In IDLE state door stays open, moving=0, if overweight=1 then requests are ignored.

In IDLE if overweight=0 and req≠0 then next_floor is assigned from priority encoder, but because of non-blocking assignment, the if check compares old next_floor. This causes one cycle delay before deciding to close doors.

When the condition is finally true, IDLE schedules door_open=0 and state=CLOSE_DOOR, so on the next clock the door closes and FSM goes to CLOSE_DOOR.

In CLOSE_DOOR state FSM sets door_open=0, moving=1, state=MOVE. On the following clock moving becomes 1 and the elevator starts moving.

In MOVE state if current_floor==next_floor FSM schedules moving=0, door_open=1, state=IDLE; if not equal, FSM increments or decrements current_floor by one depending on whether destination is above or below.

Testbench has generator, driver, monitor, scoreboard, environment.

Generator produces transactions with req and overweight values and puts them into a mailbox.

Driver takes each transaction from mailbox and applies signals to the DUT via interface. Driver logs when it drives new values.

Monitor samples interface signals every #10 units and displays current_floor, door_open, moving, req, overweight. Monitor also packs these into 4 bits {current_floor, door_open, moving} and sends to scoreboard.

Scoreboard receives packed values from monitor and prints them as Observed=xxxx for comparison.

Environment instantiates generator, driver, monitor, scoreboard, links their mailboxes, and runs them in parallel.

Top module instantiates DUT and interface, applies clock and reset, instantiates environment, runs run() task.

In waveform at time 0 driver sets Req=1000 Overweight=1. FSM in IDLE with overweight=1 ignores request. Monitor shows Floor=0 Door=1 Moving=0, scoreboard 0010.

At time 20 overweight=0. On posedge at t=25 FSM latches next_floor=3 but compares old 0, so still idle. One cycle later at t=35 FSM sees next_floor=3 vs current_floor=0, closes door, goes to CLOSE_DOOR. Monitor shows first door=1 then door=0.

At t=45 FSM in CLOSE_DOOR sets moving=1, state=MOVE. Elevator starts moving, monitor shows Floor=0 Door=0 Moving=1.

At t=55 FSM increments floor to 1, monitor shows Floor=1 Door=0 Moving=1.

At t=65 FSM increments to floor 2, monitor shows Floor=2 Door=0 Moving=1.

At t=75 FSM increments to floor 3, monitor shows Floor=3 Door=0 Moving=1.

At t=85 FSM detects current_floor==next_floor, stops moving, opens door, state=IDLE. Monitor shows Floor=3 Door=1 Moving=0. Monitor repeats this as stable.

At time 140 driver sets Req=0010. At t=145 FSM latches next_floor=1 but compares old 3 vs 3, so idle one cycle. At t=155 FSM sees 1≠3, closes door, goes to CLOSE_DOOR. Monitor shows Floor=3 Door=1 then Floor=3 Door=0.

At t=165 FSM sets moving=1, state=MOVE. At t=175 FSM decrements floor to 2, monitor shows Floor=2 Door=0 Moving=1. At t=185 decrements to floor 1, monitor shows Floor=1 Door=0 Moving=1.

At t=195 FSM sees current_floor==next_floor, stops moving, opens door, state=IDLE. Monitor shows Floor=1 Door=1 Moving=0.

Req cleared at times 40 and 160 did not cancel destination because next_floor had already been latched.

Monitor shows repeated identical lines because it samples every 10 units, not exactly at posedges.

Overall flow: driver applies inputs, FSM updates on clock edges, monitor samples signals every 10 units, scoreboard prints encoded state.

Key behaviors: FSM ignores requests when overweight=1, has one cycle delay in responding to new request, moves one floor per clock in MOVE state, and doors reopen at destination.
