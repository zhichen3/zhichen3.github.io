+++
title = "Adaptive Nuclear Statistical Equilibrium for Type Ia SN Simulation"
author = ["zhi"]
date = 2025-02-20T00:00:00-05:00
type = "gallery"
draft = false
weight = 2002
image = "enuc_slice.png"
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
one can derive the equation that determines the composition in NSE, i.e. Eq. \ref{eq:nse},

\begin{equation}
\label{eq:nse} \tag{1}
X\_i = \frac{m\_i}{\rho}g\_i \left(\frac{2\pi m\_i k\_B T}{h^2}\right)^{3/2} \exp{\left(\frac{Z\_i \mu\_p + N\_i \mu\_n + Q\_i - u^C\_i}{k\_B T}\right)}
\end{equation}

where _m_i_ is the mass, &rho; is the density, _g_i_ is the temperature-dependent partition function,
_T_ is temperature, _Z_i_ and _N_i_ are the proton and neutron number, _mu_p_ and _&mu;_n_ are the chemical
potential of free proton and neutron, _Q_i_ is the binding energy and _&mu;^C_ is coulomb contribution.
Given (T, &rho;, Y<sub>e</sub>), then Eq. \ref{eq:nse} can be solved with two constraints:

1.  Conservation of mass: \\(\sum\_i X\_i = 1\\)
2.  Conservation of charge: \\(Y\_e = \sum\_i \frac{Z\_i X\_i}{A\_i}\\)

Therefore, we can do an algebraic solve, e.g. Hybrid-Powell method or Newton-Raphson method,
to avoid the expensive integration.


## Network Prerequisite {#network-prerequisite}

Some basic prerequisite for the reaction network include:

-   Contains a least proton and Helium-4.
-   Mass fractions in equilibrium obtained from integration
    should match with the mass fraction obtained from NSE equations.
-   Screening is compatible with NSE, e.g. Chabrier &amp; Potekhin 1998 screening.


## NSE Conditions {#nse-conditions}

An overview of the NSE evolution schematic diagram is shown in Figure [1](#figure--fig:diagram)

<a id="figure--fig:diagram"></a>

{{< figure src="/ox-hugo/nse-schematic-diagram.png" caption="<span class=\"figure-number\">Figure 1: </span>NSE evolution schematic diagram" width="75%" >}}

Before using the NSE equation, a list of checks shown in Figure [1](#figure--fig:diagram)
are conducted to ensure the current simulation cell is in NSE.


### Minimum Temperature Check {#minimum-temperature-check}

Require

\\[ T > T\_{min} \\]

Temperature of the cell must exceed a certain value, by default T<sub>min</sub> = 4 &times; 10<sup>9</sup> K.
Under this temperature, NSE is not expected to occur at all.


### Mass Abundance Check {#mass-abundance-check}

The current mass abundances should be close to the NSE mass fractions.
Note, we currently do not introduce any methods of handling the _quasi-NSE_ state
when transitioning from a non-NSE state to NSEstate. Therefore, an abrupt change
in the energy is expected. To avoid this

If there are proton, neutron, and Helium-4 are present in the network,
then define r = Y<sub>&alpha;</sub>/(Y<sub>p</sub><sup>2</sup>Y<sub>n</sub><sup>2</sup>) and r<sub>nse</sub> = (Y<sub>&alpha;</sub>/(Y<sub>p</sub><sup>2</sup>Y<sub>n</sub><sup>2</sup>))<sub>nse</sub>, and require

\\[ \frac{r - r\_{nse}}{r\_{nse}} < 0.5 \\]

If there are only proton and helium-4, then define
r = Y<sub>&alpha;</sub>/Y<sub>p</sub><sup>2</sup> and r<sub>nse</sub> = (Y<sub>&alpha;</sub>/(Y<sub>p</sub><sup>2</sup>))<sub>nse</sub> and require:

\\[ \frac{r - r\_{nse}}{r\_{nse}} < 0.25 \\]

If the check above failed, then we proceed to an overall molar fraction check:

\\[ \epsilon\_{abs} = \sum\_{i} |Y^{i} - Y^{i}\_{nse}| < \epsilon\_{abs}^{nse} \\]

and

\\[ \epsilon\_{rel} = \sum\_{i} \frac{\epsilon\_{abs} }{ Y^{i} }< \epsilon\_{rel}^{nse} \\]


### NSE Grouping Procedure {#nse-grouping-procedure}

A grouping process is performed to all the nuclei based
on the available reaction rates.
Based on the final grouping configuration,
we determine whether the network is currently in NSE or not.

1.  Determine the reaction timescale for the available reaction rates,
    which is defined as

    \\[ t\_{i,k} = \frac{Y\_i}{min(b\_{f}(k), b\_{r}(k)) \\]

    where \\(Y\_i\\) is the molar fraction of the _i-th_ isotope in the reaction
    that is different from _p_, _n_, and _&alpha;_. Note that due to the constraints
    we have below, there can be at most two of these nuclei in a given
    reaction. The smaller \\(Y\_i\\) is chosen.

    Consider the _k-th_ reaction of the following form:

    \\[ A + B \rightleftarrows C + D \\]

    _b<sub>f</sub>(k)_ and _b<sub>r</sub>(k)_ are the forward and reverse reaction rate
     of the _k-th_ reaction, which is defined as following:

    \\[ b\_{f,r}(k) = (1 + \sigma\_{AB,CD}) \rho Y\_{A,C} Y\_{B,D} \frac{N\_A <\sigma v>\_{f,r}}{1 + \sigma\_{AB,CD}} \\]

    or if only single reactant or product is involved then:

    \\[ b\_{f,r}(k) = |Y\_{A,C} N\_A <\sigma v>\_{f,r}| \\]

    Note, some reactions are skipped during the process,
    and the timescale for these reactions are set to be largest or slowest:

    1.  Reactions that have no reverse rates.

    2.  Reactions involve more than three reactants or products involved.

    3.  Reactions involve more than 2 non- _n_, _p_, and _&alpha;_ in reactants
        and products.

    All reaction timescales are initialized with a maximum (slowest) machine
    number. If all the criteria are satisfied above, then the reaction
    timescale is computed.

    Lastly, we require the forward and reverse rates are close to each other:

    \\[ \frac{2.0 |b\_f(k) - b\_r(k)|}{b\_f(k) + b\_r(k)} < \epsilon  \\]

    and the reaction timescale must be faster compared to the
    sound-crossing timescale:

    \\[ t\_{i,k} = \epsilon t\_s \\]

    where the sound crossing time, \\(t\_s\\) is defined as:

    \\[ t\_s = \frac{min(dx[0], dx[1], dx[2])}{c\_s} \\]

    where dx is the size of the simulation cell and \\(c\_s\\) is the sound speed,
    and &epsilon; is chosen by the user, which is typically &sim; 0.1 or 0.01.

    Reaction rates with all these requirements satisfied will carry out
    reaction timescales that will be used later for the grouping process.

2.  To start the grouping process, all nuclei except _p_, _n_, and _&alpha;_
    initially form a group on their own.
    _p_, _n_, and _&alpha;_ form a single group, called the light-isotope-group or LIG.

3.  Grouping process starts from the fastest reaction timescale.
    We have already filtered out reaction rates that don't satisfy the NSE
    conditions during the calculation of the reaction timescale.

    Here consider two cases during the grouping process:

    1.  If there are exactly two isotopes involved in the _k-th_ reaction that are
        not in LIG, then merge the isotope in the smaller group into the
        isotop in the larger group.

        **Note, in this case, we skip this reaction if both isotopes are**
        **already in the same group. (Perhaps we can still merge the nonLIG group**
        **to LIG at this point)???**

    2.  If there is only 1 isotope involved in the _k-th_ reaction that is not
        in LIG, then merge that isotope and the group that it's in into LIG

4.  A final grouping configuration is obtained after the grouping process.
    1.  If the network has neutron, then define NSE if all the nuclei are in
        the single group with an optional LIG.

    2.  If the network does not have neutron, then consider a looser constraint
        where for isotopes  \\(\Zstroke \geq 14\\), isotopes with odd N and even N
        form two distinct groups.


## NSE Burn {#nse-burn}

Once the cell is determined to be in NSE, mass fraction is determined by the NSE equation.
However, a careful calculation is needed to determine the energy generation rates.


## Application: Double-Detonation {#application-double-detonation}

<video width="1000" height="600" controls><source src="/videos/subchandra.mp4" type="video/mp4">
Your browser does not support the video tag.
Videos only work in static folders.</video>
