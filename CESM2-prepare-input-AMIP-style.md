Notifications
---
- In the CICE namelist there is a variable called ice_ic. This the file name from where the initial ice state variables are set. 
	1. If ice_ic = 'default', this means it sets it to a uniform ice cover wherever it is "cold". 
	2. If ice_ic = 'none' it is initialized with no ice at all. 

- The SST and ICE are interpolated between the mid-month values by a linear interpolation. The model uses: SST_cpl while SST_cpl_prediddle is just there for information purpose. They are obtained used the icesst tools. This "diddling" is done with icesst tools. The icesst tools can be find at /work/ese-liangll/CESM2.1.3/my_cesm_sandbox/components/cam/tools/icesst/
  
- The "diddling"is a procedure that ensures that the monthly mean of the daily-interpolated SST anomalies matches the monthly SST anomalies. Detailed calculation can be found in the protocol for AMIP simulation: [AMIP Details index](https://pcmdi.llnl.gov/mips/amip/details/index.html)

- Empirical relationship between SST and sea-ice described: James W. Hurrell, James J. Hack, Dennis Shea, Julie M. Caron, and James Rosinski, 2008: A New Sea Surface Temperature and Sea Ice Boundary Dataset for the Community Atmosphere Model. _J. Climate_, **21**, 5145–5153.

The information above can be found in [FAQ: Data ocean slab mode (DOCN-SOM) | Page 4 | DiscussCESM Forums (ucar.edu)](https://bb.cgd.ucar.edu/cesm/threads/faq-data-ocean-slab-mode-docn-som.2017/page-4)

How to use icesst tool to construct AMIP6-like forcings
----
## Prepare data
![[tutorial4_4.png]]
## File format
I first interpolated satellite-observed data to CESM2 output resolution with python. It is important to keep the file formatted and add the attributes for each variable.

-----------

**time** (cftime.DatetimeNoleap)
- For Feburary time should be 
	cftime.DatetimeNoLeap(1980, 2, 15, 0, 0, 0, 0, has_year_zero=True)
- For months 1,3,5,7,8,10,12
	cftime.DatetimeNoLeap(1980, 1, 16, 12, 0, 0, 0, has_year_zero=True)
- For months 4,6,9,10,11
	cftime.DatetimeNoLeap(1980, 1, 16, 0, 0, 0, 0, has_year_zero=True)

**date** (int YYYYMMDD)
For Feburary, date should be 19800215
Other months, date is 19800115

**datesec** (int 43200 or 0)
- For months 1,3,5,7,8,10,12
	datesec is 43200
- For other months
	datesec is 0

## Fill values on land using ncl
The SST and SIC are interpolated over land. It is necssaey for the model to have values over land otherwise, it will crash but the values over land don't impact the simulation. Because I am not familiar with ncl, you could also format your output data here. The data must be formatted before starting the bcgen tool.

---------------------------------------------

>filename = "/mnt/e/CESM-SST/sstice_obs.nc"
>out_file = "/mnt/e/CESM-SST/sstice_obs_fillnan.nc"
>fin = addfile(filename,"r")
>SST = fin->SST
>SEAICE = fin->SEAICE
>
>poisson_grid_fill(SST, True, 1, 500, 0.01, 0.6, 0)
>poisson_grid_fill(SEAICE, True, 1, 500, 0.01, 0.6, 0)
>
>fout = addfile(out_file,"c")
>fout->SST = (/SST/)
>fout->SEAICE = (/SEAICE/)
>fout->ICEFRAC = (/SEAICE/100./)

## Use bcgen tool to do the "diddling" approach

1. Open and read README!!!
	Read README at /work/ese-liangll/CESM2.1.3/my_cesm_sandbox/components/cam/tools/icesst/
2. Check environment settings
	You will need to check Fortran compiler in your .bashrc, by typing vi ~/.bashrc
	\[ese-liangll@login02 ~]$ vi ~/.bashrc 
	> export F90=ifort
3. Go to bcgen tool directory 
>	\[ese-liangll@login02 ~]$ cd /work/ese-liangll/CESM2.1.3/my_cesm_sandbox/components/cam/tools/icesst/bcgen/
>	\[ese-liangll@login02 bcgen]$ vi Makefile

Set LIB_NETCDF and INC_NETCDF path 
![[tutorial4_1.png]]
Set other flags
Here I simply set FC=ifort
![[tutorial4_2.png]]

4. After modifying the Makefile, compile the bcgen tool
	>\[ese-liangll@login02 bcgen]$ make

If you are lucky and make it successfully, you may notice there is a "\*.o" file for each "\*.f90" file
Most importantly, you obtain the "bcgen" excutable file
![[tutorial4_3.png]]

5. Revise the namelist
>	\[ese-liangll@login02 bcgen]$ vi namelist
>
> &cntlvars
 mon1 = 1
 iyr1 = 1980
 monn = 12
 iyrn = 2024
 mon1rd = 12
 iyr1rd = 1980
 monnrd = 2
 iyrnrd = 2024
 mon1clm = 1
 iyr1clm = 1982
 monnclm = 12
 iyrnclm = 2001
 mon1out = 1
 iyr1out = 2014
 monnout = 12
 iyrnout = 2023
 /
 
6. Run the bcgen tool
>	\[ese-liangll@login02 bcgen]$./bcgen -i sstice.nc -c sstice_clim.nc -t sstice_ts.nc < namelist
