+++
title = "Adaptive Nuclear Statistical Equilibrium for Type Ia SN Simulation"
author = ["zhi"]
date = 2025-02-20T00:00:00-05:00
type = "gallery"
draft = false
weight = 2002
image = "network_abar_50ms.png"
+++

## Introduction {#introduction}

Due to the stiff nature of solving reaction equations for simulating
thermonuclear explosions, i.e. Type Ia SN and X-ray bursts, these simulations
require careful coupling of hydrodynamics and reactions to avoid being limited
by the nuclear reaction timescale instead of hydrodynamic timescale.
But as temperature reaches beyond ~ 6 billion Kelvin, numerical integration
becomes even more challenging when the timescale of the composition to reach
statistical equilibrium is much shorter than the sound crossing time.
Inspired by earlier [work in the community](https://academic.oup.com/mnras/article/493/4/5413/5766330?login=false) at adaptively evaluating the
nuclear statistical equilibrium (NSE) conditions, we have implemented an
adaptive NSE method in our simplified spectral-deferred-corrections (SDC)
coupling of hydrodynamics and reactions. This can seamlessly transition from
integrating a network to imposing the equilibrium during the reaction update of a simulation.

In this work, we demonstrate the recent effort of implementing an adaptive NSE algorithm
in Castro/Microphysics code. We will show case its viability using a 2D sub-chandrasekhar
double-detonation progenitor model for type Ia SN.


## Nuclear Statistical Equilibrium {#nuclear-statistical-equilibrium}

Nuclear Statistical Equilibrium (NSE) is an equilibrium state of the system where the
forward and reverse strong reactions are in chemical equilibrium. By imposing chemical equilibrium,
one can derive the equation that determines the composition in NSE:

\\[ X\_i = \frac{m\_i}{\rho}g\_i \left(\frac{2\pi m\_i k\_B T}{h^2}\right)^{3/2} \exp{\left(\frac{Z\_i \mu\_p + N\_i \mu\_n + Q\_i - u^C\_i}{k\_B T}\right)} \\]

Given (T, &rho;, Y<sub>e</sub>), then the NSE equation can be solved with two constraints:

1.  Conservation of mass: \\(\sum\_i X\_i = 1\\)
2.  Conservation of charge: \\(Y\_e = \sum\_i \frac{Z\_i X\_i}{A\_i}\\)

Therefore, we can do an algebraic solve, e.g. Hybrid-Powell method or Newton-Raphson method,
to avoid the expensive integration.


## Network Prerequisite {#network-prerequisite}

Some basic prerequisite for the reaction network include:

-   Contains a least proton and helium-4
-   Ideally the mass fraction in equilibrium obtained from integration
    should match with the mass fraction obtained from NSE equation
-   Screening is compatible with NSE, e.g. Chabrier &amp; Potekhin 1998 screening.


## NSE Conditions {#nse-conditions}

Before using the NSE equation, we must first determine whether the current
simulation cell is in NSE. An overview of the NSE evolution schematic diagram
is shown in [1](#figure--fig:diagram)

<a id="figure--fig:diagram"></a>

{{< figure src="/ox-hugo/nse-schematic-diagram.png" caption="<span class=\"figure-number\">Figure 1: </span>NSE evolution schematic diagram" width="75%" >}}

These checks include:

-   Min Temperature Check: temperature of the cell must exceed a certain value,
    by default this is 4 &times; 10<sup>9</sup>
-   Mass Abundance Check: compare the current mass abuandances to the NSE mass fractions.

    If there are proton, neutron, and helium-4 are present in the network,
    then define r = Y<sub>&alpha;</sub>/(Y<sub>p</sub><sup>2</sup>Y<sub>n</sub><sup>2</sup>) and r<sub>nse</sub> = (Y<sub>&alpha;</sub>/(Y<sub>p</sub><sup>2</sup>Y<sub>n</sub><sup>2</sup>))<sub>nse</sub>, and require

    \\[ \frac{r - r\_{nse}}{r\_{nse}} < 0.5\\]

    If there are only proton and helium-4, then define
    r = Y<sub>&alpha;</sub>/Y<sub>p</sub><sup>2</sup> and r<sub>nse</sub> = (Y<sub>&alpha;</sub>/(Y<sub>p</sub><sup>2</sup>))<sub>nse</sub> and require:

    \\[ \frac{r - r\_{nse}}{r\_{nse}} < 0.25\\]

    If the check above failed, then we proceed to an overall molar fraction check:

    \\[ \epsilon\_{abs} = \sum\_{i} |Y^{i} - Y^{i}\_{nse}| < \epsilon\_{abs}^{nse} \\]

    and

    \\[ \epsilon\_{rel} = \sum\_{i} \frac{\epsilon\_{abs} }{ Y^{i} }< \epsilon\_{rel}^{nse} \\]

-   NSE Grouping: Perform a grouping process to all the nuclei. Based on the final
    grouping configuration, we determine whether the network is currently in NSE or not.


## NSE Burn {#nse-burn}

Once the cell is determined to be in NSE, mass fraction is determined by the NSE equation.
However, a careful calculation is needed for finding the energy generation.


## Application: Double-Detonation {#application-double-detonation}

<video width="1000" height="600" controls><source src="/videos/subchandra.mp4" type="video/mp4">
Your browser does not support the video tag.
Videos only work in static folders.</video>
