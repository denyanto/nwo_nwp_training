# WRFv-4.4.1 Configure on CentOS 8
Configuration of WRF 4.4.1 on CenOS
These are some installation notes taken in the process of installing WRF version 4.4.1 on a computer with CentOS 8. This tutorial is for **Group Fellowship Training Course on Numerical Weather Prediction (NWP) 26 September - 25 October 2022**


## Install required software


```console
$ sudo yum groupinstall 'Development Tools'
$ sudo yum install csh gfortran m4
```

## System environment tests

First and foremost, it is very important to have a gfortran compiler, as well as gcc and cpp. If you have these installed, you should be given a path for the location of each.
```console
$ which gfortran
/user/bin/gfortran
$ which cpp
/user/bin/cpp
$ which gcc
/user/bin/gcc
```

Check your gcc version. It is recommend using version 4.4.0 or later.
```console
$ gcc --version
gcc (GCC) 8.5.0 20210514 (Red Hat 8.5.0-10)
Copyright (C) 2018 Free Software Foundation, Inc.
This is free software; see the source for copying conditions.  There is NO
warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.
```

## Build Librarties and dependences

**We need to set where to find sources for libraries**
```console

$ mkdir -p $HOME/libraries
$ mkdir -p $HOME/source

```

**First, whe need MPI libraries to compile all**
```console
$ cd $HOME/source
$ sudo yum install openmpi-devel
$ wget -nc https://download.open-mpi.org/release/open-mpi/v4.1/openmpi-4.1.4.tar.gz
$ tar -xvzf openmpi-4.1.4.tar.gz
$ ./configure --prefix=/home/<your-user-name>/libraries
$ make 
$ make install
$ cd ..
$ PATH=/home/<your-user-name>/libraries/bin:$PATH
$ export PATH
$ mpicc --version
gcc (GCC) 8.5.0 20210514 (Red Hat 8.5.0-10)
Copyright (C) 2018 Free Software Foundation, Inc.
This is free software; see the source for copying conditions.  There is NO
warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.

```

**ZLib install**
Configuring zlib: This is a compression library necessary for compiling WPS (specifically ungrib) with GRIB2 capability.
Assuming all the **export** commands from the NetCDF install are already set, you can move on to the commands to install zlib.

```console
$ wget -nc https://zlib.net/fossils/zlib-1.2.8.tar.gz
$ tar -xvzf zlib-1.2.8.tar.gz
$ cd zlib-1.2.8
$ ./configure --prefix=/home/<your-user-name>/libraries
$ make
$ make install
$ cd ..
```

**HDF5 parallel install**
```console
$ wget -nc https://support.hdfgroup.org/ftp/HDF5/releases/hdf5-1.12/hdf5-1.12.0/src/hdf5-1.12.0.tar.gz
$ tar -xvzf hdf5-1.12.0.tar.gz
$ cd hdf5-1.12.0
$ export FC=mpifort
$ export CC=mpicc
$ export CXX=mpicc
$ ./configure --prefix=/home/<your-user-name>/libraries --enable-parallel --with-zlib=/home/<your-user-name>/libraries --enable-fortran --enable-shared
$ make
$ make check
$ make install
$ cd ..
```

**netCDF (C library)**
```console
$ wget -nc ftp://ftp.unidata.ucar.edu/pub/netcdf/netcdf-c-4.7.4.tar.gz
$ tar -xvzf netcdf-c-4.7.4.tar.gz
$ cd netcdf-c-4.7.4
$ CC=mpicc CPPFLAGS="-I/home/<your-user-name>/libraries/include" LDFLAGS="-L/home/<your-user-name>/libraries/lib" ./configure --enable-shared --enable-parallel-tests --enable-netcdf4 --disable-filter-testing --disable-dap --prefix=/home/<your-user-name>/libraries
$ make 
$ make install
$ cd ..
```

**netCDF (Fortran interface library)**

```console
$ wget -nc ftp://ftp.unidata.ucar.edu/pub/netcdf/netcdf-fortran-4.5.3.tar.gz
$ tar -xvzf netcdf-fortran-4.5.3.tar.gz
$ cd netcdf-fortran-4.5.3
$ export FC=mpifort
$ export F77=mpif77
$ CPPFLAGS="-I/home/<your-user-name>/libraries/include" LDFLAGS="-L/home/<your-user-name>/libraries/lib" LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/home/<your-user-name>/libraries/lib LIBS="-lnetcdf -lhdf5_hl -lhdf5 -lz" ./configure --enable-parallel-tests --enable-shared --prefix=/home/<your-user-name>/libraries
$ make 
$ make install
$ cd ..
```


**JASPER**
Configuring JasPer: This is a compression library necessary for compiling WPS (specifically ungrib) with GRIB2 capability.
Assuming all the **export** commands from the NetCDF install are already set, you can move on to the commands to install jasper.

```console
$ wget -nc http://www2.mmm.ucar.edu/wrf/OnLineTutorial/compile_tutorial/tar_files/jasper-1.900.1.tar.gz
$ tar -xvzf jasper-1.900.1.tar.gz
$ cd jasper-1.900.1
$ ./configure --prefix=/home/<your-user-name>/libraries
$ make
$ make install
$ cd ..
$ export JASPERLIB=/home/<your-user-name>/libraries/lib
$ export JASPERINC=/home/<your-user-name>/libraries/include
```

**LIBPNG**
This is a compression library necessary for compiling WPS (specifically ungrib) with GRIB2 capability.
Assuming all the **export** commands from the NetCDF install are already set, you can move on to the commands to install libpng.
```console
$ wget -nc http://sourceforge.net/projects/libpng/files/libpng16/1.6.37/libpng-1.6.37.tar.gz
$ tar xvzf libpng-1.6.37.tar.gz 
$ cd libpng-1.6.37/
$ ./configure --prefix=/home/<your-user-name>/libraries
$ make
$ make install
$ cd 
```


## Building WRFV4

First of all, we need to download the source code from
```console
$ git clone https://github.com/wrf-model/WRF
$ cd WRF/
```

Now we are able to run the confgure 

```console
$ export NETCDF=/home/<your-user-name>/libraries
$ ./configure
```
You will see various options. Choose the option that lists the compiler you are using and the way you wish to build WRF (i.e., serially or in parallel). Although there are 3 different types of parallel (smpar, dmpar, and dm+sm), it is recommend choosing dmpar option.

```console
checking for perl5... no
checking for perl... found /usr/bin/perl (perl)
Will use NETCDF in dir: /home/wrf/wrf_io
HDF5 not set in environment. Will configure WRF for use without.
Will use PHDF5 in dir: /home/wrf/wrf_io
Will use 'time' to report timing information
Configuring to use jasper library to build Grib2 I/O...
  $JASPERLIB = /home/wrf/wrf_io/lib
  $JASPERINC = /home/wrf/wrf_io/include
------------------------------------------------------------------------
Please select from among the following Linux x86_64 options:

  1. (serial)   2. (smpar)   3. (dmpar)   4. (dm+sm)   PGI (pgf90/gcc)
  5. (serial)   6. (smpar)   7. (dmpar)   8. (dm+sm)   PGI (pgf90/pgcc): SGI MPT
  9. (serial)  10. (smpar)  11. (dmpar)  12. (dm+sm)   PGI (pgf90/gcc): PGI accelerator
 13. (serial)  14. (smpar)  15. (dmpar)  16. (dm+sm)   INTEL (ifort/icc)
                                         17. (dm+sm)   INTEL (ifort/icc): Xeon Phi (MIC architecture)
 18. (serial)  19. (smpar)  20. (dmpar)  21. (dm+sm)   INTEL (ifort/icc): Xeon (SNB with AVX mods)
 22. (serial)  23. (smpar)  24. (dmpar)  25. (dm+sm)   INTEL (ifort/icc): SGI MPT
 26. (serial)  27. (smpar)  28. (dmpar)  29. (dm+sm)   INTEL (ifort/icc): IBM POE
 30. (serial)               31. (dmpar)                PATHSCALE (pathf90/pathcc)
 32. (serial)  33. (smpar)  34. (dmpar)  35. (dm+sm)   GNU (gfortran/gcc)
 36. (serial)  37. (smpar)  38. (dmpar)  39. (dm+sm)   IBM (xlf90_r/cc_r)
 40. (serial)  41. (smpar)  42. (dmpar)  43. (dm+sm)   PGI (ftn/gcc): Cray XC CLE
 44. (serial)  45. (smpar)  46. (dmpar)  47. (dm+sm)   CRAY CCE (ftn $(NOOMP)/cc): Cray XE and XC
 48. (serial)  49. (smpar)  50. (dmpar)  51. (dm+sm)   INTEL (ftn/icc): Cray XC
 52. (serial)  53. (smpar)  54. (dmpar)  55. (dm+sm)   PGI (pgf90/pgcc)
 56. (serial)  57. (smpar)  58. (dmpar)  59. (dm+sm)   PGI (pgf90/gcc): -f90=pgf90
 60. (serial)  61. (smpar)  62. (dmpar)  63. (dm+sm)   PGI (pgf90/pgcc): -f90=pgf90
 64. (serial)  65. (smpar)  66. (dmpar)  67. (dm+sm)   INTEL (ifort/icc): HSW/BDW
 68. (serial)  69. (smpar)  70. (dmpar)  71. (dm+sm)   INTEL (ifort/icc): KNL MIC
 72. (serial)  73. (smpar)  74. (dmpar)  75. (dm+sm)   FUJITSU (frtpx/fccpx): FX10/FX100 SPARC64 IXfx/Xlfx

Enter selection [1-75] : 34
------------------------------------------------------------------------
Compile for nesting? (1=basic, 2=preset moves, 3=vortex following) [default 1]: 1

Configuration successful! 
------------------------------------------------------------------------
testing for fseeko and fseeko64
fseeko64 is supported
------------------------------------------------------------------------
Settings listed above are written to configure.wrf.
If you wish to change settings, please edit that file.
If you wish to change the default options, edit the file:
     arch/configure.defaults
NetCDF users note:
 This installation of NetCDF supports large file support.  To DISABLE large file
 support in NetCDF, set the environment variable WRFIO_NCD_NO_LARGE_FILE_SUPPORT
 to 1 and run configure again. Set to any other value to avoid this message.
  

Testing for NetCDF, C and Fortran compiler

This installation of NetCDF is 64-bit
                 C compiler is 64-bit
           Fortran compiler is 64-bit
              It will build in 64-bit

*****************************************************************************
This build of WRF will use classic (non-compressed) NETCDF format
*****************************************************************************

```

Once your configuration is complete, you should have a `configure.wrf` file, and you are ready to compile. To compile WRFV4.4.1, you will need to decide which type of case you wish to compile. The options are listed below.
```console
em_real (3d real case)
em_quarter_ss (3d ideal case)
em_b_wave (3d ideal case)
em_les (3d ideal case)
em_heldsuarez (3d ideal case)
em_tropical_cyclone (3d ideal case)
em_hill2d_x (2d ideal case)
em_squall2d_x (2d ideal case)
em_squall2d_y (2d ideal case)
em_grav2d_x (2d ideal case)
em_seabreeze2d_x (2d ideal case)
em_scm_xy (1d ideal case)
```

For this purpose we are going to compile WRF for real cases. Compilation should take about 20-30 minutes. The ongoing compilation can be checked.
```console
$ ./compile em_real >& compile.log &
$ tail -f compile.log
```

If we see this message, you done it right ;)

```console
==========================================================================
build started:   lun nov 18 21:48:48 -03 2019
build completed: lun nov 18 21:56:41 -03 2019
 
--->                  Executables successfully built                  <---
 
-rwxrwxr-x. 1 wrf wrf 57432880 nov 18 21:56 main/ndown.exe
-rwxrwxr-x. 1 wrf wrf 57309864 nov 18 21:56 main/real.exe
-rwxrwxr-x. 1 wrf wrf 56831120 nov 18 21:56 main/tc.exe
-rwxrwxr-x. 1 wrf wrf 61189576 nov 18 21:56 main/wrf.exe
 
==========================================================================
```

Once the compilation completes, to check whether it was successful, you need to look for executables in the `WRFV3/main` directory.
```console
$ ls -las main/*.exe
ndown.exe (one-way nesting)
real.exe (real data initialization)
tc.exe (for tc bogusing--serial only)
wrf.exe (model executable)
```

These executables are linked to 2 different directories. You can choose to run WRF from either directory.
```console
WRF/run
WRF/test/em_real

$ export WRF_DIR=$HOME/WRF
```

Now we need to download and compile WPS

## Building WPS

NOTE: If you choosed in WRF to run on shared memory architecture (smpar), you need to add the flag of OpenMP (-lgomp) to the WRF_LIB variable in file configure.wps (just append it after -lnetcdf).

After the WRF model is built, the next step is building the WPS program (if you plan to run real cases, as opposed to idealized cases). The WRF model MUST be properly built prior to trying to build the WPS programs. If you do not already have the WPS source code, move to your `Build_WRF` directory, download that file and unpack it. Then go into the WPS directory and make sure the WPS directory is clean.

```console
$ git clonehttps://github.com/wrf-model/WPS
$ cd WPS
```

The next step is to configure WPS, however, you first need to set some paths for the ungrib libraries and then you can configure.
```console
./configure
```
You should be given a list of various options for compiler types, whether to compile in serial or parallel, and whether to compile ungrib with GRIB2 capability. Unless you plan to create extremely large domains, it is recommended to compile WPS in serial mode, regardless of whether you compiled WRFV3 in parallel. It is also recommended that you choose a GRIB2 option (make sure you do not choose one that states "NO_GRIB2"). You may choose a non-grib2 option, but most data is now in grib2 format, so it is best to choose this option. You can still run grib1 data when you have built with grib2.

Choose the option that lists a compiler to match what you used to compile WRF, serial, and grib2. Note: The option number will likely be different than the number you chose to compile WRFV4.1.2.

```console
Found Jasper environment variables for GRIB2 support...
  $JASPERLIB = /home/wrf/wrf_io/lib
  $JASPERINC = /home/wrf/wrf_io/include
------------------------------------------------------------------------
Please select from among the following supported platforms.

   1.  Linux x86_64, gfortran    (serial)
   2.  Linux x86_64, gfortran    (serial_NO_GRIB2)
   3.  Linux x86_64, gfortran    (dmpar)
   4.  Linux x86_64, gfortran    (dmpar_NO_GRIB2)
   5.  Linux x86_64, PGI compiler   (serial)
   6.  Linux x86_64, PGI compiler   (serial_NO_GRIB2)
   7.  Linux x86_64, PGI compiler   (dmpar)
   8.  Linux x86_64, PGI compiler   (dmpar_NO_GRIB2)
   9.  Linux x86_64, PGI compiler, SGI MPT   (serial)
  10.  Linux x86_64, PGI compiler, SGI MPT   (serial_NO_GRIB2)
  11.  Linux x86_64, PGI compiler, SGI MPT   (dmpar)
  12.  Linux x86_64, PGI compiler, SGI MPT   (dmpar_NO_GRIB2)
  13.  Linux x86_64, IA64 and Opteron    (serial)
  14.  Linux x86_64, IA64 and Opteron    (serial_NO_GRIB2)
  15.  Linux x86_64, IA64 and Opteron    (dmpar)
  16.  Linux x86_64, IA64 and Opteron    (dmpar_NO_GRIB2)
  17.  Linux x86_64, Intel compiler    (serial)
  18.  Linux x86_64, Intel compiler    (serial_NO_GRIB2)
  19.  Linux x86_64, Intel compiler    (dmpar)
  20.  Linux x86_64, Intel compiler    (dmpar_NO_GRIB2)
  21.  Linux x86_64, Intel compiler, SGI MPT    (serial)
  22.  Linux x86_64, Intel compiler, SGI MPT    (serial_NO_GRIB2)
  23.  Linux x86_64, Intel compiler, SGI MPT    (dmpar)
  24.  Linux x86_64, Intel compiler, SGI MPT    (dmpar_NO_GRIB2)
  25.  Linux x86_64, Intel compiler, IBM POE    (serial)
  26.  Linux x86_64, Intel compiler, IBM POE    (serial_NO_GRIB2)
  27.  Linux x86_64, Intel compiler, IBM POE    (dmpar)
  28.  Linux x86_64, Intel compiler, IBM POE    (dmpar_NO_GRIB2)
  29.  Linux x86_64 g95 compiler     (serial)
  30.  Linux x86_64 g95 compiler     (serial_NO_GRIB2)
  31.  Linux x86_64 g95 compiler     (dmpar)
  32.  Linux x86_64 g95 compiler     (dmpar_NO_GRIB2)
  33.  Cray XE/XC CLE/Linux x86_64, Cray compiler   (serial)
  34.  Cray XE/XC CLE/Linux x86_64, Cray compiler   (serial_NO_GRIB2)
  35.  Cray XE/XC CLE/Linux x86_64, Cray compiler   (dmpar)
  36.  Cray XE/XC CLE/Linux x86_64, Cray compiler   (dmpar_NO_GRIB2)
  37.  Cray XC CLE/Linux x86_64, Intel compiler   (serial)
  38.  Cray XC CLE/Linux x86_64, Intel compiler   (serial_NO_GRIB2)
  39.  Cray XC CLE/Linux x86_64, Intel compiler   (dmpar)
  40.  Cray XC CLE/Linux x86_64, Intel compiler   (dmpar_NO_GRIB2)

Enter selection [1-40] : 3
------------------------------------------------------------------------
Configuration successful. To build the WPS, type: compile
------------------------------------------------------------------------

Testing for NetCDF, C and Fortran compiler

This installation NetCDF is 64-bit
C compiler is 64-bit
Fortran compiler is 64-bit


Your Fortran + NETCDF did not run successfully.
```
The metgrid.exe and geogrid.exe programs rely on the WRF model's I/O libraries. There is a line in the configure.wps file that directs the WPS build system to the location of the I/O libraries from the WRF model.

Above is the default setting. As long as the name of the WRF model's top-level directory is "WRFV4.4.1" and the WPS and WRF directories are at the same level (which they should be if you have followed exactly as instructed on this page so far), then the existing default setting is correct and there is no need to change it. If it is not correct, you must modify the configure file and then save the changes before compiling.

You can now compile WPS. Compilation should take a few minutes. The ongoing compilation can be checked.
```console
$ ./compile >& compile.log &
$ tail -f compile.log
```
Once the compilation completes, to check whether it was successful, you need to look for 3 main executables in the WPS top-level directory. Then verify that they are not zero-sized.
```console
$ ls -ls *.exe
geogrid.exe
metgrid.exe
ungrib.exe
```

## Static geography data

The WRF modeling system is able to create idealized simulations, though most users are interested in the real-data cases. To initiate a real-data case, the domain's physical location on the globe and the static information for that location must be created. This requires a data set that includes such fields as topography and land use categories. Move to your HOME directory, download the file and unpack it.
```console
$ cd $HOME
$ wget http://www2.mmm.ucar.edu/wrf/src/wps_files/geog_high_res_mandatory.tar.gz
$ tar -xvzf geog_high_res_mandatory.tar.gz

```
The directory infomation is given to the geogrid program in the namelist.wps file in the &geogrid section. The data expands to approximately 29 GB. This data allows a user to run the geogrid.exe program.

```console
$ mkdir run_wrf
$ cd run_wrf
$ ln -sf ../WPS/*exe .
$ ln -sf ../WPS/geogrid .
$ ln -sf ../WPS/metgrid .
$ ln -sf ../WPS/ungrib/Variable_Tables/Vtable.GFS Vtable
$ ln -sf ../WPS/link_grib.csh .
$ ln -sf ../WRF/run/* .
$ rm namelist.input
$ nano namelist.wps

&share
 wrf_core = 'ARW',
 max_dom = 1,
 start_date = '2022-09-27_00:00:00', '--:00:00', '--:00:00',
 end_date   = '2022-09-27_03:00:00', '--:00:00', '--:00:00',
 interval_seconds = 10800,
/

&geogrid
 parent_id         = 1,1,2,
 parent_grid_ratio = 1,3,3,
 i_parent_start    = 1,99,122,
 j_parent_start    = 1,52,117,
 e_we          = 600,607,457,
 e_sn          = 300,298,346,
 geog_data_res = '30s','usgs_lakes+default','usgs_lakes+default',
 dx = 9000,
 dy = 9000,
 map_proj =  'mercator',
 ref_lat   = -2.613,
 ref_lon   = 118.815,
 truelat1  = -2.613,
 truelat2  = 0,
 stand_lon = 118.815,
 geog_data_path = '/home/<your-user-name>/WPS_GEOG',
 ref_x = 310.5,
 ref_y = 156.0,
/

&ungrib
 out_format = 'WPS',
 prefix = 'FILE',
/

&metgrid
 fg_name = 'FILE',
/

$ ./link_grib.csh /home/<your-user-name>/Documents/gfs*
$ ./geogrid.exe
$ ./ungrib.exe
$ ./metgrid.exe
$ nano namelist.input

 &time_control
 run_days                            = 0,
 run_hours                           = 3,
 run_minutes                         = 0,
 run_seconds                         = 0,
 start_year                          = 2022, 2019,
 start_month                         = 09,   09, 
 start_day                           = 27,   04,
 start_hour                          = 00,   12,
 end_year                            = 2022, 2019,
 end_month                           = 09,   09,
 end_day                             = 27,   06,
 end_hour                            = 03,   00,
 interval_seconds                    = 10800
 input_from_file                     = .true.,.true.,
 history_interval                    = 60,  60,
 frames_per_outfile                  = 1, 1,
 restart                             = .false.,
 restart_interval                    = 3600,
 io_form_history                     = 2
 io_form_restart                     = 2
 io_form_input                       = 2
 io_form_boundary                    = 2
 /

 &domains
 time_step                           = 15,
 time_step_fract_num                 = 0,
 time_step_fract_den                 = 1,
 max_dom                             = 1,
 e_we                                = 600,    220,
 e_sn                                = 300,    214,
 e_vert                              = 45,     45,
 dzstretch_s                         = 1.1
 p_top_requested                     = 5000,
 num_metgrid_levels                  = 34,
 num_metgrid_soil_levels             = 4,
 dx                                  = 9000,
 dy                                  = 9000,
 grid_id                             = 1,     2,
 parent_id                           = 0,     1,
 i_parent_start                      = 1,     53,
 j_parent_start                      = 1,     25,
 parent_grid_ratio                   = 1,     3,
 parent_time_step_ratio              = 1,     3,
 feedback                            = 1,
 smooth_option                       = 0
 /

 &physics
 physics_suite                       = 'TROPICAL'
 mp_physics                          = -1,    -1,
 cu_physics                          = -1,    -1,
 ra_lw_physics                       = -1,    -1,
 ra_sw_physics                       = -1,    -1,
 bl_pbl_physics                      = -1,    -1,
 sf_sfclay_physics                   = -1,    -1,
 sf_surface_physics                  = -1,    -1,
 radt                                = 15,    15,
 bldt                                = 0,     0,
 cudt                                = 0,     0,
 icloud                              = 1,
 num_land_cat                        = 21,
 sf_urban_physics                    = 0,     0,
 fractional_seaice                   = 1,
 /

 &fdda
 /

 &dynamics
 hybrid_opt                          = 2, 
 w_damping                           = 0,
 diff_opt                            = 2,      2,
 km_opt                              = 4,      4,
 diff_6th_opt                        = 0,      0,
 diff_6th_factor                     = 0.12,   0.12,
 base_temp                           = 290.
 damp_opt                            = 3,
 zdamp                               = 5000.,  5000.,
 dampcoef                            = 0.2,    0.2,
 khdif                               = 0,      0,
 kvdif                               = 0,      0,
 non_hydrostatic                     = .true., .true.,
 moist_adv_opt                       = 1,      1,
 scalar_adv_opt                      = 1,      1,
 gwd_opt                             = 1,      0,
 /

 &bdy_control
 spec_bdy_width                      = 5,
 specified                           = .true.
 /

 &grib2
 /

 &namelist_quilt
 nio_tasks_per_group = 0,
 nio_groups = 1,
 /

$ ./real.exe
$ .wrf.exe

```
