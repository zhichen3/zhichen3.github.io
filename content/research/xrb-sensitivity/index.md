+++
title = "Sensitivity of He Flames in X-ray Bursts to Nuclear Physics"
author = ["zhi"]
date = 2025-02-02T00:00:00-05:00
type = "gallery"
draft = false
weight = 2002
image = "network_abar_50ms.png"
+++

## Introduction {#introduction}

Millisecond burst oscillation phenomenon is often observed during the rise time
of the X-ray burst light curve, where the oscillation frequency matches with the
X-ray emission pulsation of the neutron star within few Hz.
The modulation of asymmetrical burning on the surface of the neutron star
due to the spreading of the initial local hotspot is the current contender theory
that explains this behavior.
Many have attempted to model the laterally flame propagation on the neutron star surface,
with successes at studying the flame front and calculating the flame speed.
In this project, I conducted a sensitivity test on how choices of nuclear reaction network,
plasma screening routines, and integration coupling methods can influence the He flame dynamics.
Details of the work is published in [ApJ](https://iopscience.iop.org/article/10.3847/1538-4357/acec72).
**Here I'll just summarize the most important finding on the effect of nuclear reaction network
on flame dynamics and nucleosynthesis**.


## Initial Model {#initial-model}

We used [Castro](https://github.com/AMReX-Astro/Castro), a compressible hydrodynamics simulation code freely available on GitHub, to
perform all the simulations. Nuclear reaction burning related modules are provided via [Microphysics](https://github.com/AMReX-Astro/Microphysics).
To set the stage, we assumed a typical 1.4 M<sub>o</sub> neutron star with radius of 11 km.
We used a relative a relatively higher &Omega; = 1000 Hz to have a greater flame confinement due to
Coriolis force so that a smaller simulation domain.
A parallel-plane geometry with 2D axisymmetric R-Z cylindrical coordinate system is used.
We considered a simulation domain of r = 1.843 &times; 10<sup>5</sup> cm and z = 3.072 &times; 10<sup>4</sup> cm,
taking place on the surface of the rotating pole, where Coriolis force is the maximum.
A coarse grid of 1152 &times; 192 zones were used, corresponding to 160 cm resolution.
With 2 extra AMR levels, there are 9216 &times; 1536 zones for the finest grid, corresponding
to a 20 cm resolution.
A constant gravity is z is used since the mass of the accretion layer
is negligible compared to the mass the neutron star.
This corresponds to &sim; 10<sup>&circ;</sup> away from the pole at the maximum extent,
allowing us to work with a constant Coriolis force in the co-rotating frame of the neutron star.
Initially, the fuel layer is assumed to have pure He<sup>4</sup> uniformly distributed horizontally in
an isentropic atmosphere for z &gt; 2000 cm.
An isothermal base layer comprised of pure Ni<sup>56</sup> for z &lt; 2000 cm to represent the transition to
the interior of the neutron star.
Since the model is initially in hydrostatic equilibrium, we placed a temperature perturbation profile of
1.2 &times; 10<sup>9</sup> K at the base of the He<sup>4</sup> layer for r &lt; 4.096 &times; 10<sup>4</sup> cm to facilitate nuclear burning,
compared to T = 2 &times; 10<sup>8</sup> K at the base of the He<sup>4</sup> layer for r &gt; 4.096 &times; 10<sup>4</sup> cm.
Figure [1](#figure--fig:init-temp) shows the initial temperature profile on the left side of the domain.
The total simulation time is 120 ms to prevent flame propagating outside the domain.

<a id="figure--fig:init-temp"></a>

{{< figure src="/ox-hugo/init_temp.png" caption="<span class=\"figure-number\">Figure 1: </span>Slice plot showing the initial temperature pertubation. Note this only shows a small fraction of the domain." width="95%" >}}


## Reaction Network {#reaction-network}

Several reaction networks were used to test the sensitivity of nuclear physics
to flame dynamics. Here we only discuss the one that is found to be most
relevant, _subch_simple_, a network comprised of 22 isotopes and 57 rates.
See Figure [2](#figure--fig:subch-simple) for visualizations.

<a id="figure--fig:subch-simple"></a>

{{< figure src="/ox-hugo/subch_simple.png" caption="<span class=\"figure-number\">Figure 2: </span>A visualization that shows the _subch_simple_ network." width="85%" >}}

The classic 13-isotope &alpha;-chain network from \\({}^{4}\mbox{He}\\) to \\({}^{56}\mbox{Ni}\\) , _aprox13_,
is used as a reference network for comparison. See Figure [3](#figure--fig:aprox13) for visualizations.

<a id="figure--fig:aprox13"></a>

{{< figure src="/ox-hugo/aprox13.png" caption="<span class=\"figure-number\">Figure 3: </span>A visualization that shows the _aprox13_ network." width="85%" >}}

The most important difference between _subch_simple_ and _aprox13_ is inclusion of the rate sequence,
\\({}^{12}\mbox{C}(\mbox{p}, \gamma) {}^{13}\mbox{N}(\alpha, \mbox{p}){}^{16}\mbox{O}\\).
1D studies have shown that this rate sequence dominates over &alpha;-capture process
on \\({}^{12}\mbox{C}\\), \\({}^{12}\mbox{C} (\alpha, \gamma) {}^{16}\mbox{O}\\) for \\(T \gtrsim 10^9\\) K,
which is responsible for generating a burst of energy as temperature increases
during the start of the burst.


## Results {#results}

<a id="figure--fig:abar"></a>

{{< figure src="/ox-hugo/network_abar_50ms_finesst.png" caption="<span class=\"figure-number\">Figure 4: </span>Slice plots showing the mean molecular weight for simulations that used different reaction network at 50 ms simulation time. A larger coverage and deeper color of the mean molecular weight for _subch_simple_ (bottom panel) indicates a much more vigorous burning process compared to _aprox13_ (top panel)." width="75%" >}}

Our 2D simulations show a general agreement with these 1D studies.
Figure [4](#figure--fig:abar) shows the mean molecular weight, \\(\bar{A}\\), of the flame at 50 ms
using the two networks. Regions with a larger \\(\bar{A}\\) represent the ashes from nuclear burning.
Compared to _aprox13_, _subch_simple_ shows a larger coverage of ash structure,
both vertically and horizontally, indicating much more vigorous burning and a faster flame speed.
A darker color indicate ashes are composed of heavier nuclei suggesting much more frequent
late-stage burning processes.

<a id="figure--fig:profile"></a>

{{< figure src="/ox-hugo/network_time_profile_finesst.png" caption="<span class=\"figure-number\">Figure 5: </span>Time evolution of density weighted temperature and energy generation rate of the flame. _subch_simple_ (red) shows spikes in energy generation rate (right panel) initially and at t ~ 20 ms compared to a steady increase in _aprox13_ (blue). This corresponds to the steeper increase in temperature (left panel) for t &lt; 25 ms for _subch_simple_." width="85%" >}}

Figure [5](#figure--fig:profile) shows the evolution of density-weighted temperature and
\\(\dot{e}\_{\text{nuc}}\\) of the flame. Instead of a steady increase in both temperature
and \\(\dot{e}\_{\text{nuc}}\\) in _aprox13_, _subch_simple_ shows burst of energies
at \\(\sim 20\\) ms and a quick fall off afterwards.

<a id="figure--fig:species"></a>

{{< figure src="/ox-hugo/network_species_summary_log_finesst.png" caption="<span class=\"figure-number\">Figure 6: </span>Time evolution of the total mass for C12, O16, and Si32. A depletion of C12 is observed at ~ 20 ms for _subch_simple_ (red) compared to _aprox13_ (blue), indicating a much more efficient burning for C12 is available in _subch_simple_. This leads to nucleosynthesis of heavier isotopes like Si32." width="90%" >}}

To understand the behavior of this evolution trajectory,
Figure [6](#figure--fig:species) shows the total mass evolution of \\({}^{12}\mbox{C}\\), \\({}^{16}\mbox{O}\\), and \\({}^{32}\mbox{Si}\\).
In contrast to the continuous buildup of \\({}^{12}\mbox{C}\\)  in _aprox13_
since the network is bottle-necked by \\({}^{12}\mbox{C} (\alpha, \gamma) {}^{16}\mbox{O}\\),
\\({}^{12}\mbox{C}(\mbox{p}, \gamma) {}^{13}\mbox{N}(\alpha, \mbox{p}){}^{16}\mbox{O}\\)
opens up a new path way for fusing \\({}^{16}\mbox{O}\\) in _subch_simple_ at \\(t \sim 20\\) ms with a
corresponding \\(T \sim 1.3 \times 10^9\\) K. At this point, nuclear burning timescale for
\\({}^{12}\mbox{C}(\mbox{p}, \gamma) {}^{13}\mbox{N}(\alpha, \mbox{p}){}^{16}\mbox{O}\\) is faster than
the rate at which \\({}^{12}\mbox{C}\\) is produced by the triple-\\(\alpha\\) process.
This leads to a depletion of \\({}^{12}\mbox{C}\\), corresponding to the burst of energy observed in
Figure [5](#figure--fig:profile) at \\(t \sim 20\\) ms, as well as an early fuel exhaustion compared to _aprox13_.

<a id="figure--fig:front"></a>

{{< figure src="/ox-hugo/network_front_finesst.png" caption="<span class=\"figure-number\">Figure 7: </span>Time evolution of the flame front position. An initial acceleration phase is observed for _subch\\_simple_ (red) in contrast to a global uniform flame propagation in _aprox13_ (blue)." width="40%" >}}

In terms of flame speed, Figure [7](#figure--fig:front) shows as initial short acceleration phase
for _subch_simple_ following by an uniform speed of \\(\sim 1.5 \ \text{km} \ \text{s}^{-1}\\) similar to _aprox13_.
By extrapolating beyond the data, calculations show both models takes \\(\sim 1.5\\) s
to reach 30 km, roughly the distance flame needs to travel to engulf the entire star.
This matches with the rise time of the light curve as we discussed previously.
This study gives us the confidence that _subch_simple_ is the optimal network to use
for the future full-star simulation, where we determine the time for the flame to
reach maximum coverage of the star along with the influence of Coriolis force modulation
without extrapolation.


## Summary {#summary}

<video width="1000" height="600" controls><source src="/videos/xrb-sensitivity.mp4" type="video/mp4">
Your browser does not support the video tag.
Videos only work in static folders.</video>

All the results shown proves that \\({}^{12}\mbox{C}(\mbox{p}, \gamma) {}^{13}\mbox{N}(\alpha, \mbox{p}){}^{16}\mbox{O}\\)
is critical for an accurate modeling of the laterally propagating He flames in X-ray bursts
because it changes both nucleosynthesis and flame dynamics drastically.
Lastly, we provide a movie showing flame propagation. Three different panels
showing temperature, \\(\bar{A}\\), and \\(\dot{e}\_\text{nuc}\\), from top to bottom.

**Please see the complete study in [ApJ](https://iopscience.iop.org/article/10.3847/1538-4357/acec72) for more detail.**
