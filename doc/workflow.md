
# Table of Contents

1.  [Compilation](#org8ef37b3)
    1.  [Compiling `NEMO_4.0.4_hpge` ](#org4258a9a)
    2.  [Compiling MEs adapted DOMAINcfg tool](#org23c8385)
2.  [Generate envelopes ](#org2a03cd0)
3.  [Create a `domain_cfg.nc`](#org3f5a749)
    1.  [`namelist_ref::namzgr_mes` for the DOMAINcfg tool ](#org2ff60cf)
    2.  [visualize `domain_cfg.nc`](#org7c47817)
4.  [HPG error testing](#org2223ba0)
    1.  [(optional) add an initial TS depth-profile](#orgb26468c)
    2.  [run HPGE test](#org2644c85)
    3.  [Create `maximum_hpge.nc`](#org20d0ae9)
    4.  [HPGE iteration](#org81071ba)


<a id="org8ef37b3"></a>

# Compilation


<a id="org4258a9a"></a>

## Compiling `NEMO_4.0.4_hpge` <a id="org4f93cc4"></a>

    interactive --exclusive
    module purge;
    module load buildenv-intel/2018a-eb;
    module load HDF5/1.8.19-nsc1-intel-2018a-eb;
    module load netCDF/4.4.1.1-HDF5-1.8.19-nsc1-intel-2018a-eb;
    
    cd <REPO_DIR>/src/f90/NEMO_4.0.4_hpge_ovf/;
    ./makenemo -m 'bi7' -r HPGTEST -n 'HPGTEST' -j 16


<a id="org23c8385"></a>

## Compiling MEs adapted DOMAINcfg tool

    cd <REPO_DIR>/src/f90/NEMO_4.0.4_hpge_ovf/tools
    ./maketools -m 'bi7' -n DOMAINcfg


<a id="org2a03cd0"></a>

# Generate envelopes <a id="orgec4f57b"></a>

To generate envelopes:

-   create some dir, for instance here `<REPO_DIR>/src/python/envelopes/<some_dir>`
-   add an existing `bathy_meter.nc` and corresponding `domain_cfg.nc` or `coordinates.nc`
-   create `<inputname>.inp` and set it up
-   link to `<REPO_DIR>/src/python/envelopes/generate_envelopes.py`
-   conda activate `pyogcm`
-   `python generate_envelopes.py <inputname>.inp`
    
    The result is a new bathymetry file: `bathy_meter.<inputname>.nc`


<a id="org3f5a749"></a>

# Create a `domain_cfg.nc`

With the new bathymetry file we create a `domain_cfg.nc`, where
additional settings should be given in the `namelist_cfg`.

1.  Copy contents of `<REPO_DIR>/src/scripts/create_domaincfg/` to some `<DOMAIN_BLD>` dir.
2.  Edit `<DOMAIN_BLD>/files/namelist_cfg` to suit your needs (see [3.1](#org46840e1)).
3.  Start an interactive node (`interactive --exclusive`). If it
    complains about memory start a fat node (`-C fat`).
4.  Run `./create_domaincfg.sh <outputdir> <bathymetry>`
    this should take less than 2 minutes.


<a id="org2ff60cf"></a>

## `namelist_ref::namzgr_mes` for the DOMAINcfg tool <a id="org46840e1"></a>

In this example for NEMO-NORDIC we use two envelopes with 43 and 13 layers respectively.

    !-----------------------------------------------------------------------
    &namzgr_mes    !   MEs-coordinate
    !-----------------------------------------------------------------------
       ln_envl     =   .TRUE. , .TRUE. , .FALSE. , .FALSE., .FALSE.  ! (T/F) If the envelope is used
       nn_strt     =     2    ,   1    ,    1   ,   1    ,   1     ! Stretch. funct.: Madec 1996 (0) or
                                                                   ! Song & Haidvogel 1994 (1) or
                                                                   ! Siddorn & Furner 2012 (2)
       nn_slev     =     43    ,   13   ,   20   ,   15   ,    0   ! number of s-lev between env(n-1)
                                                                   ! and env(n)
       rn_e_hc     =     25.0  ,   0.0 ,   0.0  ,   0.0  ,   0.0   ! critical depth for transition to
                                                                   ! stretch. coord.
       rn_e_th     =     1.5  ,    2.5 ,   2.4  ,   0.0  ,   0.0   ! surf. control param.:
                                                                   ! SH94 or MD96: 0<=th<=20
                                                                   ! SF12: thickness surf. cell
       rn_e_bb     =     -0.3 ,    0.5 ,   0.85 ,   0.0  ,   0.0   ! bot. control param.:
                                                                   ! SH94 or MD96: 0<=bb<=1
                                                                   ! SF12: offset for calculating Zb
       rn_e_al     =     2.0  ,     0.0 ,   0.0  ,   0.0  ,   0.0   ! alpha stretching param with SF12
       rn_e_ba     =     0.024  ,    0.0 ,   0.0  ,   0.0  ,   0.0   ! SF12 bathymetry scaling factor for
                                                                   ! calculating Zb
       rn_bot_min  = 9.0       ! minimum depth of the ocean bottom (>0) (m)
       rn_bot_max  = 700.0     ! maximum depth of the ocean bottom (= ocean depth) (>0) (m)
    
       ln_pst_mes   = .false.
       ln_pst_l2g   = .false.
       rn_e3pst_min = 20.
       rn_e3pst_rat = 0.1
    /


<a id="org7c47817"></a>

## visualize `domain_cfg.nc`

in `<REPO_DIR>/src/python/plot/vcoord/`
edit and run `plot_vlevels_MEs.py` or `plot_vlevels_zps.py`


<a id="org2223ba0"></a>

# HPG error testing


<a id="orgb26468c"></a>

## (optional) add an initial TS depth-profile

1.  add an initial TS depth-profile to
    `<REPO_DIR>/src/f90/NEMO_4.0.4_hpge_ovf/src/OCE/USR/usrdef_istate.F90`
    if necessary.
2.  recompile `NEMO_4.0.4_hpge` ([1.1](#org4f93cc4))


<a id="org2644c85"></a>

## run HPGE test

1.  copy contents of `<REPO_DIR>/src/scripts/run_hpgtest/` to preferred rundir
2.  select initial TS depth-profile in `test_template/namelist_cfg` (`namtsd::nn_tsd_type`)
3.  create rundir with hpge setup and submit
    `./run_hpgetest.sh <testname> <domcfg>`


<a id="org20d0ae9"></a>

## Create `maximum_hpge.nc`

-   edit and run `create_2D_hpge_field.py` (in `<REPO_DIR>/src/python/envelopes`)
-   (optional) visualize in the test dir: `ncview maximum_hpge.nc`


<a id="org81071ba"></a>

## HPGE iteration

Not happy with the HPGE? Go back to [2](#orgec4f57b) and use
 `maximum_hpge.nc` to create a new bathymetry with HPGE aware local
 smoothing (see example `.inp` files). Note that several
 `maximum_hpge.nc` input fields can be used.

Otherwise you're done and you can start running experiments.
