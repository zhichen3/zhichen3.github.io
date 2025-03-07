+++
title = "Interferometry"
author = ["zhi"]
date = 2025-02-02T00:00:00-05:00
type = "gallery"
draft = false
image = "interferometry_setup.png"
+++

This was my research project during my [SULI](https://science.osti.gov/wdts/suli) internship in 2021.
I was interested in estimating the precision of the
[two-photon interferometry](https://astro.theoj.org/article/39641-two-photon-amplitude-interferometry-for-precision-astrometry) technique
for measuring the relative separation between two light-sources (i.e. stars)
using Markov-Chain Monte Carlos (MCMC) simulation.


## Two-photon Interferometer Overview {#two-photon-interferometer-overview}

Assume there are two sources which can be observed simultaneously from two stations,
**L**  and **R** , with single spatial mode inputs _a_, _b_  and _e_, _f_.
Both sources send out photons in the form of plane wave, the path length difference
between the two stations yielding phase delays \\(\delta\_1\\) and \\(\delta\_2\\) between the photons observed
at channels _a_, _e_ from source 1 and _b_, _f_ from source 2, respectively.
If the two detected photons are close enough in frequency and arrival time,
then the pattern of coincidences measured at the outputs _c_, _d_ and _g_, _h_
will be sensitive to the difference in phase delays
after interference at the symmetric beam splitter in each station.

\\[ \Delta\delta = \delta\_{1} - \delta\_{2} = \frac{2\pi}{\lambda} \vec{B} \cdot (\hat{s\_1} - \hat{s\_2}) + \psi \\]

where \\(\vec{B}\\) is the baseline of the two detectors and &psi; is a constant phase-shift
due to instrumental path length difference between the two telescope.
And here &Delta;&delta; encodes the relative separation between the two sources.


## Procedure of Bayesian Analysis. {#procedure-of-bayesian-analysis-dot}

The analysis involves two parts:

1.  Simulate coincidences following Poisson process
2.  Feed the sampled data to MCMC and see if we can recover the original parameters used
    such as visibility and the separation between two sources.


### Simulating Coincidence {#simulating-coincidence}

{{< figure src="/ox-hugo/simulated_coincidences.jpeg" caption="<span class=\"figure-number\">Figure 1: </span>Schematic picture the fringe pattern. The blue curve represents a theoretical fringe pattern, and orange points are events detected." width="75%" >}}

Summary:

1.  Since fringes will vary in frequency as Earth rotates, we first determine the period
    of each fringe cycle denoted by &Delta;t's.
2.  Determine the number of detections within each fringe cycle via \\(\bar{n} \times \Delta t\\)
3.  Determine the phase, &phi;, of these events in each fringe cycle.
4.  Find the timestamp of these events corresponding to the phase, &phi;.

We knew the rate for different two-photon coincidence rate type (i.e. the blue curve)
will be in the form:

\\[ R\_{\pm}(t) = \bar{n} \left(1 \pm V \cos(\delta\_1 - \delta\_2) \right) \\]

where \\(\bar{n}\\) is the fringed-average value of R, and _V_ is the fringe visibility calculated
from the fluxes of the two sources. We can determine &Delta;t's from the curve, as well as the
number of events in each fringe cycle.

Now the form of R(t) tells us that the probability density function is of the form:

\\[ PDF(x) = \frac{1 \pm V\cos{(x)}}{2\pi}  \quad \quad \quad  x\in[-\pi, \pi] \\]

and the cumulative density function (CDF) after integrating PDF from -&pi; to &phi;.

\\[ CDF(\phi) = \frac{\phi \pm V\sin{(\phi)} + \pi}{2\pi}  \quad \quad \quad  \phi\in[-\pi, \pi] \\]

With a random number generator following Poisson distribution, we feed a number from 0 to 1
to CDF. By inverting the CDF, we then obtain the phase, &phi;, representing a coincidence.
After obtaining &phi; for each fringe cycle,
we can just find the corresponding timestamp corresponding to R(t).


### MCMC Sampling {#mcmc-sampling}

After simulating our data points following Poisson distribution,
now we explore the posterior using MCMC procedure. There are 4 parameters to vary,
visibility, _V_, separation of the two sources in the east-west direction &Delta;d_E,
north-south direction &Delta;d_N, and the arbitrary phase, &psi;.

The result gives a bunch of triangle correlation plots showing correlation between different
parameters.

{{< figure src="/ox-hugo/ew_1as_mix_20_1.png" caption="<span class=\"figure-number\">Figure 2: </span>Triangle correlation polots generated via the [corner](https://github.com/dfm/corner.py) package. The vertical dashed lines represent 2.3%, 16%, 50%, 84%, and 99.4% quantiles of the GAussian. The orange point indicate the true value of each parameter." width="75%" >}}


## Result {#result}

In the end, I was able to show two telescopes with an effective collecting area of \\(\sim 2\text{m}^2\\),
we could detect fringing and measure the astrometric separation of the sources at \\(\sim\\) 100 &micro;as
of precision in a few hours of observations.

This work is published in [Physical Review D](https://journals.aps.org/prd/abstract/10.1103/PhysRevD.107.023015)
