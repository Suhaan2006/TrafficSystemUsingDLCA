# TrafficSystemUsingDLCA
Sure! Here's a thorough theory writeup followed by the truth table.

---

## Theory: Two-Road Traffic Light Controller

### What is it?

A traffic light controller is a **sequential logic circuit** — meaning its output depends not just on current inputs, but also on its *stored state* (which road is currently green). This is what makes it different from a purely combinational circuit.

---

### System overview

The circuit controls two roads, Road A and Road B, which are always in **complementary states** — when A is green, B must be red, and vice versa. A single state bit `Q` (stored in the D flip-flop) represents this: `Q=1` means Road A is green, `Q=0` means Road B is green.

---

### Inputs

| Signal | Name | Meaning |
|--------|------|---------|
| `S` | Sensor | 1 = vehicle detected waiting on Road A |
| `T` | Timer | 1 = current green phase has fully elapsed |
| `E` | Emergency | 1 = override, force Road A green immediately |

---

### Gates and their roles

**G1 — AND gate** computes `S · T`. This ensures a phase switch only happens when *both* a vehicle is present *and* the timer has expired. If only one condition is true, the phase doesn't switch — this prevents premature switching and indefinite green phases.

**G2 — OR gate** computes `(S · T) + E`, producing the final `SWITCH` signal. The emergency input `E` bypasses the AND condition entirely — any emergency immediately triggers a switch regardless of the timer or sensor.

**FF1 — D Flip-Flop** is the memory element. It stores the current phase as bit `Q`. Every time `SWITCH` pulses HIGH (on the clock edge), Q toggles because Q' is fed back into D — making it behave as a **T flip-flop** (toggle flip-flop). This is the key sequential element of the circuit.

**G3 — NOT gate** inverts Q to produce `RA` (Road A Red). When Road A is green (Q=1), RA must be 0, and when A is red (Q=0), RA must be 1.

**G4 — NOT gate** inverts Q to produce `GB` (Road B Green). This is the same inversion as RA — in hardware, G3 and G4 can actually share a single NOT gate output since `RA = GB = Q'`.

---

### Output equations

```
GA  =  Q               (Road A Green)
RA  =  Q'  =  NOT Q    (Road A Red)
GB  =  Q'  =  NOT Q    (Road B Green)
RB  =  Q               (Road B Red)

SWITCH  =  (S · T) + E
```

The complementary nature means only **2 NOT gates** are needed for 4 outputs. GA and RB are the same wire; RA and GB are the same wire.

---

### Why a D flip-flop and not an SR latch?

An SR latch has a forbidden state (S=R=1). A D flip-flop is cleaner — it has a single data input and changes state only on a clock edge, making it more predictable and noise-resistant. By feeding Q' back into D, we turn it into a toggle flip-flop, which is exactly the behaviour needed here.

---

Now here's the full truth table:A few things worth noting about the truth table:

The green-highlighted rows (★) are the normal switching events — both S and T must be 1 simultaneously for SWITCH to fire. This is the AND gate's job: it prevents a switch if the timer hasn't expired yet (rows 3–4) or if no vehicle is waiting (rows 5–6).

The amber-highlighted rows show the emergency override — E=1 forces SWITCH=1 regardless of S and T. Notice row 10: if A is already green when the emergency fires, it actually toggles *away* from A. In a real system you'd want to add extra logic to handle this (e.g. only trigger emergency if Q=0), but this table shows the raw circuit behaviour as designed.

The `Q(next) = Q XOR SWITCH` relationship at the bottom is the compact way to express the toggle flip-flop behaviour — it holds for every row in the table.
