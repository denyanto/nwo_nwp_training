# WRF Configure on HPC
Configuration of WRF on HPC Centos
These are some installation notes taken in the process of installing WRF on a HPC with CentOS 8. This tutorial is for **Group Fellowship Training Course on Numerical Weather Prediction (NWP) 26 September - 25 October 2022**


## Connect the server


```console
$ ssh test@182.16.251.51 -p 1234
```

## Create a work directori


```console
$ pwd
$ ls
$ mkdir NWP_<your-name>
$ cd NWP_<your-name>

```

## Clone the WRF models 

First and foremost, it is very important to have a installed WRF, as well as WPS and WRFDA. If you have these installed, you should be given a path for the location of each.
```console
$ locate geogrid.exe
$ locate real.exe
$ ls /opt/ohpc/pub/model
$ ln -sf /opt/ohpc/pub/model/WRF .
$ ln -sf /opt/ohpc/pub/model/WPS .
$ ln -sf /opt/ohpc/pub/model/WRFPLUS .
$ ln -sf /opt/ohpc/pub/model/WRFDA .
$ ln -sf /opt/ohpc/pub/model/WRFDomainWizard .
$ ln -sf /opt/ohpc/pub/model/ARWpost .
$ ln -sf /opt/ohpc/pub/model/WPS_GEOG .
$ mkdir DATA 	# for input data (GFS etc)

```

## Prepare the simulation
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
```


## Setting namelist.wps

```console
$ nano namelist.wps

```

```console
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
 geog_data_path = '/home/<your-user-name>/NWP_<your-name>/WPS_GEOG',
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
```


## Linking input data and running WPS

```console

$ ./link_grib.csh /home/<your-user-name>/NWP_<your-name>/DATA/gfs*
$ ./geogrid.exe
$ ./ungrib.exe
$ ./metgrid.exe
```


## Setting namelist.input

```console
$ nano namelist.input

```

```console
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
```

## Running WRF process

```console
$ ./real.exe
$ mpirun -n 16 ./wrf.exe &
$ tail -f rsl.out.0000
```

## Running WRF process with **sbatch**

```console
$ nano job.sh
```

```console
	#!/bin/bash
	# sbatch config file
	# for running WRF (wrf.exe) using openMP

	#SBATCH -J job1          			# Job name
	#SBATCH -o result.txt               # Name of stdout output file (%j expands to jobId)
	#SBATCH --ntasks=20                 # Total number of nodes requested
	#SBATCH --cpus-per-task=88          # Total number of mpi tasks requested
	#SBATCH --mem-per-cpu=1000          # Total number of memory requested (mb)
	#SBATCH -t 01:30:00                 # Run time (hh:mm:ss) - 1.5 hours

	# Launch MPI-based executable
	cd /home/<your-user-name>/NWP_<your-name>/run_wrf
	mpirun ./wrf.exe
```

```console
$ sbatch job.sh
$ squeue

```

## Check the results

```console
$ ncdump wrfout_d01_2022-09-27_00:00:00
```
## Finish