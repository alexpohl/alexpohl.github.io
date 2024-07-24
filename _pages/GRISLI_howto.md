---
layout: smallfont_page
permalink: /GRISLI_howto/
title: GRISLI version 8, a (non-official) user-guide
description: Tutorial to run the GRISLI (Grenoble ice sheet and land ice) model, with specific indications for the cluster CCUB in Dijon.
nav: false
---

A user-guide for the GRISLI (Grenoble ice sheet and land ice) model at U. Bourgogne. A 3D ice-sheet model designed for deep-time, large-scale investigations.

This page is not intended to replace the official [GRISLI user guide](https://www.cmcc.it/publications/rp0249-grenoble-ice-shelf-and-land-ice-model-a-practical-user-guide), but rather to provide additional / alternative material.

Updated model versions are now available [here](https://gmd.copernicus.org/articles/11/5003/2018/) for instance. This tutorial refers to the version used by e.g. [Pohl et al. (2016)](https://agupubs.onlinelibrary.wiley.com/doi/full/10.1002/2016PA002928) or [Ladant et al. (2014)](https://agupubs.onlinelibrary.wiley.com/doi/full/10.1002/2013PA002593).

Information on the CCUB cluster can be found [here](https://ccub.u-bourgogne.fr/dnum-ccub/spip.php?article959).

Although the tutorial is built with many sections for readibility, I suggest progressing linearly since knowhow acquired in section n will sometimes be useful for section n+1.

__Tutorial status__: this pags is under construction.

# Introduction

This tutorial describes how to run the GRISLI (Grenoble ice sheet and land ice) model ([Ritz, 2001](https://agupubs.onlinelibrary.wiley.com/doi/abs/10.1029/2001jd900232)) on the linux cluster of Univ. Bourgogne (CCUB, Dijon). It should be easy to adapt to other clusters.

This tutorial shows how to use the model in a simple way, by forcing the model with climatic boundary conditions (precipitations and surface air temperature) derived from the atmospheric general circulation model LMDZ, using a standard, large model grid of resolution 40 km × 40 km centered over the South Pole. [Ladant et al. (2014)](https://agupubs.onlinelibrary.wiley.com/doi/full/10.1002/2013PA002593) developed an asynchronous coupling method permitting to account for the feedbacks of the ice sheet on global climate (also used by [Pohl et al. (2016)](https://agupubs.onlinelibrary.wiley.com/doi/full/10.1002/2016PA002928)); this more sophisticated setup is not described here. The documentation may be updated later upon request. The experimental setup desribed in this tutorial hence permits to investigate glacial inception a-la [Ladant et al. (2016)](https://www.nature.com/articles/ncomms12771) but cannot be used to quantify ice-sheet-climate equilibria (For a comparison, compare Figs. 2 and 3 of [Pohl et al. (2016)](https://agupubs.onlinelibrary.wiley.com/doi/full/10.1002/2016PA002928)).

Please note that the GRISLI source code is not freely available.

# Requirements

Before starting anything, you need to correctly configure your environment by loading the appropriate modules:

```bash
module purge
module load netcdf/4.5.0/intel/2018
module load intel/2018
module load cdo
```

For comparison, here should be the result of the `module list` command:

```bash
Currently Loaded Modulefiles:
 1) netcdf/4.5.0/intel/2018   2) intel/2018   3) cdo/1.9.2/intel/2018
```

Remark: At some point, following cluster module updates, we will probably have to change some modules. Ideally, we'll also have to check that results are identical.

# Installing and compiling

1. Download the model source code from [here](/assets/data/GRISLI-version8-svn.zip). Then, uncompress the model source `tar xfvz GRISLI-version8-svn.zip` on your work `/work/crct/zz9999zz` (replace `zz9999zz` with your CCUB login). As for now, the model source code is not freely available; this is why it is password-protected. Then, enter the directory `cd GRISLI-version8-svn`.

2. On the CCUB cluster, you should be all set with libraries correctly linked in the `Makefile` in the `SOURCES` directory on lines 25–53:

```fortran
NCDF_INC  = /soft/c7/netcdf/4.5.0/intel/2018/include
NCDF_LIB  = -L/soft/c7/netcdf/4.5.0/intel/2018/lib -lnetcdff -L/soft/c7/netcdf/4.5.0/intel/2018/lib -lnetcdf -lnetcdf
MKL_LIB = -L${MKLROOT}/lib/intel64 -lmkl_intel_lp64 -lmkl_sequential -lmkl_core -liomp5 -lpthread
IFORT = ifort
```

{:start="3"}
3. `Make clean` to clean compiling directory from previous iterations.

4. Compile: `Make Maassud`. Everything going well, you should get an executable called `Maassud` in directory `GRISLI-version8-svn/bin`. Now, the model is compiled and ready to run. Let's see how to launch an experiment from a directory with all boundary conditions ready. We'll see how to create these boundary conditions later.

At this point, you are probably wondering: why `Maassud`? To avoid making you wake at night wondering about this, here is an explanation. This is some heritage from Christophe Dumas' naming, when Christophe designed a model grid to study the Maastrichtian southern hemisphere.

# Running an experiment from existing boundary conditions

## Directory with boundary conditions and initial conditions

The model boundary conditions live in directory `GRISLI-version8-svn/INPUT/MAASSUD`. They consist in 3 types of files. For illustration purposes, we will use an Ordovician setup for 440 Ma in the following, derived from a LDMZ simulations conducted at 1680 ppm pCO2 under a cold summer orbital configuration:
- `t2m_440tcCSO1680_hemisud.ijz` is a file providing GRISLI with monthly surface air temperature data interpolated onto the (40 km × 40 km) model grid.
- `precip_440tcCSO1680_hemisud.ijz` is the same but for precipitations.
- `topo_440tc_hemisud.dat`is the file providing the topography and bathymetry.

These 3 files are provided with the model source code and will be used below to run a GRISLI simulation. They are associated with the study by [Marcilly et al. (2022)](https://www.sciencedirect.com/science/article/pii/S0012821X22003533), although we eventually did not use the GRISLI results in the paper.

While the climatic boundary conditions will generally be specific to one experiment, the topo-bathy boundary conditions may be used for several GRISLI runs, for instance at various pCO2 levels and/or orbital configurations with the same paleogeographical configuration.

## Run directory / launching a simulation

In directory `GRISLI-version8-svn/RESULTATS`, you will find a run directory named `440tc_CSO_1680` that is almost ready to run.

1. Copy the executable that you previously compiled: `cp ../../bin/Maassud .`.

2. In file `Job`, replace `zz9999zz` with your login.

3. Submit the simulation in batch mode: `qsub Job`.

4. Check that the simulation is running with the command `qstat`: you should see a job named `j440CO06X` running. In the run directory, files like `440CO06X-k0000.hz` should rapidly appeaer. Well done, the model is running and generating some output !

### Content of file `Job`

File `Job` (content copied below for your convenience) is a bash script that provides all necessary instructions to launch the simulation in batch mode. It requests to launch the simulation on 1 processor under a specific run name (`j440CO06X`) that will appear when executign the command `qstat`. It load the modules needed, defines the path to the run directory as well as the file used for the model standard output, and finally requests to run the executable `Maassud` and to compress the output when the simulation finishes.

```bash
#!/bin/bash
#$ -N j440CO06X
#$ -pe dmp* 1
#$ -q batch

module purge
module load netcdf/4.5.0/intel/2018
module load intel/2018

# donne le chemin du repertoire des resultats du run "Dir_result"
Dir_result='440tc_CSO_1680'

# Chemin d'acces au repertoire des resultats :
Dir_exec='/work/crct/al1966po/GRISLI/GRISLI-version8-svn/RESULTATS/'

Execution=${Dir_exec}${Dir_result}
fileout=${Dir_result}.out

cd ${Execution}
time ./Maassud >& ${fileout}

# ziper les gros fichiers :
gzip -f *.hz
```

### Content of file `maassud_param_list.dat`

File `maassud_param_list.dat` defines an ensemble of parameter values defining the simulation. It is relatively lengthy and we will focus on important parts below. Other parameters should be kept unchanged in most cases.

```bash
!___________________________________________________________
&runpar                  ! nom du bloc  parametres du run

 runname      =  "440CO06X"        ! 8 caracteres
 comment_run  = "440tc CSO oneway 1680 ppm"
```
The block above defines a run name (440CO06X``) that will be used to name the output files. Line `comment_run` allows the user to provide all info that may be necessary to keep track of what was done. This is basically a free space that can be used to come back to the file later and have an overview/reminder of the simulation design.

```bash
!___________________________________________________________
&topo_file
 file1        =  "topo_440tc_hemisud.dat"    ! "topo-21k.g40"
 file2        =  "topo_440tc_hemisud.dat"    !  hemin2.g40
!___________________________________________________________
```

The block above is used to provide the topo-bathy filename (repeated).

```bash
!___________________________________________________________
&timesteps                ! bloc timestep

 tend      =   300000.
!___________________________________________________________
```

The duration of the simulation, expressed in years. 300 kyrs should be enough to reach ice-sheet equilibrium.
Please note that this long duration is the reason why coupled climate-ice-sheet models are difficult to run: GCMs are usually run over a few weeks to months so simulate 2000 years; running a 300-kyr simulation would take years / decades ! Hence also the asynchronous coupling methods developed in paleo applications.

```bash
!___________________________________________________________
&snap_forcage_mois                                   ! module climat_forcage_mois_mod
filtr_t1 = "t2m_440tcCSO1680_hemisud.ijz"
filtr_p1 = "precip_440tcCSO1680_hemisud.ijz"
!___________________________________________________________
```
Above are provided the climatic boundary conditions.

The parameters listed in this section are the ones that you will need to adapt to setup your own simulations.

# Generating boundary conditions

















# Checking for equilibrium and generating output files

## Checking for equilibrium

After 2000 model years of simulation, you will be interested in checking for deep-ocean thermal equilibrium. To that purpose:

1. Generate time series by copying file `UTIL/EvolBis.py` into your `history/ocean` model output directory and running it with `python EvolBis.py`. It will take a while and output a file named `history.ocean.evol.an001a2000.nc`.

2. Plot the time-evolution of temperature over the water column using the script `UTIL/checkevol.py` (which looks for the file that you previously created, `history.ocean.evol.an001a2000.nc`).

You will get a figure like the one below, also calculating the deep-ocean temperature drift over the last 100 years. The latter should be very small.

<img src="/assets/img/FOAM_checkevol.png" alt="Slarti Screenshot" class="center">

If equilibrium has not been reached after 2000 model years, you have several options:

1. Give up and go and play outside.

2. Restart the simulation for another, say, 1000 years. To do that, just edit file `run_params`. `FILTPHIS` and `INITIAL` should be set to `F`, `FINISHED` to 2000 years and `RUNLNG` to 1000 additional years:

```fortran
RESTFRQ: 360
HISTFRQ: 360
FILTPHIS: F
INITIAL: F
PREFIX: /work/crct/zz9999zz/foam/phanero/300rd/300rd_T36/EcN_8X
STORAGE: /work/crct/zz9999zz/foam/phanero/300rd/300rd_T36/EcN_8X
TIME_INV: /work/crct/zz9999zz/foam/phanero/300rd/300rd_T36/BC_300rd_T37
FINISHED: 720000
RUNLNG: 360000
```

{:start="3"}
3. If you think the issue comes from inadequate initial conditions (e.g., too warm an initial ocean), you can use this knowledge to run a new simulation with better-suited initial conditions (just change the `om3.temp24` file and start the simulation from scratch; previous output files with be overwritten).

__With the slab model, you can check for equilibrium by running `UTIL/EvolSlab.py` in your `history/atmos` model output folder and subsequently using the output time-series NetCDF file to plot the time-evolution of atmospheric surface temperature `TS1` (but it's not even necessary, the slab model will reach equilibrium in 50 years).__

## Generating output files

As any GCM, FOAM displays some internal variability. In order to get 'climatology' files, we will have to calculate mean monthly values over the last 30–50 years – personally and below, I use 50 years. It will require 2 steps:

### Running the model with monthly output

Edit file `run_params` (or create a new one, named for instance `run2`) as follows:

```fortran
RESTFRQ: 360
HISTFRQ: 30
FILTPHIS: F
INITIAL: F
PREFIX: /work/crct/zz9999zz/foam/phanero/300rd/300rd_T36/EcN_8X
STORAGE: /work/crct/zz9999zz/foam/phanero/300rd/300rd_T36/EcN_8X
TIME_INV: /work/crct/zz9999zz/foam/phanero/300rd/300rd_T36/BC_300rd_T37
FINISHED: 720000
RUNLNG: 18000
```

- `HISTFRQ`: set to 30 for monthly output (every 30 days).
- `FILTPHIS` and `INITIAL`: set to `F` for a restart.
- `FINISHED`: restarting from year 2000. 
- `RUNLNG`: 18 000 days (50 years of 360 days each).

Then, submit `pbs.foam16p.script` (if you created a new `run2` file, you have to make sure that `pbs.foam16p.script`, or a newly-created `pbs2` file, calls the right parameters, here `run2`). It will take around 4 hours to run for 50 years.

Important remark: With the CCUB, some `restart/atmos` files are not named properly. Check for any issue before restarting an experiment. Indeed, each atmospheric restart should consist in two files (here for year 2000) `restart.atmos.0720000.A` and `restart.atmos.0720000`. Typically, `restart.atmos.0720000` will be incorrectly named `r2001`, or similar. Just `mv r2001 restart.atmos.0720000`, or `ln -s r2001 restart.atmos.0720000`.

__With the slab model, you previously ran the model for 100 years with monthly output. No need to run an additional job, then. Climatology will be calculated over the last 50 years, see below.__

### Generating climatology files

We generate climatology files for each model component: ocean, atmosphere and coupler, by calculating monthly means over the last 50 model years.

1. Once the monthly output generated, copy the python script `UTIL/NewMois.py` into your ocean output directory (`cp UTIL/NewMois.py /work/crct/zz9999zz/foam/phanero/300rd/300rd_T36/EcN_8X/history/ocean`), check that everything is OK (see code and explanations below) and run the script (`python NewMois.py`).

    ```python
    prefix = 'history.ocean.'
    year = 2000
    nyears = 50
    endname ='300rd_1368W_EccN_ocean_2240ppm.nc'
    ```

    - `prefix`: to be adjusted, `ocean`, `atmos` or `coupl`.
    - `year`: duration of the spinup (long run with annual output), here 2000 years.
    - `nyears`: duration of the short run with monthly output, here 50 years.
    - `endname`: name of the output climatology file; to be adjusted, including `ocean`, `atmos` or `coupl`.

{:start="2"}
2. Then, `cp NewMois.py ../atmos/.`, `cd ../atmos/.` and edit `NewMois.py`: `prefix = 'history.atmos.'` and `endname ='300rd_1368W_EccN_atmos_2240ppm.nc'`. Run the script.

3. Do the same for the coupler (`cp NewMois.py ../coupl/.` etc.).

As a result, the output of each FOAM experiment is presented in the form of 3 files, each associated with a given model component, and each containing specific variables (oceanic variables for the ocean output file etc.): 

- `300rd_1368W_EccN_ocean_2240ppm.nc`;
- `300rd_1368W_EccN_atmos_2240ppm.nc`;
- `300rd_1368W_EccN_coupl_2240ppm.nc`.

__With the slab model, use `NewMoisSlab.py` and skip step #1 (there is no oceanic output). As a result, the output of each FOAM-slab experiment is presented in the form of only 2 (instead of 3) files.__ 

# Debugging

Different cases:

## Nothing bad happened

Sometimes, the model just crashes without known reason, probably due to issues with the cluster. In that case, just check that `restart/atmos` files are well named (see [Known Issues below](https://alexpohl.github.io/FOAM_howto/#restart-files)) and restart the experiment (see e.g. [this previous section](https://alexpohl.github.io/FOAM_howto/#running-the-model-with-monthly-output) for how to restart an experiment).

## Timestep issue

Sometimes, the model crashes and an error like `point IJ out of bounds` shows up in the error files (`om3.out.4_8.0` or `pccm.out.0720000`, I can't remember). It means that you're not respecting the CFL criteria. In short, your model particle traveled over a distance longer than a grid point during the model integration time (time step).

In order to solve this issue, you can temporarily decrease the time step. In `atmos_params`, change the following:
- `DTIME  =  1200.,` (instead of 1800);
- `DIF4=1.E16,` (instead of 2.E16).

Run the model for a few time steps and then revert back to the standard values.

This workaround is useful in the presence of a sea-ice front located at the mid latitudes (which generates very strong temperature contrasts and winds/currents) or when simulating a very, very warm climate.

## Salt anomaly

Another usual issue, which will probably show up in your first fully coupled model simulations due to small mistakes in Slarti.

An isolated ocean point in the land-sea mask, or a deep ocean point surrounded by shallower ocean grid points, or a very flat and relatively shallow bathymetry at the tropics (with negative P-E balance) or at the high latitudes (with sea-ice formation and associated salt fluxes), will lead to the excessive accumulation of salt and make the model crash.

This issue can generally be spotted by plotting the maximum salinity over the whole water column and determining where salinity exceeds reasonable values, and corrected by going back to Slarti, correcting the boundary conditions locally, and running a new simulations using these corrected boundary conditions.

# Known issues

## Restart files

With the CCUB, some `restart/atmos` files are not named properly, which makes the model crash upon restart.

Check for any issue before restarting an experiment. Indeed, each atmospheric restart should consist in two files (here for year 2000) `restart.atmos.0720000.A` and `restart.atmos.0720000`. Typically, `restart.atmos.0720000` will be incorrectly named `r2001`, or similar. Just `mv r2001 restart.atmos.0720000`, or `ln -s r2001 restart.atmos.0720000`.

## Too long a name

Too long a path (e.g., `/work/crct/zz9999zz/foammmmmmmmm/phanero/300rd/300rd_T36/EcN_8X`) will make the model crash with no explicit error message. It even happened that a given path name made the model crash... for unknown reasons except this given name?

# Looking at the model output

The FOAM model output consists in standard, self-describing and cross-platform NetCDF files. You can use various tools to open such files and generate figures and diagnostics. Here is a short selection:

1. The legacy software used to treat the FOAM output is the very convenient and simple scripting language [Ferret](https://ferret.pmel.noaa.gov/Ferret/), which can be installed on Linux clusters or even locally, including on MacOS. PyFerret is just an upgrade but remains largely similar.

2. A more complex but way more powerful tool is Python.

3. You can also think about R (free), Matlab (expensive) and others (Scilab, NCL etc.).

4. If you're looking for a graphical interface, [Panoply](https://www.giss.nasa.gov/tools/panoply/) could do the job.

5. A more complex but more powerful tool with graphical interface is a GIS, such as QGIS (free) or ArcGIS (very expensive).

# References using FOAM

Here is a (non-exhaustive) list of references illustrating the use of FOAM:

- Chaboureau, A.C., Donnadieu, Y., Sepulchre, P., Robin, C., Guillocheau, F. and Rohais, S. 2012. The Aptian evaporites of the South Atlantic: A climatic paradox? Climate of the Past, 8, 1047–1058, https://doi.org/10.5194/cp-8-1047-2012.
- Dal Corso, J., Mills, B.J.W., Chu, D., Newton, R.J. and Song, H. 2022. Background Earth system state amplified Carnian (Late Triassic) environmental changes. Earth and Planetary Science Letters, 578, 117321, https://doi.org/10.1016/j.epsl.2021.117321.
- Donnadieu, Y., Pucéat, E., Moiroud, M., Guillocheau, F. and Deconinck, J.F. 2016. A better-ventilated ocean triggered by Late Cretaceous changes in continental configuration. Nature Communications, 7, 10316, https://doi.org/10.1038/ncomms10316.
- Goddéris, Y., Donnadieu, Y., Le Hir, G., Lefebvre, V. and Nardin, E. 2014. The role of palaeogeography in the Phanerozoic history of atmospheric CO2 and climate. Earth-Science Reviews, 128, 122–138, https://doi.org/10.1016/j.earscirev.2013.11.004.
- Hearing, T.W., Harvey, T.H.P., et al. 2018. An early Cambrian greenhouse climate. Science Advances, 4, eaar5690, https://doi.org/10.1126/sciadv.aar5690.
- Ladant, J.-B. and Donnadieu, Y. 2016. Palaeogeographic regulation of glacial events during the Cretaceous supergreenhouse. Nature communications, 7, 12771.
- Le Hir, G., Donnadieu, Y., et al. 2009. The snowball Earth aftermath: Exploring the limits of continental weathering processes. Earth and Planetary Science Letters, 277, 453–463, https://doi.org/10.1016/j.epsl.2008.11.010.
- Le Hir, G., Donnadieu, Y., Goddéris, Y., Meyer-Berthaud, B., Ramstein, G. and Blakey, R.C. 2011. The climate change caused by the land plant invasion in the Devonian. Earth and Planetary Science Letters, 310, 203–212, https://doi.org/10.1016/j.epsl.2011.08.042.
- Lee, S.-Y. and Poulsen, C.J. 2006. Sea ice control of Plio–Pleistocene tropical Pacific climate evolution. Earth and Planetary Science Letters, 248, 253–262.
- Licht, A., van Cappelle, M., et al. 2014. Asian monsoons in a late Eocene greenhouse world. Nature, 513, 501–506.
- ills, B.J.W., Donnadieu, Y. and Goddéris, Y. 2021. Spatial continuous integration of Phanerozoic global biogeochemistry and climate. Gondwana Research, https://doi.org/10.1016/j.gr.2021.02.011.
- Nardin, E., Goddéris, Y., Donnadieu, Y., Le Hir, G., Blakey, R.C., Pucéat, E. and Aretz, M. 2011. Modeling the early Paleozoic long-term climatic trend. Bulletin of the Geological Society of America, 123, 1181–1192, https://doi.org/10.1130/B30364.1.
- Pohl, A., Donnadieu, Y., Le Hir, G., Buoncristiani, J.F. and Vennin, E. 2014. Effect of the Ordovician paleogeography on the (in)stability of the climate. Climate of the Past, 10, 2053–2066.
- Pohl, A., Donnadieu, Y., et al. 2020. Carbonate platform production during the Cretaceous. GSA Bulletin, 132, 2606–2610, https://doi.org/10.1130/B35680.1.
- Pohl, A., Lu, Z., et al. 2021. Vertical decoupling in Late Ordovician anoxia due to reorganization of ocean circulation. Nature Geoscience, 14, https://doi.org/10.1038/s41561-021-00843-9.
- Poulsen, C.J., Pierrehumbert, R.T. and Jacob, R.L. 2001. Impact of ocean dynamics on the simulation of the neoprotozoic ‘snowball earth’. Geophysical Research Letters, 28, 1575–1578, https://doi.org/10.1029/2000GL012058.
- Poulsen, C.J., Gendaszek, A.S. and Jacob, R.L. 2003. Did the rifting of the Atlantic Ocean cause the Cretaceous thermal maximum? Geology, 31, 115–118.
- Saupe, E.., Qiao, H.., et al. 2019. Extinction intensity during Ordovician and Cenozoic glaciations explained by cooling and palaeogeography. Nature Geoscience, 13, 65–70, https://doi.org/10.1038/s41561-019-0504-6.
- Wong Hearing, T.W., Pohl, A., et al. 2021. Quantitative comparison of geological data and model simulations constrains early Cambrian geography and climate. Nature Communications, 12, 3868, https://doi.org/10.1038/s41467-021-24141-5.
- Zacaï, A., Monnet, C., Pohl, A., Beaugrand, G., Mullins, G., Kroeck, D.M. and Servais, T. 2021. Truncated bimodal latitudinal diversity gradient in early Paleozoic phytoplankton. Science Advances, 7, eabd6709, https://doi.org/10.1126/sciadv.abd6709.


