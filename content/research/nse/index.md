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
forward and reverse strong reactions are in chemical equilibrium.
By imposing chemical equilibrium,

\begin{equation}
\label{eq:nse\_cond} \tag{1}
\mu\_i = Z\_i \mu\_p + N\_i \mu\_n
\end{equation}

one can derive the equation that determines the composition in NSE, i.e. Eq. \ref{eq:nse},

\begin{equation}
\label{eq:nse} \tag{2}
X\_i = \frac{m\_i}{\rho}g\_i \left(\frac{2\pi m\_i k\_B T}{h^2}\right)^{3/2} \exp{\left(\frac{Z\_i \mu\_p + N\_i \mu\_n + B\_i - u^C\_i}{k\_B T}\right)}
\end{equation}

where _m_i_ is the mass, &rho; is the density, _g_i_ is the temperature-dependent partition function,
_T_ is temperature, _Z_i_ and _N_i_ are the proton and neutron number, _mu_p_ and _&mu;_n_ are the chemical
potential of free proton and neutron, _B_i_ is the binding energy and _&mu;^C_ is coulomb contribution.
Given (&rho;, T, Y<sub>e</sub>), then Eq. \ref{eq:nse} can be solved with two constraints:

1.  Conservation of mass: \\(\sum\_i X\_i = 1\\)
2.  Conservation of charge: \\(Y\_e = \sum\_i \frac{Z\_i X\_i}{A\_i}\\)

Therefore, we can do an algebraic solve, e.g. Hybrid-Powell method or
Newton-Raphson method, to avoid the expensive integration.


## Network Prerequisite {#network-prerequisite}

Some basic prerequisite for the reaction network include:

1.  Contains a least proton and Helium-4.
2.  Mass fractions in equilibrium obtained from integration
    should match with the mass fraction obtained from NSE equations.
3.  Screening is compatible with NSE, e.g. Chabrier &amp; Potekhin 1998 screening.


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
when transitioning from a non-NSE state to NSE state. Therefore, an abrupt change
in the energy is expected. To avoid this

If there are proton, neutron, and Helium-4 are present in the network,
then define r = Y<sub>&alpha;</sub>/(Y<sub>p</sub><sup>2</sup>Y<sub>n</sub><sup>2</sup>) and r<sub>\mathrm{nse}</sub> = (Y<sub>&alpha;</sub>/(Y<sub>p</sub><sup>2</sup>Y<sub>n</sub><sup>2</sup>))<sub>\mathrm{nse}</sub>, and require

\\[ \frac{r - r\_{\mathrm{nse}}}{r\_{\mathrm{nse}}} < 0.5 \\]

If there are only proton and helium-4, then define
r = Y<sub>&alpha;</sub>/Y<sub>p</sub><sup>2</sup> and r<sub>\mathrm{nse}</sub> = (Y<sub>&alpha;</sub>/(Y<sub>p</sub><sup>2</sup>))<sub>\mathrm{nse}</sub> and require:

\\[ \frac{r - r\_{\mathrm{nse}}}{r\_{\mathrm{nse}}} < 0.25 \\]

If the check above failed, then we proceed to an overall molar fraction check:

\\[ \epsilon\_{\mathrm{abs}} = \sum\_{i} |Y^{i} - Y^{i}\_{\mathrm{nse}}| < \epsilon\_{\mathrm{abs}}^{\mathrm{nse}} \\]

and

\\[ \epsilon\_{\mathrm{rel}} = \sum\_{i} \frac{\epsilon\_{\mathrm{abs}} }{ Y^{i} }< \epsilon\_{\mathrm{rel}}^{\mathrm{nse}} \\]


### NSE Grouping Procedure {#nse-grouping-procedure}

A grouping process is performed to all the nuclei based
on the available reaction rates.
Based on the final grouping configuration,
we determine whether the network is currently in NSE or not.


#### Determine Reaction Timescale {#determine-reaction-timescale}

Determine the reaction timescale for the available reaction rates,
which is defined as

\\[ t\_{i,k} = \frac{Y\_i}{\min(b\_{f}(k), b\_{r}(k))} \\]

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

\\[ t\_s = \frac{\min(\mathrm{dx}[0], \mathrm{dx}[1], \mathrm{dx}[2])}{c\_s} \\]

where dx is the size of the simulation cell and \\(c\_s\\) is the sound speed,
and &epsilon; is chosen by the user, which is typically &sim; 0.1 or 0.01.

Reaction rates with all these requirements satisfied will carry out
reaction timescales that will be used later for the grouping process.


#### Initialization {#initialization}

To start the grouping process, all nuclei except _p_, _n_, and _&alpha;_
initially form a group on their own.
_p_, _n_, and _&alpha;_ form a single group, called the light-isotope-group or LIG.


#### Grouping Process {#grouping-process}

Grouping process starts from the fastest reaction timescale.
We have already filtered out reaction rates that don't satisfy the NSE
conditions during the calculation of the reaction timescale.

Here consider two cases during the grouping process:

1.  If there are exactly two isotopes involved in the _k-th_ reaction that are not in LIG, then merge the isotope in the smaller group into the isotop in the larger group.
    **Note, in this case, we skip this reaction if both isotopes are**
    **already in the same group. (Perhaps we can still merge the nonLIG group**
    **to LIG at this point)???**
2.  If there is only 1 isotope involved in the _k-th_ reaction that is not
    in LIG, then merge that isotope and the group that it's in into LIG


#### Grouping Configuration {#grouping-configuration}

A final grouping configuration is obtained after the grouping process.

1.  If the network has neutron, then define NSE if all the nuclei are in the single group with an optional LIG.
2.  If the network does not have neutron, then consider a looser constraint where for isotopes  \\(Z \geq 14\\), isotopes with odd N and even N form two distinct groups.


## NSE Burn {#nse-burn}

Once the cell is determined to be in NSE, mass fraction is determined by the NSE equation.
However, a careful calculation is needed to determine &rho;, T, and \\(Y\_e\\) of the next time step
to accurately determine the appropriate NSE state as well as energy generation rates.
We will proceed with a 2nd order Runge-Kutta scheme following the similar fashion described
in this [paper](https://iopscience.iop.org/article/10.3847/1538-4357/ad8a66), which uses a table that stores NSE states at different theromodynamic conditions
instead of solving the NSE state directly on the grid.

During the reactive update, the quantities we want to update are
\\(\rho\\), \\(\rho \vec{U}\\), \\(\rho e\\), \\(\rho E\\), and \\(\rho X\_k\\). They are updated with the form:

\\[ \boldsymbol{\mathcal{U}}^{n+1} = \boldsymbol{\mathcal{U}}^{n} + \Delta t \left([\boldsymbol{\mathcal{A}}(\boldsymbol{\mathcal{U}})]^{n+1/2} + [{\bf R}(\boldsymbol{\mathcal{U}})]^{n+1/2}\right) \\]

The advective contribution, \\(\boldsymbol{\mathcal{A}}(\boldsymbol{\mathcal{U}})\\),
is already time-centered from the simplified-SDC algorithm,
so it comes down to evaluate the midpoint reactive source,
\\([{\bf R}(\boldsymbol{\mathcal{U}})]^{n+1/2}\\). The general steps are the following:

1.  Compute the advective source term for \\(Y\_e\\) via:
    \\[ \boldsymbol{\mathcal{A}}(\rho Y\_e) = \sum\_k \frac{Z\_k}{A\_k} \boldsymbol{\mathcal{A}}(\rho X\_k) \\]
    Note that this bit is already time-centered.
2.  Compute \\([{\bf R}(\rho Y\_e)]^n\\) and \\([{\bf R}(\rho e)\_{\mathrm{nuc}}]^n\\) using
    \\([\rho]^n\\), \\([T]^n\\), \\([Y\_e]^n\\) and \\([e]^n\\):
    1.  Find the NSE composition with given \\([\rho]^n\\), \\([e]^n\\),
        and \\([Y\_e]^n\\). An EOS inversion algorithm is used so
        that we determine \\([T^\*]^n\\) such that \\([e]^n\\) remains
        unchanged after switching to the NSE composition.
        Here we use \\([T]^n\\) as the initial guess and updated
        to the solution, \\([T^\*]^n\\), in the end.
    2.  Compute the thermal neutrino losses,
        \\(\epsilon\_{\nu,\mathrm{thermal}}\\), using the NSE composition.
    3.  Evaluate \\(\dot{Y}\_{\mathrm{weak}}\\) and neutrino losses, \\(\epsilon\_{\nu,\mathrm{react}}\\),
        from weak reactions only as they are the only contributing reactions in NSE.
    4.  Evaluate \\([{\bf R}(\rho Y\_e)]^n\\) as:
        \\[ [{\bf R}(\rho Y\_e)]^n = [\rho]^n \sum\_k Z\_k [\dot{Y}\_{\mathrm{k, weak}}]^n \\]
    5.  Evaluate \\([{\bf R}(\rho e)\_{\mathrm{nuc}}]^n\\) as:
        \\[ [{\bf R}(\rho e)\_{\mathrm{nuc}}]^n = - N\_A c^2 \sum\_k [\dot{Y}\_{\mathrm{k, weak}}]^n m\_k \\]
        where the nuclei mass, \\(m\_k\\) is defined as:
        \\[ m\_k c^2 = (A\_k - Z\_k) m\_n c^2 + Z\_k (m\_p + m\_e) c^2 - B\_k \\]
    6.  The full reactive source term, \\([{\bf R}(\rho e)]^n\\) is then:
        \\[ [{\bf R}(\rho e)]^n = [{\bf R}(\rho e)\_{\mathrm{nuc}}]^n - [\rho]^n \left(\epsilon\_{\nu,\mathrm{thermal}} + \epsilon\_{\nu,\mathrm{react}}\right) \\]
3.  Now evolve \\(\rho\\), \\(\rho e\\), and \\(\rho Y\_e\\) to midpoint in time:
    \\[ \boldsymbol{\mathcal{U}}^{n+1/2} = \boldsymbol{\mathcal{U}}^{n} + \frac{\Delta t}{2} \left([\boldsymbol{\mathcal{A}}(\boldsymbol{\mathcal{U}})]^{n+1/2} + [{\bf R}(\boldsymbol{\mathcal{U}})]^{n}\right) \\]
    Note that there is no reactive source term for \\(\rho\\) and the advective
    source term is constant throughout the reactive update.
4.  Compute \\([{\bf R}(\rho Y\_e)]^{n+1/2}\\) and
    \\([{\bf R}(\rho e)\_{\mathrm{nuc}}]^{n+1/2}\\) following the same
    procedure as above. This time, it uses
    \\([\rho]^{n+1/2}\\), \\([Y\_e]^{n+1/2}\\) and \\([e]^{n+1/2}\\) as input
    and uses the updated \\([T]^n\\) as initial guess for the EOS inversion
    algorithm.

Now that we obtain the midpoint reactive source term, we can
evolve all thermodynamic quantities to new time, \\(t^{n+1}\\) via:

\\[ \boldsymbol{\mathcal{U}}^{n+1} = \boldsymbol{\mathcal{U}}^{n} + \Delta t \left([\boldsymbol{\mathcal{A}}(\boldsymbol{\mathcal{U}})]^{n+1/2} + [{\bf R}(\boldsymbol{\mathcal{U}})]^{n+1/2}\right) \\]

Lastly, the composition is updated by finding the corresponding NSE state
using \\([\rho]^{n+1}\\), \\([e]^{n+1}\\), and \\([Y\_e]^{n+1}\\).


## Application: Double-Detonation {#application-double-detonation}

Here we showcase the use of NSE integration in the double-detonation
model for Type Ia Supernovae. The supernovae starts off with a surface
helium detonation, which releases a shock wave inward to the carbon core,
which is indicated by the density gradient. This shock wave ignites the
carbon core releasing the carbon detonation. During carbon detonation,
temperature can reach more than &sim; 6 billion Kelvin, which is
sufficient for the core to be in NSE.

The movie below showcases the double-detonation, where the
black curve in the energy generation plot maps out the outline
at which NSE takes place. We basically see the NSE region grows
as the second detonation propagates outward.

<video width="1000" height="600" controls><source src="/videos/subchandra.mp4" type="video/mp4">
Your browser does not support the video tag.
Videos only work in static folders.</video>
