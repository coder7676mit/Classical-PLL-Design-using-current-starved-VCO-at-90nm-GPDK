# Loop Filter

Visible in the top-level schematic ([image](../images/01_pll_top_level.png)), between
the charge-pump output (`Vo`) and the VCO control input (`Vc`) — passive, no op-amp.

## Topology

Standard passive lag-lead (3rd-order loop overall once combined with the charge pump's
current integration and the VCO's inherent phase integration):

- **R2 = 8.5 kΩ** in series with **C2 = 10 pF**, both returned to ground — this
  series R-C branch forms the stabilizing zero.
- **C1 = 1 pF** in parallel with the R2–C2 branch (directly from `Vc` to ground) —
  a smaching/ripple-suppression cap, forming a third high-frequency pole.

## Derivation

**Zero** (from the R2–C2 series branch):

$$ f_z = \frac{1}{2\pi R_2 C_2} = \frac{1}{2\pi \times 8.5\text{k} \times 10\text{p}}
\approx 1.87\,\text{MHz} $$

**Third pole** (from R2 with the much smaller C1, valid approximation since
C1 ≪ C2):

$$ f_{p3} \approx \frac{1}{2\pi R_2 C_1} = \frac{1}{2\pi \times 8.5\text{k} \times 1\text{p}}
\approx 18.7\,\text{MHz} $$

$$ \frac{f_{p3}}{f_z} = \frac{C_2}{C_1} = 10 $$

A roughly 10× zero-to-third-pole spacing is a standard rule of thumb for this filter
topology — enough separation to get real phase margin benefit from the zero before the
extra pole starts eating it back, while still giving meaningful spur attenuation at the
reference frequency (50 MHz here, which sits comfortably above `f_p3`, so reference
spurs from the charge-pump switching ripple should be reasonably suppressed by the time
they reach `Vc`).

## Observed behavior

The transient plot ([image](../images/07_transient_lock_waveform.png)) shows `Vc`
(green) doing exactly what this filter predicts: a fast initial ramp during acquisition,
then a decaying sawtooth ripple (sharp corrective steps from each charge-pump pulse,
smoothed by C1/C2) settling toward the DC lock value (≈ 690 mV) as the up/dn pulse
widths shrink to their steady-state minimum.

## For the repo / next step

If loop bandwidth and phase margin haven't been computed yet, the natural next step
with these numbers is:

$$ \omega_n = \sqrt{\frac{I_{CP} \cdot K_{VCO}}{2\pi N C_2}}, \qquad
\zeta = \frac{\omega_n R_2 C_2}{2} $$

using `I_CP ≈ 100 µA` ([charge pump doc](03_charge_pump.md)), `K_VCO` from the
VCO sweep ([VCO doc](05_vco_cs.md) — note it's nonlinear, so pick the value local
to the actual lock point, ≈ 690 mV), and `N = 48`. Worth doing before claiming a
specific phase margin number for the report, since `K_VCO` varies by nearly 2× across
the segments measured.
