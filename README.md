# UFS
Disclaimer:
This is a personal documentation.
For UFS tutorials and relevant resources, please contact EPIC.  EPIC is funded  by NOAA to support the UFS community.


Useful links:

User guide: https://ufs-srweather-app.readthedocs.io/en/release-public-v2.2.0/index.html
Forum: https://github.com/ufs-community/ufs-srweather-app/discussions

CCPP page can be found at
https://dtcenter.org/community-code/common-community-physics-package-ccpp
CCPP technical doc: https://ccpp-techdoc.readthedocs.io/en/ufs_srw_app_v2.2.0/
CCPP scientific doc: https://dtcenter.ucar.edu/GMTB/UFS_SRW_App_v2.2.0/sci_doc/index.html

Directory structure can be found at section 1.2.2.2:
https://ufs-srweather-app.readthedocs.io/en/release-public-v2.2.0/BackgroundInfo/TechnicalOverview.html

A complete description of the levels of support, along with a list of preconfigured and configurable platforms:
https://github.com/ufs-community/ufs-srweather-app/wiki/Supported-Platforms-and-Compilers
 
EPIC: https://epic.noaa.gov/


 
 UFS SRW Quick Start

1.	Clone SRW App from GitHub
> git clone -b release/public-v2.2.0 https://github.com/ufs-community/ufs-srweather-app.git

UFS SRW App will be in $DIR/ufs-srweather-app
See UFS SRW User Guide for code repo and directory structure
https://ufs-srweather-app.readthedocs.io/en/release-public-v2.2.0/BackgroundInfo/TechnicalOverview.html#code-repositories-and-directory-structure

2.	Check out external repositories
> cd $DIR/ufs-srweather-app
> ./manage_externals/checkout_externals

Checking out externals: ufs_utils, ufs-weather-model, upp, arl_nexus, aqm-utils, workflow-tools (specified in $DIR/ufs-srweather-app/Externals.cfg)

Check ufs-srweather-app/sorc/ for these external repo

3.	Build SRW executable
	> ./devbuild.sh --platform=derecho

Pre-configured platforms are platforms where all of the required software libraries for building community releases of UFS models and applications are available in a central place via HPC-Stack or spack-stack. This includes: Derecho, Hera, Hercules, Jet, Orion, Gaea and NOAA cloud

The build process at Derecho takes about 12-13 min; Executables will be in $DIR/ufs-srweather-app/exec directory.  The default setting is to build UFS, UFS_UTILS, and UPP only. (i.e., BUILD_NEXUS and BUILD_AQM_UTILS are ‘off’)

UFS SRW App configuration summary will be in:
ufs-srweather-app/exec/ufs_srweather_app.settings

The build logs for the CMake and Make step will be in ufs-srweather-app/build/log.cmake and ufs-srweather-app/build/log.make.  

4.	Load the python enviSourcronment for the workflow
> Source $DIR/ufs-srweather-app/etc/lmod-setup.sh derecho
> module use $DIR/ufs-srweather-app/modulefiles
> module load wflow_derecho
> conda activate workflow_tools

Now you are ready to run UFS SRW 
You need to load the environment everytime 

5.	Configure and generate regional workflow
create or modify ush/config.yaml   //See SRW Experiment Configuration//
> cd $DIR/ufs-srweather-app/ush
> ./generate_FV3LAM_wflow.py

This script generates SRW experiment and stages the data.

Workflow parameters are discussed in section 3.1 in the user guide
https://ufs-srweather-app.readthedocs.io/en/release-public-v2.2.0/CustomizingTheWorkflow/ConfigWorkflow.html


6.	Run the regional workflow
> cd $EXPTDIR 
> ./launch_FV3LAM_wflow.sh

 $EXPDIR =$DIR/expt_dirs/$EXPT_SUBDIR, $EXPT_SUBDIR is specified in config.yaml
	
The logs for generating and launching the workflow will be in $EXPDIR/log.generate_FV3LAM_wflow and $EXPDIR/vlog.launch_FV3LAM_wflow.

The logs for each step will be in $EXPDIR/log directory


7.	Monitor the experiment progress and relaunch the workflow, if needed
> ./launch_FV3LAM_wflow.sh; tail -n 40 log.launch_FV3LAM_wflow

When config.yaml automates the workflow using cron
> rocotostat -w FV3LAM_wflow.xml -d FV3LAM_wflow.db -v 10

You can track the experiment’s progress by reissuing the rocotostat or check $EXPDIR/log/FV3LAM_wflow.log 
SRW Experiment Configuration
srw_my_first_run 
Use config.community.yaml as the template
<   MACHINE: derecho
<   ACCOUNT: UALB0039
---
>   MACHINE: hera
>   ACCOUNT: an_account
27d25
<   EXTRN_MDL_SOURCE_BASEDIR_ICS: /glade/work/epicufsrt/contrib/UFS_SRW_data/v2p2/input_model_data/FV3GFS/grib2/${yyyymmddhh}
32d29
<   EXTRN_MDL_SOURCE_BASEDIR_LBCS: /glade/work/epicufsrt/contrib/UFS_SRW_data/v2p2/input_model_data/FV3GFS/grib2/${yyyymmddhh}

Specifying the following parameters:
$MACHINE, $ACCOUNT, $EXPT_SUBDIR, $EXTRN_MDL_SOURCE_BASEDIR_ICS, and $EXTRN_MDL_SOURCE_BASEDIR_LBCS

On Level 1 systems, the data required to run SRW App tests are already available (see Table 2.4 in SRW doc; https://ufs-srweather-app.readthedocs.io/en/release-public-v2.2.0/BuildingRunningTesting/RunSRW.html)

srw_2nd_run_cronjob 
automate the workflow using cron, modified from config.yaml_my_first_run
<   USE_CRON_TO_RELAUNCH: true
<   CRON_RELAUNCH_INTVL_MNTS: 3
---
>   USE_CRON_TO_RELAUNCH: false

No need to relaunch the workflow
UFS output history files (dynf and phyf files) will be in $EXPDIR/YYYYMMDDHH
UFS post-processed files (srw.t18z.natlev.f###.rrfs_conus_25km.grib2 and srw.t18z.prslev.f###.rrfs_conus_25km.grib2) will be in $EXPDIR/YYYYMMDDHH/postprd

srw_3rd_run_addplot
Add plot task to task:rocoto:taskgroups, modified from config.yaml_2nd_run_cronjob
40,42d38
<   PLOT_FCST_START: 0
<   PLOT_FCST_INC: 6
<   PLOT_FCST_END: 12
50d45
< 	taskgroups: '{{ ["parm/wflow/prep.yaml", "parm/wflow/coldstart.yaml", "parm/wflow/post.yaml", "parm/wflow/plot.yaml"]|include }}'

Identical workflow as srw_2nd_run_cronjob except one new task (plot_allvars)
##.png will be in $EXPDIR/YYYYMMDDHH/postprd.  

The plots are 10mwind, 250wind, 2mdew, 2mt, qpf, refc, sfcape,  slp and uh25 for 10 m wind, 250 mb wind, 2m dew point temp, 2m temp, accumulated precip, composite reflectivity, sfc-based CAPE, sea level pressure, and 2-5km updraft helecity


srw_4th_run_addvx
Add vrfy task; modified from config.yaml_3rd_run_addplot
need to specify the directory for staged obs dataset:
CCPA_OBS_DIR: Climatology-Calibrated Precipitation Analysis (CCPA) data.
MRMS_OBS_DIR: Multi-Radar/Multi-Sensor (MRMS) System Analysis data
NDAS_OBS_DIR: NAM Data Assimilation System (NDAS) data
12,14c12,14
<   CCPA_OBS_DIR: ""
<   MRMS_OBS_DIR: ""
<   NDAS_OBS_DIR: ""
---
>   CCPA_OBS_DIR: /glade/work/epicufsrt/contrib/UFS_SRW_data/v2p2/obs_data/ccpa/proc
>   MRMS_OBS_DIR: /glade/work/epicufsrt/contrib/UFS_SRW_data/v2p2/obs_data/mrms/proc
>   NDAS_OBS_DIR: /glade/work/epicufsrt/contrib/UFS_SRW_data/v2p2/obs_data/ndas/proc
50c50
< 	taskgroups: '{{ ["parm/wflow/prep.yaml", "parm/wflow/coldstart.yaml", "parm/wflow/post.yaml", "parm/wflow/plot.yaml"]|include }}'
---
> 	taskgroups: '{{ ["parm/wflow/prep.yaml", "parm/wflow/coldstart.yaml", "parm/wflow/post.yaml", "parm/wflow/verify_pre.yaml", "parm/wflow/verify_det.yaml"]|include }}'

Identical workflow as srw_2nd_run_cronjob and srw_3rd_run_addpost except for new tasks (get_obs and run_MET)

Met statistics (GridStat,  PcpCombine_fcst, PointStat) will be in $EXPDIR/YYYYMMDDHH/metprd


srw_5th_run_phys
Change physics suite from GFS_v16 to RAP, modified from config.yaml_2nd_run_cronjob
<   CCPP_PHYS_SUITE: FV3_GFS_v16
---
>   CCPP_PHYS_SUITE: FV3_RAP

Check valid_vals_CCPP_PHYS_SUITE in ush/valid_param_vals.yaml: 
"FV3_GFS_2017_gfdlmp", "FV3_GFS_2017_gfdlmp_regional", "FV3_GFS_v15p2",
"FV3_GFS_v15_thompson_mynn_lam3km", "FV3_GFS_v16", "FV3_GFS_v17_p8", "FV3_RRFS_v1beta", "FV3_WoFS_v0", "FV3_HRRR",
"FV3_RAP"

srw_5th_rerun_phys_dfplot
Repeat the run with RAP physics suite, add plotting-diff option, modified from config.yaml_5th_run_phys
39c38
<   COMOUT_REF: "${EXPT_BASEDIR}/srw_2nd_run_cronjob/${PDY}${cyc}/postprd"
---
>   COMOUT_REF: ""
47d45
< 	taskgroups: '{{ ["parm/wflow/prep.yaml", "parm/wflow/coldstart.yaml", "parm/wflow/post.yaml", "parm/wflow/plot.yaml"]|include }}'



Set the COMOUT_REF variable to plot the difference between two experiments (srw_5th_run_phys versus srw_2nd_run_cronjob)

EXPT_BASEDIR:  The base directory in which the experiment directory will be created.
see $DIR/ush/config_defaults.yaml for explanation

Check the experiment's variable definition file (var_defns.sh) in $EXPTDIR
EXPTDIR='/glade/work/clu/ufs4HU/expt_dirs/srw_5th_run_phys'
CCPP_PHYS_SUITE='FV3_RAP'
EXPT_BASEDIR='/glade/work/clu/ufs4HU/expt_dirs'
EXPT_SUBDIR='srw_5th_run_phys'

Note: 1) $EXPT_SUBDIR remains the same as the previous run (srw_5th_run_phys); and 2)  PREEXISTING_DIR_METHOD: rename
Previous run is renamed as $EXPT_BASEDIR/srw_5th_run_phys_old_20240210_200023

Check the user guide
https://ufs-srweather-app.readthedocs.io/en/release-public-v2.2.0/index.html
$PREEXISTING_DIR_METHOD: Pre-Existing Directory Parameter, This variable determines how to deal with pre-existing directories.  The options include delete, rename, reuse, and quit.

Identical workflow as previous srw_5th_run_cronjob except one new task (plot_allvars)
##.png will be in $EXPDIR/YYYYMMDDHH/postprd.  
10mwind_diff_conus_f003.png  2mt_diff_conus_f003.png   sfcape_diff_conus_f003.png
250wind_diff_conus_f003.png  qpf_diff_conus_f003.png   slp_diff_conus_f003.png
2mdew_diff_conus_f003.png	refc_diff_conus_f003.png  uh25_diff_conus_f003.png


Let’s do one more out-of-the-box SRW experiment

srw_6th_run_Ind3km
Change the domain to SUBCONUS_Ind_3km , modified from config.yaml_3rd_run_addplot
<   PREDEF_GRID_NAME: SUBCONUS_Ind_3km
---
>   PREDEF_GRID_NAME: RRFS_CONUS_25km
43d42
<   PLOT_DOMAINS: ["regional"]

The plotting scripts are designed to generate plots over the entire CONUS by default. PLOT_DOMAINS is not explicitly specified in the srw_3rd_run_addplot experiment’s config.yaml.  Check var_defns.sh:   PLOT_DOMAINS=( "conus" )

For grid information, you can examine [task_make_grid] and  [grid_params] in var_defns.sh 

For pre-defined domains (PREDEF_GRID_NAME), you can review ush/valid_param_vals.yaml and ush/predef_grid_params.yaml
"RRFS_CONUS_25km", "RRFS_CONUS_13km", "RRFS_CONUS_3km",
"RRFS_CONUScompact_25km", "RRFS_CONUScompact_13km", "RRFS_CONUScompact_3km",
"RRFS_AK_13km", "RRFS_AK_3km",
"CONUS_25km_GFDLgrid", "CONUS_3km_GFDLgrid", "GSD_HRRR_25km",
"AQM_NA_13km", "RRFS_NA_13km", "RRFS_NA_3km",
"SUBCONUS_Ind_3km", "WoFS_3km"


 
Now - let’s try to tweak config.yaml 


Workflow End-to-end (WE2E) Tests (https://ufs-srweather-app.readthedocs.io/en/release-public-v2.2.0/tables/Tests.html)
It’s quite helpful to review sample config.yaml for various experiment set-up ufs-srweather-app/tests/WE2E/test_configs

It contains 8 directories: aqm, default_configs, grids_extrn_mdls_suites_nco, verification, custom_grids, grids_extrn_mdls_suites_community, ufs_case_studies and wflow_features


To make a new grid, you can browse sample yaml or review user guide
ufs-srweather-app/tests/WE2E/test_configs/custom_grids 

https://ufs-srweather-app.readthedocs.io/en/release-public-v2.2.0/CustomizingTheWorkflow/LAMGrids.html#creating-user-generated-grids

https://ufs-srweather-app.readthedocs.io/en/release-public-v2.2.0/CustomizingTheWorkflow/ConfigWorkflow.html#make-grid-configuration-parameters

 
srw_7th_run_neus12km
Create a new grid (neus_12km), modified from config.yaml_6th_run_ind3km 
Add task_make_grid and task_run_post; modify task_run_fcst
<   GRID_GEN_METHOD: ESGgrid
---
>   PREDEF_GRID_NAME: SUBCONUS_Ind_3km
28,36d26
< task_make_grid:
<   ESGgrid_LON_CTR: -78.
<   ESGgrid_LAT_CTR:  43.
<   ESGgrid_DELX: 12000.0
<   ESGgrid_DELY: 12000.0
<   ESGgrid_NX: 200
<   ESGgrid_NY: 140
<   ESGgrid_WIDE_HALO_WIDTH: 6
<   ESGgrid_PAZI: 0.0
47,50d36
<   DT_ATMOS: 60
<   LAYOUT_X: 4
<   LAYOUT_Y: 4
<   BLOCKSIZE: 32
52,66d37
<   WRTCMP_write_groups: 1
<   WRTCMP_write_tasks_per_group: '{{ LAYOUT_Y }}'
<   WRTCMP_output_grid: lambert_conformal
<   WRTCMP_cen_lon: '{{ task_make_grid.ESGgrid_LON_CTR }}'
<   WRTCMP_cen_lat: '{{ task_make_grid.ESGgrid_LAT_CTR }}'
<   WRTCMP_lon_lwr_left: -91.6565
<   WRTCMP_lat_lwr_left: 34.8696
<   WRTCMP_stdlat1: '{{ task_make_grid.ESGgrid_LAT_CTR }}'
<   WRTCMP_stdlat2: '{{ task_make_grid.ESGgrid_LAT_CTR }}'
<   WRTCMP_nx: 198
<   WRTCMP_ny: 138
<   WRTCMP_dx: 12000.0
<   WRTCMP_dy: 12000.0
< task_run_post:
<   POST_OUTPUT_DOMAIN_NAME: neus_12km

This experiment fails at  plot_allvars step
log/plot_allvars_2019061518.log:
/glade/work/clu/ufs4HU/expt_dirs/srw_7th_run_neus12km/2019061518//postprd/srw.t18z.prslev.f000.srw_7th_run_neus12km.grib2

/glade/work/clu/ufs4HU/expt_dirs/srw_7th_run_neus12km/2019061518/postprd/srw.t18z.natlev.f009.neus_12km.grib2

The fix is to revise
task_run_post:
  POST_OUTPUT_DOMAIN_NAME: srw_7th_run_neus12km


Re-run the experiment with  config.yaml_7th_rerun
workflow:
  EXPT_SUBDIR: srw_neus_12km
task_run_post:
  POST_OUTPUT_DOMAIN_NAME: srw_neus_12km



srw_8th_run_add_predef_grid
Create user-generated grids, NEUS_12km 
Revise predef_grid_params.yaml: "NEUS_12km"
Revise valid_param_vals.yaml: valid_vals_PREDEF_GRID_NAME
Revise config.yaml to use the new grid


19c19
<   PREDEF_GRID_NAME: NEUS_12km
---
>   GRID_GEN_METHOD: ESGgrid
25a26,34
> task_make_grid:
>   ESGgrid_LON_CTR: -78.
>   ESGgrid_LAT_CTR:  43.
>   ESGgrid_DELX: 12000.0
>   ESGgrid_DELY: 12000.0
>   ESGgrid_NX: 200
>   ESGgrid_NY: 140
>   ESGgrid_WIDE_HALO_WIDTH: 6
>   ESGgrid_PAZI: 0.0
35a45,48
>   DT_ATMOS: 60
>   LAYOUT_X: 4
>   LAYOUT_Y: 4
>   BLOCKSIZE: 32
36a50,62
>   WRTCMP_write_groups: 1
>   WRTCMP_write_tasks_per_group: '{{ LAYOUT_Y }}'
>   WRTCMP_output_grid: lambert_conformal
>   WRTCMP_cen_lon: '{{ task_make_grid.ESGgrid_LON_CTR }}'
>   WRTCMP_cen_lat: '{{ task_make_grid.ESGgrid_LAT_CTR }}'
>   WRTCMP_lon_lwr_left: -91.6565
>   WRTCMP_lat_lwr_left: 34.8696
>   WRTCMP_stdlat1: '{{ task_make_grid.ESGgrid_LAT_CTR }}'
>   WRTCMP_stdlat2: '{{ task_make_grid.ESGgrid_LAT_CTR }}'
>   WRTCMP_nx: 198
>   WRTCMP_ny: 138
>   WRTCMP_dx: 12000.0
>   WRTCMP_dy: 12000.0
52c78
< 	taskgroups: '{{ ["parm/wflow/prep.yaml", "parm/wflow/coldstart.yaml", "parm/wflow/post.yaml"]|include
---
> 	taskgroups: '{{ ["parm/wflow/prep.yaml", "parm/wflow/coldstart.yaml", "parm/wflow/post.yaml", "parm/wflow/plot.yaml"]|include


This run failed. 
/glade/work/clu/ufs4HU/expt_dirs/srw_8th_run_add_predef_grid_old_20240214_163404/log/FV3LAM_wflow.log,  the error msg is:
2024-02-14 16:27:07 -0700 :: derecho6 :: Submission status of previously pending run_fcst_mem000 is failure!  qsub: directive error: -l select={{ task_run_fcst.NNODES_RUN_FCST // 1 }}:mpiprocs=128:ncpus=128
Var_defns.sh: NNODES_RUN_FCST='{{ (PE_MEMBER01 + PPN_RUN_FCST - 1) // PPN_RUN_FCST }}'
The fix is to hard code WRTCMP_write_tasks_per_group to 4 in ush/predef_grid_params.yaml
The issue is that the ush/predef_grid_params.yaml file is expecting hard coded values for the listed parameters, not YAML {{...}} entries. 
See https://github.com/ufs-community/ufs-srweather-app/discussions/1000 for more details

Revise  ush/predef_grid_params.yaml  (the old one is renamed as  ush/predef_grid_params.yaml_failed) and resubmit the experiment

Same config.yaml.  With revised ush/predef_grid_params.yaml, the 8th-run completed successfully.    

srw_9th_run_phys_namelist
Change namelist variable, modified from  config.yaml_8th_run_add_predef_grid
17c17
<   EXPT_SUBDIR: srw_9th_run_phy_namelist
---
>   EXPT_SUBDIR: srw_8th_run_add_predef_grid
26d25
<   FV3_NML_YAML_CONFIG_FN: /glade/work/clu/ufs4HU/ufs-srweather-app/parm/FV3.para.input.yml

For srw_8th_run_add_predef_grid experiment:
Check var_defns.sh 
FV3_NML_YAML_CONFIG_FN='FV3.input.yml'
Check input.nml 
 iaer=5111  (for FV3_GFS_v16)

To change iaer from 5111 to 111, you will need to modify input config file (FV3_NML_YAML_CONFIG_FN)

One option is to modify /ufs-srweather-app/parm/FV3.input.yml directly.
FV3.input.yml:  This configuration file maintains the modifications that need to be made to the base FV3 namelist specified in parm/input.nml.FV3

My approach is to create another input config file (FV3_NML_YAML_CONFIG_FN)
/ufs-srweather-app/parm/FV3.para.input.yml: 
Change iaer to 111 for FV3_GFS_v16 physics suite

Check run directory /glade/work/clu/ufs4HU/expt_dirs/srw_9th_run_phy_namelis
Input.nml: iaer 111


 
Wrap-up

When you are seeking some clarification, you can post at repo: Discussions
https://github.com/ufs-community/ufs-srweather-app/discussions/992

When you identify a bug, you can post at repo: Issues
https://github.com/ufs-community/ufs-srweather-app/issues/1004

When you like to try different config, check use cases
$DIR/ufs-srweather-app/tests/WE2E/test_configs/



Will share a SandBox tarball
It includes parm and ush

parm:  
FV3.input.yml  FV3.para.input.yml

ush:
config.yaml_my_first_run  		config.yaml_2nd_run_cronjob  
config.yaml_3rd_run_addplot 	config.yaml_4th_run_addvx
config.yaml_5th_run_phys		config.yaml_5th_rerun_phys_dfplot config.yaml_6th_run_ind3km       	config.yaml_7th_run_neus12km     
config.yaml_7th_rerun   		config.yaml_8th_run_add_predef_grid  		   	         config.yaml_9th_run_phy_namelist	
      	       

predef_grid_params.yaml_ORIG	predef_grid_params.yaml
valid_param_vals.yaml_ORIG		valid_param_vals.yaml



