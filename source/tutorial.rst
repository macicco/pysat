
Tutorial
========

Basics
------

The core functionality of pysat is exposed through the pysat.Instrument object. The intent of the Instrument object is to offer a single interface for interacting with science data that is independent of measurement platform. The layer of abstraction presented by the Instrument object allows for things to occur in the background that can make science data analysis simpler and more rigorous.

To begin, 

.. code:: python
   
   import pysat

The data directory pysat looks in for data (pysat_data_dir) needs to be set upon the first import,

.. code:: python

   pysat.utils.set_data_dir(path=path_to_existing_directory)

**Instantiation**

----

To work with Magnetometer data from the Vector Electric Field Instrument onboard the Communications/Navigation Outage Forecasting System (C/NOFS), start with a pysat Instrument object.

.. code:: python

   vefi = pysat.Instrument(platform='cnofs', name='vefi', tag='dc_b')

Behind the scenes pysat uses a python module named cnofs_vefi that understands how to interact with 'dc_b' data. 

**Download**

----

Let's download some data,

.. code:: python

   # define date range to download data and download
   start = pysat.datetime(2009,5,9)
   stop = pysat.datetime(2009,5,12)
   vefi.download(start, stop)

The data is downloaded to pysat_data_dir/platform/name/tag/, in this case pysat_data_dir/cnofs/vefi/dc_b/.


**Load Data**

----

Data is loaded into vefi using the .load method using year, day of year; date; or filename.

.. code:: python

   vefi.load(2009,129)
   vefi.load(date=start)
   vefi.load(fname='cnofs_vefi_bfield_1sec_20090509_v05.cdf')
   
When the pysat load routines runs it stores the instrument data into vefi.data. The data structure is a pandas DataFrame_, a highly capable structure with labeled rows and columns. Convenience access to the data is also available at the instrument level.

.. _DataFrame: http://pandas.pydata.org/pandas-docs/stable/dsintro.html#dataframe

.. code:: python

    # all data
    print vefi.data
    # particular magnetic component
    print vefi.data.dB_mer

    # Convenience access
    print vefi['dB_mer']
    # slicing
    print vefi[0:10, 'dB_mer']
    # slicing by date time
    print vefi[start:stop, 'dB_mer']

See pysat.Instrument for more.

**Clean Data**

-----

Before data is available in .data it passes through an instrument specific cleaning routine. The amount of cleaning is set by the clean_level keyword,

.. code:: python

   vefi = pysat.Instrument(platform='cnofs', name='vefi', 
			   tag='dc_b', clean_level='none')

Four levels of cleaning may be specified, 

===============     ===================================
**clean_level** 	        **Result**
---------------     -----------------------------------
  clean		    Generally good data
  dusty		    Light cleaning, use with care
  dirty		    Minimal cleaning, use with caution
  none		    No cleaning, use at your own risk
===============     ===================================

**Metadata**

----

Metadata is also stored along with the main science data.

.. code:: python

   # all metadata
   print vefi.meta.data
   # dB_mer metadata
   print vefi.meta['dB_mer']
   # units
   vefi.meta['dB_mer'].units
   # update units for dB_mer
   vefi.meta['dB_mer'] = {'units':'new_units'}
   # update display name, long_name
   vefi.meta['dB_mer'] = {'long_name':'Fancy Name'}
   # add new meta data
   vefi.meta['new'] = {'units':'fake', 'long_name':'Display'}

Data may be assigned to the instrument, with or without metadata.

.. code:: python
   
   vefi['new_data'] = new_data

The same activities may be performed for other instruments in the same manner. In particular, measurements from the Ion Velocity Meter and profiles of electron density from COSMIC,

.. code:: python

   # assignment with metadata
   ivm = pysat.Instrument(platform='cnofs', name='ivm', tag='')
   ivm.load(date=date)
   ivm['double_mlt'] = {'data':2.*inst['mlt'], 'long_name':'Double MLT', 
                        'units':'hours'}

.. code:: python

   cosmic = pysat.Instrument('cosmic2013','gps', tag='ionprf',  clean_level='clean')
   start = pysat.datetime(2009,1,2)
   stop = pysat.datetime(2009,1,3)
   # requires CDAAC account 
   cosmic.download(start, stop, user='', password='')
   cosmic.load(date=start)
   # the profiles column has a DataFrame in each element which stores
   # all relevant profile information indexed by altitude
   # print part of the first profile, selection by integer location
   print cosmic[0,'profiles'].iloc[55:60, 0:3]
   # print part of profile, selection by altitude value
   print cosmic[0,'profiles'].ix[196:207, 0:3]

Output for both print statements:

.. code:: python

                  ELEC_dens    GEO_lat    GEO_lon
   MSL_alt                                       
   196.465454  81807.843750 -15.595786 -73.431015
   198.882019  83305.007812 -15.585764 -73.430191
   201.294342  84696.546875 -15.575747 -73.429382
   203.702469  86303.039062 -15.565735 -73.428589
   206.106354  87460.015625 -15.555729 -73.427803
    
Custom Functions
----------------

Science analysis is built upon custom data processing. To simplify this task custom functions may be attached to the Instrument object. Each function is run automatically when new data is loaded.

Modify Functions

	The instrument object is passed to function without copying, modify in place.

.. code:: python

   def custom_func_modify(inst, optional_param=False):
       inst['double_mlt'] = 2.*inst['mlt']

Add Functions

	A copy of the instrument is passed to function, data to be added is returned.

.. code:: python

   def custom_func_add(inst, optional_param=False):
       return 2.*inst['mlt']

Add Function Including Metadata

.. code:: python

   def custom_func_add(inst, optional_param1=False, optional_param2=False):
       return {'data':2.*inst['mlt'], 'name':'double_mlt', 
               'long_name':'doubledouble', 'units':'hours'}

Attaching Custom Function

.. code:: python

   ivm.custom.add(custom_func_modify, 'modify', optional_param2=True)
   ivm.load(2009,1)
   print ivm['double_mlt']
   ivm.custom.add(custom_func_add, 'add', optional_param2=True)
   ivm.bounds = (start,stop)
   custom_complicated_analysis_over_season(ivm)

The output of custom_func_modify will always be available from instrument object, regardless of what level the science analysis is performed.


Iteration
---------

The whole VEFI data set may be iterated over on a daily basis

.. code:: python

    for vefi in vefi:
	print 'Maximum meridional magnetic perturbation ', vefi['dB_mer'].max()

Each loop of the python for initiates a vefi.load() for the next date, starting with the first available date. By default the instrument instance will iterate over all available data. It is equivalent to

.. code:: python
   
   date_array = pysat.utils.season_date_range(start,stop)
   for date in date_array:
       vefi.load(date=date)
       print 'Maximum meridional magnetic perturbation ', vefi['dB_mer'].max()

The output is,

.. code:: python

   Returning cnofs vefi dc_b data for 05/09/10
   Maximum meridional magnetic perturbation  19.3937
   Returning cnofs vefi dc_b data for 05/10/10
   Maximum meridional magnetic perturbation  23.745
   Returning cnofs vefi dc_b data for 05/11/10
   Maximum meridional magnetic perturbation  25.673
   Returning cnofs vefi dc_b data for 05/12/10
   Maximum meridional magnetic perturbation  26.583

Bounds may be set to control the dates covered by the iteration, 

.. code:: python

   # continuous season
   vefi.bounds = (start, stop)
   # multi-season season
   vefi.bounds = ([start1, start2], [stop1, stop2])
   # iterate over custom season
   for vefi in vefi:
	print 'Maximum meridional magnetic perturbation ', vefi['dB_mer'].max()


Orbit Support
-------------

Pysat has functionality to determine orbits on the fly from loaded data. These orbits will span day breaks as needed (generally). Information about the orbit needs to be provided at intialization. The 'index' is the name of the data to be used for determining orbits, and 'kind' indicates type of orbit. See pysat.Orbit for latest inputs.

.. code:: python
    
   info = {'index':'mlt', 'kind':'local time'}
   ivm = pysat.Instrument(platform='cnofs', name='ivm', tag='', 
                          clean_level='clean', orbit_info=info)
   start = [pd.datetime(2009,1,1), pd.datetime(2010,1,1)]
   stop = [pd.datetime(2009,4,1), pd.datetime(2010,4,1)]
   ivm.bounds = (start, stop)
   for ivm in ivm.orbits:
       print 'next available orbit ', ivm.data

  