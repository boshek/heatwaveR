# heatwaveR <img src="vignettes/logo.png" width=200 align="right" />

[![CRAN\_Status\_Badge](http://www.r-pkg.org/badges/version/heatwaveR)](https://cran.r-project.org/package=heatwaveR)
[![Coverage
status](https://codecov.io/gh/robwschlegel/heatwaveR/branch/master/graph/badge.svg)](https://codecov.io/github/robwschlegel/heatwaveR?branch=master)
[![JOSS](http://joss.theoj.org/papers/10.21105/joss.00821/status.svg)](https://doi.org/10.21105/joss.00821)
[![DOI](https://zenodo.org/badge/DOI/10.5281/zenodo.1324308.svg)](https://doi.org/10.5281/zenodo.1324308)
[![Downloads](https://cranlogs.r-pkg.org/badges/grand-total/heatwaveR)](https://cran.r-project.org/web/packages/heatwaveR/index.html)

The **`heatwaveR`** package is a project-wide update to the
[**`RmarineHeatWaves`**](https://github.com/ajsmit/RmarineHeatWaves)
package, which is itself a translation of the original [Python
code](https://github.com/ecjoliver/marineHeatWaves) written by Eric C.
J. Oliver. The **`heatwaveR`** package also uses the same naming
conventions for objects, columns, and arguments as the Python code.

The **`heatwaveR`** R package contains the original functions from the
**`RmarineHeatWaves`** package that calculate and display marine
heatwaves (MHWs) according to the definition of Hobday et al. (2016) as
well as calculating and visualising marine cold-spells (MCSs) as first
introduced in Schlegel et al. (2017a). It also contains the
functionality to calculate the categories of MHWs as outlined in Hobday
et al. (2018).

This package does what **`RmarineHeatWaves`** does, but faster. The
entire package has been deconstructed and modularised, and we are
continuing to implement slow portions of the code in C++. This has
alleviated the bottlenecks that slowed down the climatology creation
portions of the code as well as generally creating an overall increase
in the speed of the calculations. Currently the R code runs about as
fast as the original python functions, at least in as far as applying it
to single time series of temperatures. Readers familiar with both
languages will know about the ongoing debate around the relative speed
of the two languages. In our experience, R can be as fast as python,
provided that attention is paid to finding ways to reduce the
computational inefficiencies that stem from i) the liberal use of
complex and inefficient non-atomic data structures, such as data frames;
ii) the reliance on non-vectorised calculations such as loops; and iii)
lazy (but convenient) coding that comes from drawing too heavily on the
`tidyverse` suite of packages. We will continue to ensure that
**`heatwaveR`** becomes more-and-more efficient so that it can be
applied to large gridded data products with ease.

This new package was developed and released in order to better
accommodate the inclusion of the definitions of atmospheric heatwaves in
addition to MHWs. Additionally, **`heatwaveR`** also provides the first
implementation of a definition for a ‘compound heatwave’. There are
currently multiple different definitions for this type of event and each
of which has arguments provided for it within the `ts2clm()` and
`detect_event()` functions.

This package may be installed from CRAN by typing the following command
into the console:

`install.packages("heatwaveR")`

Or the development version may be installed from GitHub with:

`devtools::install_github("robwschlegel/heatwaveR")`

## The functions

| Function          | Description                                                                                                    |
| ----------------- | -------------------------------------------------------------------------------------------------------------- |
| `ts2clm()`        | Constructs seasonal and threshold climatologies as per the definition of Hobday et al. (2016).                 |
| `detect_event()`  | The main function which detects the events as per the definition of Hobday et al. (2016).                      |
| `block_average()` | Calculates annual means for event metrics.                                                                     |
| `category()`      | Applies event categories to the output of `detect_event()` based on Hobday et al. (2018).                      |
| `exceedance()`    | A function similar to `detect_event()` but that detects consecutive days above/below a given static threshold. |
| `event_line()`    | Creates a time series line graph of the heatwave or cold-spell results from `detect_event()`.                  |
| `lolli_plot()`    | Creates a lolliplot time series of a selected event metric from the results generated by `detect_event()`.     |
| `geom_flame()`    | Creates flame polygons of heatwaves or cold-spells from a time series.                                         |
| `geom_lolli()`    | Creates lolliplots from a time series of a selected event metric.                                              |

The package also provides data of observed SST records for three
historical MHWs: the 2011 Western Australia event, the 2012 Northwest
Atlantic event, and the 2003 Mediterranean event.

## The heatwave metrics

The `detect_event()` function will return a list of two tibbles (see the
**`tidyverse`**), `climatology` and `event`, which are the time series
climatology and MHW (or MCS) events, respectively. The climatology
contains the full time series of daily temperatures, as well as the the
seasonal climatology, the threshold and various aspects of the events
that were detected. The software was designed for detecting extreme
thermal events, and the units specified below reflect that intended
purpose. However, various other kinds of extreme events (e.g. rainfall)
may be detected according to the ‘heatwave’ specifications, and if that
is the case, the appropriate `minDuration` etc. and units of measurement
need to be determined by the user.

| Climatology metric  | Description                                                                                                                                                                                                                                                 |
| ------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `doy`               | Julian day (day-of-year). For non-leap years it runs 1…59 and 61…366, while leap years run 1…366. This column will be named differently if another name was specified to the `doy` argument.                                                                |
| `t`                 | The date of the temperature measurement. This column will be named differently if another name was specified to the `x` argument.                                                                                                                           |
| `temp`              | If the software was used for the purpose for which it was designed, seawater temperature (deg. C) on the specified date will be returned. This column will of course be named differently if another kind of measurement was specified to the `y` argument. |
| `seas`              | Climatological seasonal cycle (deg. C).                                                                                                                                                                                                                     |
| `thresh`            | Seasonally varying threshold (e.g., 90th percentile) (deg. C).                                                                                                                                                                                              |
| `var`               | Variance (standard deviation) per `doy` of `temp` (deg. C). (not returned by default as of v0.3.5)                                                                                                                                                          |
| `threshCriterion`   | Boolean indicating if `temp` exceeds `thresh`.                                                                                                                                                                                                              |
| `durationCriterion` | Boolean indicating whether periods of consecutive `threshCriterion` are \>= `minDuration`.                                                                                                                                                                  |
| `event`             | Boolean indicating if all criteria that define a MHW or MCS are met.                                                                                                                                                                                        |
| `event_no`          | A sequential number indicating the ID and order of occurrence of the MHWs or MCSs.                                                                                                                                                                          |

The events are summarised using a range of event metrics:

| Event metric           | Description                                                                                                                                         |
| ---------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------- |
| `event_no`             | A sequential number indicating the ID and order of the events. This allows one to match/join results between the `climatology` and `event` outputs. |
| `index_start`          | Row number from the given time series where the event starts.                                                                                       |
| `index_peak`           | Row number from the given time series where the event peaks.                                                                                        |
| `index_end`            | Row number from the given time series where the event ends.                                                                                         |
| `duration`             | Duration of event (days).                                                                                                                           |
| `date_start`           | Start date of event (date).                                                                                                                         |
| `date_peak`            | Date of event peak (date).                                                                                                                          |
| `date_end`             | End date of event (date).                                                                                                                           |
| `intensity_mean`       | Mean intensity (deg. C).                                                                                                                            |
| `intensity_max`        | Maximum (peak) intensity (deg. C).                                                                                                                  |
| `intensity_var`        | Intensity variability (standard deviation) (deg. C).                                                                                                |
| `intensity_cumulative` | Cumulative intensity (deg. C x days).                                                                                                               |
| `rate_onset`           | Onset rate of event (deg. C / day).                                                                                                                 |
| `rate_decline`         | Decline rate of event (deg. C / day).                                                                                                               |

`intensity_max_relThresh`, `intensity_mean_relThresh`,
`intensity_var_relThresh`, and `intensity_cumulative_relThresh` are as
above except relative to the threshold (e.g., 90th percentile) rather
than the seasonal climatology.

`intensity_max_abs`, `intensity_mean_abs`, `intensity_var_abs`, and
`intensity_cumulative_abs` are as above except as absolute magnitudes
rather than relative to the seasonal climatology or threshold.

Note that `rate_onset` and `rate_decline` will return `NA` when the
event begins/ends on the first/last day of the time series. This may be
particularly evident when the function is applied to large gridded data
sets. Although the other metrics do not contain any errors and provide
sensible values, please take this into account in the interpretation of
the output. It must also be noted that events whose `date_peak` occur on
the same day as the `date_start` or `date_end` of the event will return
small negative values. This tends to only occur in areas with persistent
ice cover. The authors are currently thinking about how best to handle
this exception.

## The Vignettes

For detailed explanations and walkthroughs on the use of the
**`heatwaveR`** package please click on the Vignettes tab in the toolbar
above, or follow the links below:

  - For a basic introduction to the [detection and
    visualisation](https://robwschlegel.github.io/heatwaveR/articles/detection_and_visualisation.html)
    of events.
  - For an explanation on the use of the
    [exceedance](https://robwschlegel.github.io/heatwaveR/articles/exceedance.html)
    function.
  - For a walkthrough on the calculation and visualisation of [event
    categories](https://robwschlegel.github.io/heatwaveR/articles/event_categories.html).
  - For examples on the calculation of atmospheric events with
    [alternative
    thresholds](https://robwschlegel.github.io/heatwaveR/articles/complex_clims.html).
  - For a demonstration on how to [download and prepare OISST
    data](https://robwschlegel.github.io/heatwaveR/articles/OISST_preparation.html).
  - Which may then have the `detect_event()` function applied to the
    [gridded
    data](https://robwschlegel.github.io/heatwaveR/articles/gridded_event_detection.html),
    and then fit a GLM and plot the results.

## The Marine Heatwave Tracker

To see the **`heatwaveR`** package in action, check out the [Marine
Heatwave Tracker](http://www.marineheatwaves.org/tracker.html) website.
This is a daily updating global analysis of where in the world marine
heatwaves are occurring. It has near real-time information as well as
historic data going back to January 1st, 1982 and uses the Hobday et
al. (2018) colour scheme to show how intense the MHWs are.

## Contributing to **`heatwaveR`**

To contribute to the package please follow the guidelines
[here](https://robwschlegel.github.io/heatwaveR/CONTRIBUTING.html).

Please use this [link](https://github.com/robwschlegel/heatwaveR/issues)
to report any bugs found.

## Citing **`heatwaveR`**

Because **`heatwaveR`** is and always will be free to use open source
software, its citation in scientific literature and other sources is the
primary metric through which the continued development of this package
is motivated for. Therefore, if the **`heatwaveR`** package is used in
any analyses please acknowledge this through the following citation:

Robert W. Schlegel and Albertus J. Smit (2018). heatwaveR: A central
algorithm for the detection of heatwaves and cold-spells. Journal of
Open Source Software, 3(27), 821, <https://doi.org/10.21105/joss.00821>

The BibTeX citation may be accessed in R with:

`citation("heatwaveR")`

For a list of sources that have cited **`heatwaveR`** see the Citations
tab in the toolbar at the top of this page. If you do not see your
publication in the list of citations and would like it added please
contact the developer (see below).

## References

Hobday, A.J. et al. (2016). A hierarchical approach to defining marine
heatwaves. Progress in Oceanography, 141, pp. 227-238.

Schlegel, R. W., Oliver, E. C. J., Wernberg, T. W., Smit, A. J. (2017a).
Nearshore and offshore co-occurrences of marine heatwaves and
cold-spells. Progress in Oceanography, 151, pp. 189-205.

Schlegel, R. W., Oliver, E. C., Perkins-Kirkpatrick, S., Kruger, A.,
Smit, A. J. (2017b). Predominant atmospheric and oceanic patterns during
coastal marine heatwaves. Frontiers in Marine Science, 4, 323.

Hobday, A. J., Oliver, E. C. J., Sen Gupta, A., Benthuysen, J. A.,
Burrows, M. T., Donat, M. G., Holbrook, N. J., Moore, P. J., Thomsen, M.
S., Wernberg, T., Smale, D. A. (2018). Categorizing and naming marine
heatwaves. Oceanography 31(2).

## Acknowledgements

The Python code was written by Eric C. J. Oliver.

Contributors to the Marine Heatwaves definition and its numerical
implementation include Alistair J. Hobday, Lisa V. Alexander, Sarah E.
Perkins, Dan A. Smale, Sandra C. Straub, Jessica Benthuysen, Michael T.
Burrows, Markus G. Donat, Ming Feng, Neil J. Holbrook, Pippa J. Moore,
Hillary A. Scannell, Alex Sen Gupta, and Thomas Wernberg.

The translation from Python to R was done by A. J. Smit and the graphing
functions were contributed by Robert. W. Schlegel.

## Contact

Robert W. Schlegel

Data Scientist

Laboratoire d’Océanographie de Villefranche-sur-Mer, LOV

Institut de la Mer de Villefranche, IMEV

<robert.schlegel@imev-mer.fr>
