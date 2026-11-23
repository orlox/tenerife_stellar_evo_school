+++
title = "Isochrones"
hascode = true
rss = ""
rss_title = "Isochrones"
rss_pubdate = Date(2026, 11, 23)

tags = ["isochrone", "mesa"]
+++

# Isochrones

\toc

This section is meant as a more open extra task for people who have completed the previous parts of the exercise. Idea of this exercise is to combine multiple simulations of single star evolution to construct an isochrone. This involves manually processing the output files produced by single star simulations.

The general objective is to produce isochrones akin to Figure  of [Choi et el (2016)](https://scixplorer.org/abs/2016ApJ...823..102C/abstract).To avoid excessive cost of simulations by resolving the TP-AGB or the helium flash, simulations should be restricted to masses above $3M_\odot$ and a custom termination condition can be placed to finish a simulation at core helium depletion (would also suggest restricting simulations to be $<100M_\odot$). If you wish to include wind mass loss you can lookout for the relevant options in the inlists of part 2. The same template used in the convergence exercise can be adapted here. If working with others who reached this stage you can discuss steps to follow and also spread the work to perform the necessary simulations.