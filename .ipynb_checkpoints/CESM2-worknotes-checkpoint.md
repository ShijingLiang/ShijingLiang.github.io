layout: page
title: "CESM2-worknotes"
permalink: /CESM2-worknotes/

# CESM work notes: How to change CO2 in CESM2?

[//]:# (B1850 case)
[//]:# (B1PCT case)
[//]:# (BCO2x4 case)
[//]:# (BHIST_BDRD case)
[//]:# (BHIST_BPRP case)

## Projectives:
- Understand basic variables specified in *env_run.xml* and *user_nl_cam*.
- Learned how to change the way CO2 evolves and prescribed CO2 concentration in the model.
----------------------------------------------
# **Variables in env_run.xml:** settings of how CO2 will change in the model and each components
<table><tr><td bgcolor=gold>Note: <font color="red">DO NOT</font> change these variables in env_xml.</td></tr></table>
Variables in env_run.xml depend on the compset settings and changing them will directly alter the case build process.

## **CCSM_BGC**: exchange of surface upward flux of CO2 among atm, lnd, ocn
<font color="red">CCSM_BGC is responsible for deciding whether to send CO2 from atmosphere to land and ocean components in the model, and how CO2 will change —— either prognostic (calculated) or diagnostic (presribed).</font>

Compsets with prognostic CO2 will have long name ending with "BPRP"
Compsets with diagnostic CO2 will have long name ending with "DBRD"

For example, [compset settings in CESM2.1.3](https://docs.cesm.ucar.edu/models/cesm2/config/2.1.3/compsets.html):

|Compset|Long name|
|---|---|
|BHIST|HIST_CAM60_CLM50%BGC-CROP_CICE_POP2%ECO%ABIO-DIC_MOSART_CISM2%NOEVOLVE_WW3_BGC%<font color="red">BPRP</font>|
|BHIST_BPRP|HIST_CAM60_CLM50%BGC-CROP_CICE_POP2%ECO%ABIO-DIC_MOSART_CISM2%NOEVOLVE_WW3_BGC%<font color="red">BPRP</font>|

> valid_values: none, CO2A, CO2B, CO2C  
>- CO2A
    - sets the driver namelist variable flds_co2a = .true.
    - **prognostic/diagnostic** CO2 at the lowest model level to be sent from the atmosphere to the land and ocean.  

>- CO2B
    - sets the driver namelist variable flds_co2b = .true.
    - **prognostic/diagnostic** CO2 at the lowest model level to be sent from the atmosphere just to the land, and the **surface upward flux of CO2** to be sent from the land back to the atmosphere 

>- CO2C
    - sets the driver namelist variable flds_co2c = .true.
    - **prognostic/diagnostic** CO2 at the lowest model level to be sent from the atmosphere to the land and ocean, and the **surface upward flux of CO2** to be sent from the land and the open ocean back to the atmosphere.

## **CCSM_CO2_PPMV** : set constant CO2 values
Mechanism for setting the CO2 value in ppmv for CLM if CLM_CO2_TYPE is constant for POP if OCN_CO2_TYPE is constant. For example,

>entry id="CCSM_CO2_PPMV" value="284.7"

### **CLM_CO2_TYPE**

>valid_values:constant, diagnostic, prognostic

### **OCN_CO2_TYPE**
>valid_values: constant, prognostic, diagnostic, box_atm_co2

Determines how CO2 changes in CLM or OCN.  
- If value is **constant**, CO2 constant value will be specified in **CCSM_CO2_PPMV**  
- If value is either **diagnostic/prognostic**, the atmosphere model MUST send CO2 to CLM.

# **Variables in user_nl_cam**: specify different types of CO2 values in the model
Users are suggest to make modifications in user_nl_cam. Variables in user_nl_cam  associate with CO2 concentration variations in the atmosphere. If you are going to change prescribed CO2 concentration, modifications are expected here. For more details, please refer to [CAM6.3 Namelist Definitions](https://docs.cesm.ucar.edu/models/cesm2/settings/current/cam_nml.html)

## **Catogories: CO2_cycle**
Variables under the catogory CO2_cycle, related to whether surface emissions of anthropogenic CO2 will be passed to the atmosphere.

### **co2_flag** 

|Entry type|Valid values|
|---       |---         |
|logical   |.true. <br>.false.</br>|

>- If TRUE turn on CO2 code.
>- Default: set by build-namelist

### **co2_cycle_rad_passive**  

|Entry type|Valid values|
|---       |---         |
|logical   |.true. <br>.false.</br>|

>- Defalt: **FALSE**

This settings is corresponding to do independent BGC (biogeochemical) and RAD (radiative passive) experiments.  
- If TRUE, CO2 tracer is in radiative computations in the atmosphere.  
- If FALSE, radiative computations use prescribed CO2, and the CO2 tracer is a passive diagnostic tracer.

### **co2_readflux_fuel & co2flux_fuel_file**

|Entry type|Valid values|
|---       |---         |
|logical   |.true. <br>.false.</br>|

>- If TRUE read co2 fuel flux from file.
>- File path specificed in co2flux_fuel_file. For example: "/cam/ggas/emissions-cmip6_CO2_anthro_surface_175001-201512_fv_1.9x2.5_c20181011.nc"
>- Default: set by build-namelist

### **co2_readflux_aircraft & aircraft_co2_file**

|Entry type|Valid values|
|---       |---         |
|logical   |.true. <br>.false.</br>|

>- If true, read co2 aircraft flux from file.
>- File path specificed in aircraft_co2_file. For example: "atm/cam/ggas/emissions-cmip6_CO2_anthro_ac_175001-201512_fv_0.9x1.25_c20181011.nc"
>- Default: set by build-namelist

### **co2_read_flux_ocn & co2flux_ocn_file**

|Entry type|Valid values|
|---       |---         |
|logical   |.true. <br>.false.</br>|

>- If TRUE read co2 ocn flux from file. File path specificed in co2flux_ocn_file.  
>- Default: **FALSE**

## **Catogories: ghg_cam**

### **scenario_ghg**
<table><tr><td bgcolor=palegoldenrod>Controls <font color='orangered'>PRESRIBED</font> CO2, CH4, N2O, CFC11, CFC12 volumn mixing ratios.</td></tr></table>

![test](./pics/scenario_ghg.png)

|Entry type|Valid values|
|---       |---         |
|char*16   |'FIXED'<br>'RAMPED'<br>'RAMP_CO2_ONLY'<br>'CHEM_LBC_FILE'|

>- Default: set by build-namelist

## **FIXED:** When atmosphere CO2 concentration is a fixed value specified in [co2vmr](#co2vmr)

### **co2vmr**

|Entry type|Possible default values|
|---       |---         |
|real      |367.0e-6    |

>- CO2 volume mixing ratio. This is used as the [***time invariant surface value***](#b.-CCSM_CO2_PPMV) of CO2 if no time varying values are specified.  
>- Default: set by build-namelist
>- [scenario_ghg='FIXED'](#Catogories:-ghg_cam)

## **RAMPED_CO2_ONLY:** only CO2 mixing ratios are ramped at a rate determined by  [ramp_co2_annual_rate](#ramp_co2_annual_rate), [ramp_co2_cap](#ramp_co2_cap), [ramp_co2_start_ymd](#ramp_co2_start_ymd)
This setting is applied to 1pct increase CO2 experiments.

### **ramp_co2_annual_rate**

|Entry type|Valid values|
|---       |---         |
|real      |any real    |

>- Amount of co2 ramping per year (percent).
>- Only used if scenario_ghg = 'RAMP_CO2_ONLY'  
>- Default: 1.0

### **ramp_co2_cap**

|Entry type|Valid values|
|---       |---         |
|real      |any real    |

>- CO2 cap if > 0, floor otherwise. Specified as multiple or fraction of inital value
>- e.g., Setting to 4.0 will cap at 4x initial CO2 setting.
>- Only used if scenario_ghg = 'RAMP_CO2_ONLY'  
>- Default: boundless if ramp_co2_annual_rate > 0, zero otherwise.

### **ramp_co2_start_ymd**

|Entry type|Valid values|
|---       |---         |
|integer   |any integer |

>- Date on which ramping of CO2 begins. The date is encoded as an integer in the form YYYYMMDD.
>- Only used if scenario_ghg = 'RAMP_CO2_ONLY'
>- Default: 0

## **CHEM_LBC_FILE:** volumn mixing ratios are set from the chemistry lower boundary conditions dataset specified by [flbc_file](#flbc_file)
Apply to prescribed CO2 concentration experiments. CO2 concentration is specified as low boundary forcing.

### **flbc_file**

|Entry type|Valid values|
|---       |---         |
|char*256  |any char    |

>- Full pathname of dataset for fixed lower boundary conditions. For example, "atm/waccm/lb/LBC_1765-2100_1.9x2.5_CCMI_RCP60_za_RNOCStrend_c141002.nc"
>- Default: set by build-namelist.

### **flbc_type**

|Entry type|Valid values|
|---       |---         |
|char*8    |'CYCLICAL'<br>'SERIAL'<br>'FIXED'|

>- Type of time interpolation for fixed lower boundary data.
>- Default: 'CYCLICAL'

### **flbc_cycle_yr**

|Entry type|Valid values|
|---       |---         |
|integer   |any integer |

>- The cycle year of the fixed lower boundary data if flbc_type is 'CYCLICAL'.
>- Format: YYYY
>- Default: 0

### **flbc_fixed_ymd**

|Entry type|Valid values|
|---       |---         |
|integer   |any integer |

>- The date at which the fixed lower boundary data is fixed
>- flbc_type is 'FIXED'.
>- Format: YYYYMMDD
>- Default: 0

## **RAMPED:** surface GHGs are time interpolated from [bndtvghg](#bndtcghg)
### **bndtcghg**

|Entry type|Valid values|
|---       |---         |
|char*256  |any char    |

>- Full pathname of time-variant boundary dataset for greenhouse gas surface values. For example, "atm/cam/ggas/ghg_hist_1765-2005_c091218.nc"
>- Default: set by build-namelist

### **rampyear_ghg**

|Entry type|Valid values|
|---       |---         |
|integer   |any integer |

>- The value of this variable (> 0) fixes the year of the lower bounding value (i.e., the value for calendar day 1.0) used in the interpolation.
>- For example, if rampyear_ghg = 1950, then the GHG surface values will be the result of interpolating between the values for 1950 and 1951 from the dataset.
>- Default: 0
