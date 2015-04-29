
Installation
============

Pysat itself may be installed via::

   pip install pysat

Pysat requires some external non-python libraries for loading science data sets stored in netCDF and CDF formats.

- CDF Library from http://cdf.gsfc.nasa.gov
  and SpacePy,
::

   pip install spacepy

- netCDF C Library from http://www.unidata.ucar.edu/software/netcdf/ 
  and netCDF4-python,
::

  pip install netCDF4

To get the forked pandas that accommodates pandas Series and DataFrames within each cell of a Series use::

   pip install git+https://github.com/rstoneback/pandas.git

The forked pandas is required for higher dimensional data sets. A pull-request is planned.

Pysat will maintain organization of data from various platforms. Upon the first

.. code:: python

   import pysat

pysat will remind you to set the top level directory that will hold the data,

.. code:: python

   pysat.utils.set_data_dir(path=path)




