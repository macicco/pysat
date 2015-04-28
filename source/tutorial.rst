The core functionality of pysat is exposed through the pysat.Instrument object. To work with Magnetometer data from the Vector Electric Field Instrument onboard the Communications/Navigation Outage Forecasting System (C/NOFS) begin with

.. code:: python

   vefi = pysat.Instrument(platform='cnofs', name='vefi', tag='dc_b')

Behind the scenes pysat uses a python module named cnofs_vefi that understands how to interact with 'dc_b' data. Let's download some data,

.. code:: python

   import pysat
   # define date range to download data and download
   start = pysat.datetime(2009,5,9)
   stop = pysat.datetime(2009,5,12)
   vefi.download(start, stop)

The data is downloaded to pysat_data_dir/platform/name/tag/*, in this case
pysat_data_dir/cnofs/vefi/dc_b/. pysat_data_dir should have previously been set using

.. code:: python
   
   pysat.utils.set_data_dir(path=path)

Data is loaded into vefi using the .load method using year, day of year; by date; or by filename.

.. code:: python

   vefi.load(2009,1)
   vefi.load(date=start)
   vefi.load(fname=fname)
   
The pandas DataFrame holding the data is available in .data. Convenience access is also available at the instrument level.

.. code:: python

    # all data
    print vefi.data
    # particular magnetic component
    print vefi.data.dB_mer
    # alternative access
    print vefi['dB_mer']
    # slicing
    print vefi[0:10, 'dB_mer']
    # slicing by date time
    print vefi[start:stop, 'dB_mer']

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








	  

	  