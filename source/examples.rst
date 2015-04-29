Examples
========

pysat tends to reduce certain science data investigations to the construction of a routine(s) that makes that investigation unique, a call to a seasonal analysis routine, and some plotting commands. Several demonstrations are offered in this section.

Seasonal Occurrence by Orbit
----------------------------

How often does a particular thing occur on a orbit-by-orbit basis? Let's find out. For VEFI, let us calculate the occurrence of a positive perturbation in the meridional component (North/South) of the geomagnetic field.

.. code:: python

   import os
   import pysat
   import matplotlib.pyplot as plt
   import pandas as pds
   import numpy as np

   # set the directory to save plots to
   results_dir = ''

   # select vefi dc magnetometer data, use longitude to determine where
   # there are changes in the orbit (local time info not in file)
   orbit_info = {'index':'longitude', 'kind':'longitude'}
   vefi = pysat.Instrument(platform='cnofs', name='vefi', tag='dc_b', 
                           clean_level=None, orbit_info=orbit_info)

   # define function to remove flagged values
   def filter_vefi(inst):
       idx, = np.where(vefi['B_flag']==0)
       vefi.data = vefi.data.iloc[idx]
       return
   # attach function to vefi 
   vefi.custom.add(filter_vefi,'modify')
   # set limits on dates analysis will cover, inclusive
   start = pds.datetime(2010,5,9)
   stop = pds.datetime(2010,5,15)

   # if there is no vefi dc magnetometer data on your system 
   # run command below
   # where start and stop are pandas datetimes (from above)
   # pysat will automatically register the addition of this data at the end    
   # of download
   vefi.download(start, stop)

   # leave bounds unassigned to cover the whole dataset 
   vefi.bounds = (start,stop)

   # perform occurrence probability calculation
   # any data added by custom functions is available within routine below
   ans = pysat.ssnl.occur_prob.by_orbit2D(vefi, [0,360,144], 'longitude', 
                [-13,13,104], 'latitude', ['dB_mer'], [0.], returnBins=True)

   # a dict indexed by data_label is returned
   # in this case, only one, we'll pull it out
   ans = ans['dB_mer']
   # plot occurrence probability
   f, axarr = plt.subplots(2,1, sharex=True, sharey=True)
   masked = np.ma.array(ans['prob'], mask=np.isnan(ans['prob']))                                   
   im=axarr[0].pcolor(ans['binx'], ans['biny'], masked)
   axarr[0].set_title('Occurrence Probability Delta-B Meridional > 0')
   axarr[0].set_ylabel('Latitude')
   axarr[0].set_yticks((-13,-10,-5,0,5,10,13))
   axarr[0].set_ylim((ans['biny'][0],ans['biny'][-1]))
   plt.colorbar(im,ax=axarr[0], label='Occurrence Probability')

   im=axarr[1].pcolor(ans['binx'], ans['biny'],ans['count'])
   axarr[1].set_xlabel('Longitude')  
   axarr[1].set_xticks((0,60,120,180,240,300,360))
   axarr[1].set_xlim((ans['binx'][0],ans['binx'][-1]))
   axarr[1].set_ylabel('Latitude')
   axarr[1].set_title('Number of Orbits in Bin')

   plt.colorbar(im,ax=axarr[1], label='Counts')
   f.tight_layout()                                 
   plt.show()
   plt.savefig(os.path.join(results_dir,'ssnl_occurrence_by_orbit_demo') )

Result

.. image:: ./images/ssnl_occurrence_by_orbit_demo.png

Orbit-by-Orbit Plots
--------------------

Plotting a series of orbit-by-orbit plots is a great way to become familiar with a data set. If the data set doesn't come with orbit information, this can be a challenge. Orbits also go past day breaks, so if data comes in daily files this requires loading multiple files at once, joining the data together, etc. pysat goes through that trouble for you.

.. code:: python

   import os
   import pysat
   import matplotlib.pyplot as plt
   import pandas as pds

   # set the directory to save plots to
   results_dir = ''

   # select vefi dc magnetometer data, use longitude to determine where
   # there are changes in the orbit (local time info not in file)
   orbit_info = {'index':'longitude', 'kind':'longitude'}
   vefi = pysat.Instrument(platform='cnofs', name='vefi', tag='dc_b', 
                           clean_level=None, orbit_info=orbit_info)

   # set limits on dates analysis will cover, inclusive
   start = pysat.datetime(2010,5,9)
   stop = pysat.datetime(2010,5,12)

   # if there is no vefi dc magnetometer data on your system
   # then run command below
   # where start and stop are pandas datetimes (from above)
   # pysat will automatically register the addition of this data at the end 
   # of download
   vefi.download(start, stop)

   # leave bounds unassigned to cover the whole dataset 
   vefi.bounds = (start,stop)

   for orbit_count, vefi in enumerate(vefi.orbits):
       # for each loop pysat puts a copy of the next available 
       # orbit into   vefi.data
       # changing .data at this level does not alter other orbits
       # reloading the same orbit will erase any changes made
    
       # satellite data can have time gaps, which leads to plots
       # with erroneous lines connecting measurements on 
       # both sides of the gap
       # command below fills in any data gaps using a 
       # 1-second cadence with NaNs
       # see pandas documentation for more info
       vefi.data = vefi.data.resample('1S',  fill_method='ffill', 
                                      limit=1, label='left' )

       f, ax = plt.subplots(7, sharex=True, figsize=(8.5,11))
    
       ax[0].plot(vefi['longitude'], vefi['B_flag'])
       ax[0].set_title( vefi.data.index[0].ctime() +' - ' + 
                        vefi.data.index[-1].ctime() )
       ax[0].set_ylabel('Interp. Flag')
       ax[0].set_ylim((0,2))
    
       p_params = ['B_north', 'B_up', 'B_west', 'dB_mer',
		   'dB_par', 'dB_zon']
       for a,param in zip(ax[1:],p_params):	
          a.plot(vefi['longitude'], vefi[param])
          a.set_title(vefi.meta[param].long_name)
          a.set_ylabel(vefi.meta[param].units)
    
       ax[6].set_xlabel(vefi.meta['longitude'].long_name)
       ax[6].set_xticks([0,60,120,180,240,300,360])
       ax[6].set_xlim((0,360))   
    
       f.tight_layout()
       fname = 'orbit_%05i.png' % orbit_count
       plt.savefig(os.path.join(results_dir, fname) )
       plt.close()

Output

.. image:: ./images/orbit_00000.png

