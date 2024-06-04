How to use icesst tool to construct AMIP6-like forcings (sea surface temperature and sea ice)
----


## Prepare data
I first interpolated satellite-observed data to CESM2 output resolution with python. It is important to keep the file formatted and add the attributes for each variable.  
![[tutorial3_1.png]]


**time** should be cftime.DatetimeNoleap
For Feburary time should be 
>cftime.DatetimeNoLeap(1980, 2, 15, 0, 0, 0, 0, has_year_zero=True)
For months 1,3,5,7,8,10,12
cftime.DatetimeNoLeap(1980, 1, 16, 12, 0, 0, 0, has_year_zero=True)
For months 4,6,9,10,11
>cftime.DatetimeNoLeap(1980, 1, 16, 0, 0, 0, 0, has_year_zero=True)


**date** should be \int with format \YYYYMMDD
For Feburary, date should be \19800215
Other months, date is \19800115


**datesec** should be \int
For months 1,3,5,7,8,10,12
datesec is \43200
For other months
datesec is \0

## Fill values on land using ncl
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

Because I am not familiar with ncl, you could also format your output data here.
The data must be formatted above to be put into the bcgen tool


## Use bcgen tool to do the "diddling" approach
0. Open and read README!!!
Read README at /work/ese-liangll/CESM2.1.3/my_cesm_sandbox/components/cam/tools/icesst/
1. Check environment settings
You will need to check Fortran compiler in your .bashrc, by typing vi ~/.bashrc
\[ese-liangll@login02 ~]$ vi ~/.bashrc 
> export F90=ifort

1.1 Go to bcgen tool directory:
>/work/ese-liangll/CESM2.1.3/my_cesm_sandbox/components/cam/tools/icesst/bcgen/

1.2 Open Makefile in the bcgen tool directory 
>\[ese-liangll@login02 bcgen]$ vi Makefile

1.3 Set LIB_NETCDF and INC_NETCDF path  
![[tutorial3_2.png]]

1.4 Set other flags
Here I simply set FC=ifort  
![[tutorial3_21.png]]

2. Compile the bcgen tool
>\[ese-liangll@login02 bcgen]$ make

If you are lucky and make it successfully, you may notice there is a "\*.o" file for each "\*.f90" file.
Most importantly, you obtain the "bcgen" excutable file.

3. Revise the namelist
\[ese-liangll@login02 bcgen]$ vi namelist

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

4.  Run the bcgen tool
>\[ese-liangll@login02 bcgen]$./bcgen -i sstice.nc -c sstice_clim.nc -t sstice_ts.nc < namelist


These procedures are tedious, if you need to construct your own SST forcing file to put into the CESM2, you should follow these steps. Feel free to contact me if you met any problem.   
