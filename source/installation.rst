
Installation
============
Starting from scratch:
   Python and associated packages for science are freely available from a couple 	commercial vendors, `Enthought <https://store.enthought.com>`_ and `Contiuum Analytics <http://continuum.io/downloads>`_. Core science packages such as numpy, scipy, matplotlib, pandas and many others may also be installed directly via pip or your favorite package manager. 


----

**pysat**

----

Pysat itself may be installed from a terminal command line via::

   pip install pysat

Pysat requires some external non-python libraries for loading science data sets stored in netCDF and CDF formats.

----

**Common Data Format**

----

- CDF Library from NASA (http://cdf.gsfc.nasa.gov) 
   - `Mac OS X Installer <http://cdaweb.gsfc.nasa.gov/pub/software/cdf/dist/cdf36_0/macosX/cdf36_0-setup_universal_binary.tar.gz>`_
- SpacePy

::

   pip install spacepy

----

**netCDF**

----

- netCDF C Library from Unidata (http://www.unidata.ucar.edu/downloads/netcdf/index.jsp)
- netCDF4-python

::

  pip install netCDF4

----

**pandas**

----

To get the forked pandas that accommodates pandas Series and DataFrames within each cell of a Series use::

   pip install git+https://github.com/rstoneback/pandas.git

The forked pandas is required for full support of higher dimensional data sets. A pull-request is planned.

----

**Data Directory**

----

Pysat will maintain organization of data from various platforms. Upon the first

.. code:: python

   import pysat

pysat will remind you to set the top level directory that will hold the data,

.. code:: python

   pysat.utils.set_data_dir(path=path)




