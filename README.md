<div align="center" style="padding-bottom: 3em;">
<img width="300px" align="center" src="./src/PASCal/static/images/PASCal_logo_v1.png">
</div>

<div align="center">

[![PASCal](https://img.shields.io/badge/try-PASCal-navy?style=flat&color=navy&cacheSeconds=3600&link=https%3A%2F%2Fpascalapp.co.uk)](https://www.pascalapp.co.uk)

[![JOSS](https://joss.theoj.org/papers/3bfe66743c9a86f34d3c1b2f17906261/status.svg)](https://joss.theoj.org/papers/3bfe66743c9a86f34d3c1b2f17906261)

[![Documentation](https://img.shields.io/badge/docs-online-black?style=flat&logoColor=white&logo=bookstack&color=olive&cacheSeconds=3600&link=https%3A%2F%2Fdocs.pascalapp.co.uk)](https://docs.pascalapp.co.uk)

</div>

Principal Axis Strain Calculator (PASCal) is a web tool and open source software package designed to help scientists analyse non-ambient lattice parameter data.
It is written entirely in Python, using plotly to visualise the data, and is also available without installation as a web tool, available at [pascalapp.co.uk](https://www.pascalapp.co.uk).
The code is available at [MJCliffe/PASCal](https://github.com/MJCliffe/PASCal) on GitHub, and the documentation at [docs.pascalapp.co.uk](https://docs.pascalapp.co.uk).
Installation instructions can be found [below](#offline-installation), and example Jupyter notebooks showcasing the functionality can be found in `./examples` folder on GitHub, or in the online documentation.

A paper summarising the original motivation and theory behind PASCal is published at [J. Appl. Cryst. (2012). 45, 1321-1329](https://doi.org/10.1107/S0021889812043026) ([arXiv](http://arxiv.org/pdf/1204.3007.pdf)). Please cite this if you use PASCal in a publication. This publication was produced for a previous version of PASCal and so description of the software itself is out of date. PASCal is designed might not be the most appropriate method for full analysis of your data, so please read the sections below on [errors](#errors), [fitting](#fitting) and [strains](#strain-calculation) if you have particular needs.

# Quick start

PASCal takes experimentally determined lattice parameters measured as a function of a parameter (temperature, pressure or state of charge), then calculates the strain matrix, diagonalises this matrix to find the principal strains, and then carries out fits to simple functions to get summary metrics.
Paste your data in, click calculate and see the results!
There are helpful tips on hover, but more detailed information is below.

## Input

Data should be input as plain text, with eight values per row, the control parameter $X$, its error $\sigma(X)$ and the lattice parameters (in Å and degrees $^\circ$): $(a, b, c, \alpha, \beta, \gamma)$.

Lines beginning with "#" are treated as comments and ignored: `# This line is metadata`.

There are a number of other options:

- Data type: whether the data vary with temperature, pressure or electrochemical state-of-charge.
This does not alter how the principal strains are calculated, but rather the fits (and units) that are carried out.
- For pressure data, whether this is a high pressure phase, and if so at what critical pressure it undergoes the transition.
- For electrochemical data, the maximum degree of Chebyshev polynomial to use to fit the lattice parameter and volume data.
- Advanced strain options, Eulerian strain and finite/infinitesimal strain.
This determines which strain formalism is used to calculate the strain.

## Output

### Summary Table

The summary table shows the three principal axes (in crystallographic coordinates [UVW]) and the coefficient of thermal expansion/compressibility/electrochemical strain derivative along each of these directions with the errors (see [below](#errors) for details). The coefficients are determined through fitting: using the linear fit for temperature, empirical fit for pressure and the Chebyshev fit for electrochemical strain. The principal axis shown is for the median data point, with the full list below.

### Plots

Below the summary tables are the key plots. These interactive plots are generated using plotly. A high resolution png can be saved using the camera button. Any given data series can be hidden by clicking on its symbol in the key and the plot. The raw data for each plot is shown in tables below (including the fitted values) if you wish to replot using different software. The indicatrix is a plot of the expansivity/compressibility tensor, showing its value along each direction. Positive values are blue, negative values are red and exact values can be measured using mouseover.

Available plots are (by supported data type, T, P and X):

1. Principal strains as a function of parameter with fit (TPX)
2. Derivative of fitted strain as a function of parameter (PX)
3. Volume (TPX)
4. Fit as a function of degree of polynomial (X)
5. Indicatrix (TPX)

### Tables

The raw data plotted in the plots is shown in the tables at the bottom of the page (data type):

1. Principal strains with fitted strain value (TPX).
2. Principal strain axes (TPX). The axes are also used to assign the eigenstrains to a particular direction (1,2 or 3). This is a difference from the original version of PASCal which just sorted by the magnitude of the strain. The directions are insensitive to inversion and are shown with the largest component positive.
3. Calculated derivative of the strain with parameter (PX).
4. Volume with fitted volume (TPX).
5. Parsed input values (TPX). These allow you to check the values have been read correctly in the event of anomalous results.

## Errors

PASCal does not use uncertainties in cell parameters or volumes in its fitting or strain calculations, and instead only uses errors in the control parameter, $x$. The uncertainties in $x$ are used as weights for both linear and non-linear fitting but are not directly propagated through to the final parameter uncertainties. This usually provides a reasonable approximation as typically the inherent scatter of the strain is the primary source of uncertainty in the fitted parameter. In the case that the data has very small errors, this method will overestimate the magnitude of the errors. It can also prove inaccurate for small data sets.

If you have the magnitudes of the errors on your lattice parameters, it is possible to propagate those errors through to get more accurate estimates, and [some software packages](#other-useful-software) (e.g. WINStrain) can give you errors on strains from errors on lattice parameters. However, to get very accurate estimates of errors it is also necessary to include the covariance of lattice parameters, and for low symmetry systems, the propagation of errors in lattice parameter angles to strain errors is a problem with no unique solution.
Additionally, since the errors that are calculated from Rietveld fits or single crystal calculations often seem to underestimate the true errors in lattice parameters, these will underestimate the true error in your calculated values if used naively (see [Haestier, J Appl. Cryst 2009, 42, 798](https://doi.org/10.1107/S0021889809024376), [Herbstein, Acta Cryst B 2000, 56, 547](https://doi.org/10.1107/S010876810000269X); [Taylor, R. & Kennard, O. Acta Cryst B 1986, 42, 112.](https://doi.org/10.1107/S0108768186098506))

In PASCal, linear fits use White’s heteroscedascity consistent error estimates and non-linear fits use the residuals to calculate the error estimates.

## Fitting

PASCal as an automated fitting tool cannot guarantee always finding the best fit and it is worth considering whether another [software package](#other-useful-software) to carry out these kind of calculation if they wish to examine this data more finely. There are some specific issues that users should consider, especially for high pressure data sets:

- PASCal does not use cell-parameter or volume errors in fitting or strain calculation, which can skew results. The contribution of volume errors to final errors is discussed in Angel ([Reviews in Mineralogy and Geochemistry, Vol. 41, High-Temperature and High Pressure Crystal Chemistry 2000](https://doi.org/10.1515/9781501508707-020)).
- Fitting to non-linear equations of state can be unstable, especially if the equation of state does not describe the behaviour well. Even where this instability occurs, it does not often seem to perturb the fitted parameters too badly. The errors, being derived from the derivatives of the parameters, are much more susceptible to instabilities and failure to converge. As the input errors (as currently implemented) are just weights, multiplying the errors by a constant can sometimes nudge the model towards convergence.
 - Ambient pressure (or critical pressure) data points can have outsize effects on the fitted parameters for both the Birch-Murnaghan and empirical equations of state. Care should be taken with this point.
As different equations of state are used for the principal axis and volume fits, they should not be expected to produce self-consistent results. Other equations of state are of course in use and may be more appropriate for some materials.

## Strain calculation

PASCal calculates finite Lagrangian strains by default. Options to use finite and/or Lagrangian strains are provided. PASCal also uses a numerical approach that means the strains calculated are effectively averages of both direction and magnitude. It assigns the strain to the axes by comparing the eigenvectors to the first data point, however this approach can fail where there is significant eigenvector rotation or very noisy data. Fitting of the calculated strains outside of PASCal and manually assigning the strains to the appropriate axes is currently the best approach for these kinds of systems. The directions of the principal axes are taken, like the expansivities, from the median data point.

## Offline Installation

PASCal can be also run offline using a browser as a GUI.
This requires Python 3.9+, NumPy, Plotly and Flask.
Full requirements (with versions) available in `pyproject.toml`.
For the simplest use case, you can just clone or downloaded this repository, and use `pip` to install the local copy of the repository (ideally into a virtual environment of your choice):

```shell
git clone git@github.com:MJCliffe/PASCal
cd PASCal
pip install .
```

You can then either launch the app with `flask --app src/PASCal/app.py run` and access the app in your browser at [http://localhost:5000](http://localhost:5000) (by default), or simply use the Python library in your own scripts.
The software was initially designed to used as a web app, but please do submit any issues to the [GitHub Repository](./CONTRIBUTING.md).

## Issues and Feature Requests

If you find any bugs in the code, errors in the documentation or have any feature requests, please do [contribute](./CONTRIBUTING.md) via the GitHub Repository.

## Other Useful Software

There are a wide range of other useful programs available:

- [WINstrain and EOSfit](http://www.rossangel.com/). WINStrain provides a large number of different options for
calculating strains from lattice parameters, but is no longer supported. EOSfit is a powerful tool for fitting equations of state (principally pressure) and recent versions include strain calculation also.
- [STRAIN](https://www.cryst.ehu.es/cryst/strain.html). The Bilbao Crystallographic Server can calculate strain calculations for a single pair of lattice parameters.
- [ELATE](https://progs.coudert.name/elate). A tool for analysing full elastic constant tensors.

If you know of any other useful software that should be added to the list or have any other questions,
please email matthew[dot]cliffe[at]nottingham[dot]ac[dot]uk
