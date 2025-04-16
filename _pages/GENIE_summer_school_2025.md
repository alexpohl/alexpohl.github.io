---
layout: smallfont_page
permalink: /GENIE_summer_school_2025/
title: cGENIE summer school 2025
description: Instructions to connect to the CCUB cluster and get GENIE running
nav: false
---
## Connecting to the cluster of Univ. Bourgogne Europe with NoMachine

NoMachine permits working on a remote Linux cluster with a graphical interface by generating a Desktop just like on your local computer. It notably permits opening a file explorer and easily displaying figures or even working interactively on Matlab, Spyder (Python) or Rstudio. 

- Open the `NoMachine` app.
- Use your CCUB login and password to connect.
- You are ready to open a terminal window to work (Menu > Emulateur de terminal); would you be given a choice, choose the terminal called Rxfce, which is black by default.
- When needed, you can use the local `FileZilla` application to transfer files between the local computer and the remote cluster using a graphical interface.

__Remark:__ it is also possible to just connect without graphical interface through ssh: 
```
ssh -Y -C -o StrictHostKeyChecking=no CCUBlogin@krenek2002.u-bourgogne.fr
```
… and to transfer files between the cluster and your local computer through scp.

<p>&nbsp;</p>

## Downloading and installing cGENIE

### Go to the workdir (this is where the model is going to run)

`cd /work/crct/CCUBlogin` (with CCUBlogin, your CCUB login)

### Downloading model source code

`git clone https://github.com/derpycode/cgenie.muffin.git`

### Installing (by adapting to CCUB)

1. `cd cgenie.muffin/`
2. `git checkout UBE25`
3. `chmod 744 install_on_CCUB.sh`
4. Adapt file `install_on_CCUB.sh` (first line) with your own CCUB login 
5. `./install_on_CCUB.sh`

### Sourcing the .kshrc file

Before moving forward, we’ll need to close the terminal and open a new one. (This permits sourcing the `.kshrc` file that we just created.)

### Checking everything works well

After opening a new terminal window:
1. `cd /work/crct/CCUBlogin/cgenie.muffin/genie-main`
2. `make cleanall`
3. `geniemod`
4. `make testbiogem`(that should not output any error, and give `**TEST OK**` as a result).

### Running a simulation

Running GENIE consists in (creating the appropriate files) and launching an instruction from `genie-main` (`cd /work/crct/CCUBlogin/cgenie.muffin/genie-main`).

An instruction is in the form:
```
./runmuffin.sh base-config directory user-config duration
```
with:
- `base-config`: the name of the base-config file, which contains the basic information regarding the model simulation, such as paleogeography, model components, biogeochemical tracers etc.
- `directory`: the name of the subdirectory containing the user-config file.
- `user-config`: the name of the user-config file, which contains additional model parameters.
- `duration`: the run duration in years.

As an example of running GENIE:
```
./runmuffin.sh cgenie.eb_go_gs_ac_bg.worjh2.BASEFePRE14Crbcolx PUBS/submitted/Tetard_et_al.Nature.2023 SPIN.worjh2.Fe14C.preAge.Dye.pO2 20000
```
(don’t forget to run the commands `make cleanall` and `geniemod` like we did above before launching the simulation)

The model executable should compile, the simulation should start and you should see the model run. Because the model takes hours to days to run, we do not run it interactively like this but instead launch the simulations in batch mode. To this end, first cancel the ongoing model execution by using `Ctrl+C`, then use the following command:
```
qsub -m n -N test -q batch@bartok* -pe dmp* 1 -j y -o /work/crct/CCUBlogin/cgenie_log -V -S /bin/bash ./runlmuffin.sh… [see instruction above]
```

Note that, as a baseline, you will have to run each new simulation interactively for a few years (`./runmuffin.sh...`) before being able to submit it in batch model (`qsub...`). This is because the interactive step permits compiling the code and creating an executable that is required for the model to run in batch mode.

You can check that a simulation is running: the `qstat` command should show the name of the simulation, and you should be able to access the output (`cd /work/crct/CCUBlogin/cgenie_output/SPIN.worjh2.Fe14C.preAge.Dye.pO2`)

### Looking at the output

The GENIE output of interest is located in directory biogem. It consists in 2D and 3D NetCDF files (`fields_biogem_2d.nc` and `fields_biogem_3d.nc`) and time-series (`*.res` ascii files). They can be plotted using diverse software such as Python; for the purpose of this summer school, you are free to use what you feel the most at ease with but support will only be provided for `gnuplot` and `pyFerret`. Basic gnuplot and pyFerret scripts are provided in directory `cgenie.muffin/basic_scripts`. The gnuplot script permits plotting the .res time series. The pyFerret .jnl script can be used to explore the NetCDF files. To run these scripts, simply adapt them and run the commands `gnuplot plot_res_file.gnuplot` in the terminal, and `go plot.jnl` using the pyFerret prompt, respectively. To launch pyFerret, just run the following commands: (i) `pyferretmod` (which permits loading the right modules), and (ii) `pyferret` (which starts pyFerret).

It is also possible to download the GENIE NetCDF output and look at it using Panoply on your local computer.

### GENIE output example

Example GENIE output files can be downloaded using the following command line:

```
wget "https://sdrive.cnrs.fr/s/y7kCN9562j8RGA3/download/GENIE_results_example.tar.gz"
```

First uncompress the archive:
```
tar xfvz GENIE_results_example.tar.gz
```

This will give you a directory `GENIE_results_example` containing three items:
- `AP.muffin.CB.GIteiiaa.BASESFeTDTL_rb.SPIN` is a GENIE (spinup) preindustrial simulation that was run for 20 000 years.
- `AP.muffin.CB.GIteiiva.BASESFeTDTL_rb_DO0.0.SPIN` is a GENIE (spinup) simulation for the Last Glacial Maximum that was also run for 20 000 years.
- `1_woa23_all_o00_01_regridded_GIteiiaa.nc` is the modern oceanic dissolved oxygen concentration from the World Ocean Atlas 2023 regridded to the GENIE grid.

## Material for case studies

User-config files made available for the case studies are located in directory `cgenie.muffin/genie-userconfigs/UBgraduateprogramme`.








