# Top-Level Architecture

![Top level](../images/01_pll_top_level.png)

## Topology

Classic third-order (type-II) charge-pump PLL:

```
vref ──► PFD ──up/dn──► charge pump ──► loop filter (R2,C2,C1) ──Vc──► VCO_CS ──Vout──►
 ▲                                                                                 │
 │                                                                     inverter buffer
 │                                                                                 │
 └──────────────────────── Divider_48 (÷48, TSPC) ◄───────────────────── div_in ◄──┘
```

- **vref**: external reference, `vpulse` with 20 ns period → F_ref = 50 MHz.
- **vdiv**: feedback from the ÷48 divider's `div_out`, fed back to the PFD's second input.
- **up / dn**: PFD phase/frequency-error pulses.
- **Vo / Vc**: charge-pump output / loop-filter control voltage (same node, renamed at
  the junction in the schematic).
- **Vout**: VCO output, buffered through a single CMOS inverter before entering the
  divider's `div_in`.

Two independent 1.2 V sources are present: `V1`/`V2` feed the PFD reference path
(labeled `v2=1.2`, `v1=0` style pulse source), and a separate `vdc=1.2` directly biases
the VCO/test side — worth double-checking both are tied to the same physical VDD rail
in the actual tapeout-intent version, since separate sources in a testbench can mask a
shared-supply IR-drop effect that would show up on silicon.

## Loop Equation / Lock Validation

$$ F_{out} = N \times F_{ref} = 48 \times 50\,\text{MHz} = 2.4\,\text{GHz} $$

This is confirmed directly by the transient simulation
([waveform](../images/07_transient_lock_waveform.png)): `Vout` toggles far faster than
`vref`/`vdiv` (visually a dense solid block at this timescale, consistent with a ~417 ps
period over a 1 µs window — roughly 2,400 cycles, too dense to resolve individually),
while `vref` and `vdiv` are both clearly resolved as the expected ~50 MHz-rate pulses in
the same window. The `up`/`dn` pulses are wide during acquisition and collapse to
near-zero width once locked — expected PFD behavior at lock.

`Vc` (green trace) settles to ≈ 690 mV with a decaying ripple — see
[loop filter doc](04_loop_filter.md) for the ripple-suppression analysis and
[VCO doc](05_vco_cs.md) for why this differs from the standalone VCO's ≈ 545–550 mV at
the same output frequency.

## Blocks

| Block | Function | Detail |
|---|---|---|
| PFD | Phase/frequency detector, DFF+AND reset | [02_pfd.md](02_pfd.md) |
| Charge pump | Converts up/dn into ±I_CP into the loop filter | [03_charge_pump.md](03_charge_pump.md) |
| Loop filter | Passive lag-lead (R2, C2, C1) | [04_loop_filter.md](04_loop_filter.md) |
| VCO_CS | Current-starved ring oscillator | [05_vco_cs.md](05_vco_cs.md) |
| Inverter | VCO→divider buffer | [06_inverter_buffer.md](06_inverter_buffer.md) |
| Divider_48 | ÷48 feedback divider, TSPC-based | [07_divider_48.md](07_divider_48.md) |
