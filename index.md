@def title = "MESA tutorial"
@def tags = ["syntax", "code"]

# Introduction

\tableofcontents <!-- you can use \toc as well -->

This website is part of the *XXXVII Canary Islands Winter School of Astrophysics: Massive stars as cosmic tools*, and is meant to provide a basic introduction to numerical stellar evolution using the MESA stellar evolution code. This is a brief tutorial meant to cover the very basics to produce single and binary star simulations, for a full week of hands on tutorials the yearly MESA summer school is highly recommended.

It is assumed that going into the tutorial session you will already have MESA installed and running in your computer. If not, your task through the session will be to install it and try to catch up with the rest!

## Pre-requisites

Before starting you need to have a working MESA installation. Straightforward instructions are provided in the MESA website. In particular, do the following:

- Install MESA in your computer following the instructions in [https://docs.mesastar.org/en/26.4.1/installation.html](https://docs.mesastar.org/en/26.4.1/installation.html)
- Verify your MESA installation by running a test model following the instructions in [https://docs.mesastar.org/en/26.4.1/using_mesa/running.html](https://docs.mesastar.org/en/26.4.1/using_mesa/running.html)

Throughout the tutorial we will be making use of version `26.4.1`. If using a newer version changes might be required to run the included templates.

## Reading MESA data

MESA provides a powerful set of live-plotting tools ([see here](https://docs.mesastar.org/en/26.4.1/using_mesa/using_pgstar.html)) that can be adjusted in various ways. These are useful to debug your runs quickly as well as to provide something to see during your runs other than a wall of text. However, when presenting results you often want to use a dedicated plotting package. If you 

## Structure of a single star template

MESA provides a basic starting point to run single star simulations in `$MESA_DIR/star/work`. This can be used as a starting point to build more complicated setups. It is recommended to not modify this directory, but rather copy it outside the MESA source code to keep the original. The barebones structure of such a work directory looks something like this (this is for one specific template we will use for single star simulations):

```bash
pablom@fedora:~/work/class/tenerife_school/lab_tests/part1$ tree
.
├── clean
├── history_columns.list
├── inlist
├── inlist_pgstar
├── inlist_project
├── LOGS
├── make
│   └── makefile
├── mk
├── photos
├── profile_columns.list
├── re
├── rn
└── src
    ├── run.f90
    └── run_star_extras.f90

```

Each of these files serve different purposes:

- `rn`, `re`: These are used to run or restart a simulation.
- `clean`, `mk`: Scripts used to compile the work directory before a run. This should be done once before starting to run simulations, and afterwards its only needed if files in `src` are modified.
- `make`: Contains a `makefile` used to define how this work directory is compiled.
- `history_columns.list` and `profile_columns.list`: These define properties that will be printed in output files. History files contain properties that are given by a single scalar per timestep, while profiles contain the full structure of the star at a given step.
- `inlist*`: Inlist files are the main configuration tool for MESA. You will be working with very barebones files but there are several default options. You can explore these in the files under `$MESA_DIR/star/defaults`. The main file containing properties of the simulated star is `inlist_project`.
- `src`: contains Fortran source files that can be used to program custom behavior in the simulation. Normally one does not touch `run_star_extras.f90`.
- `LOGS`: Contains output data as ASCII tables. `history.data` files contain single values per step (such as effective temperature), and for some steps profiles will be saved with the name `profileX.data`, where `X` will be an integer. Each of these is a full snapshot of the structure of the star. A file `profiles.index` contains the mapping between profiles and model number.
- `photos`: MESA stores binary files here that can be used to restart a simulation. This can be done by executing `./re x050` where `x050` is a stored snapshot produced by the simulation. By default the numbers give the last three digits of the model number.

## Structure of a binary template

A binary star template looks very similar to the prior one, with a few differences:

```bash
pablom@fedora:~/work/class/tenerife_school/lab_tests/part2$ tree
.
├── clean
├── history_columns.list
├── inlist
├── inlist1
├── inlist2
├── inlist_pgbinary
├── inlist_pgstar
├── inlist_project
├── LOGS1
├── make
│   └── makefile
├── mk
├── photos
├── profile_columns.list
├── re
├── rn
└── src
    ├── binary_run.f90
    ├── run_binary_extras.f90
    └── run_star_extras.f90
```

In particular you can see that there are now two inlists for each star `inlist1` and `inlist2`, containing properties defined uniquely for each, as well as two output directories for logs of each star. In addition to this when a simulation is ran, a `binary_history.data` file is created at the top level containing information about scalar properties of the binary orbit at each step (such as separation, period, rate of angular momentum loss, etc...).

The template also includes an additional source file, `run_binary_extras.f90`. This can be used to write custom code concerning the binary system and its evolution (for instance, a new prescription for orbital angular momentum loss).