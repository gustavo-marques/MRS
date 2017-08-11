## Scripts used in testing MOM6 commits against the various regression repositories

- These scripts are not for general consumption - they are mostly written in gmake so good luck figuring out how they work! :)
  But seriously they are written for a particular sequence of tests assuming a specific setup and so they really will not be much use for general application.


## Initial setup

Either recursively clone the regression repository and add this repository to it with:
```bash
git clone --recursive git@gitlab.gfdl.noaa.gov:ogrp/Gaea_c3-stats-MOM6-examples.git
cd Gaea_c3-stats-MOM6-examples
git clone https://github.com/adcroft/MRS.git
```

or clone then manually update:
```bash
git clone git@gitlab.gfdl.noaa.gov:ogrp/Gaea_c3-stats-MOM6-examples.git
cd Gaea_c3-stats-MOM6-examples
git clone https://github.com/adcroft/MRS.git
make -f MRS/Makefile.clone clone
```
or
```bash
git submodule init
git submodule update --recursive 
(cd MOM6-examples; git submodule init; git submodule update)
(cd MOM6-examples/src/MOM6; git submodule init; git submodule update)
```

## Clone additional components from internal gitlab server

```bash
make -f MRS/Makefile.clone clone_gfdl
```

## Build executable

```bash
make -f MRS/Makefile.build build_gnu -s -j
make -f MRS/Makefile.build build_intel -s -j
make -f MRS/Makefile.build build_pgi -s -j
```

## Run model

```bash
make -f MRS/Makefile.run gnu_all MEMORY=dynamic_symmetric -s -j
make -f MRS/Makefile.run intel_all MEMORY=dynamic_symmetric -s -j
make -f MRS/Makefile.run pgi_all MEMORY=dynamic_symmetric -s -j
```
should yield a clean MOM6-examples (uses the correct layouts).

Test the non-symmetric executables
```bash
make -f MRS/Makefile.run gnu_all -s -j
make -f MRS/Makefile.run intel_all -s -j
make -f MRS/Makefile.run pgi_all -s -j
```
will produce difference MOM_parameter_doc.layout files in MOM6-examples but with the right answers.

or 
```bash
make -f MRS/Makefile.run gnu_static_ocean_only MEMORY=static -s -j
make -f MRS/Makefile.run intel_static_ocean_only MEMORY=static -s -j
make -f MRS/Makefile.run pgi_static_ocean_only MEMORY=static -s -j
```
Test with alternative PE counts
```bash
make -f MRS/Makefile.run gnu_all -s -j LAYOUT=alt
make -f MRS/Makefile.run intel_all -s -j LAYOUT=alt
make -f MRS/Makefile.run pgi_all -s -j LAYOUT=alt
```

## Copy results to regressions/
```bash
make -f MRS/Makefile.sync -s -k
```
will sync the newly generated ocean/seaice.stats files and report their status.

```bash
make -f MRS/Makefile.sync -s -k gnu
```
will sync only the gnu stats files.


## Test restarts

```bash
make -f MRS/Makefile.restart gnu_ocean_only -s -j RESTART_STAGE=02
make -f MRS/Makefile.restart gnu_ocean_only -s -j RESTART_STAGE=01
make -f MRS/Makefile.restart gnu_ocean_only -s -j RESTART_STAGE=12
make -f MRS/Makefile.restart gnu_ice_ocean_SIS2 -s -j RESTART_STAGE=02
make -f MRS/Makefile.restart gnu_ice_ocean_SIS2 -s -j RESTART_STAGE=01
make -f MRS/Makefile.restart gnu_ice_ocean_SIS2 -s -j RESTART_STAGE=12
make -f MRS/Makefile.restart restart_gnu_ocean_only -s -j
make -f MRS/Makefile.restart restart_gnu_ice_ocean_SIS2 -s -j
```
Last commands alone is sufficient but seems more susceptible to lustre file systems flakiness.
