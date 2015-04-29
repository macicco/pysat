
Installation
============

Pysat requires some external non-python libraries for loading science data sets stored in netCDF and CDF formats.

- CDF: http://cdf.gsfc.nasa.gov
- netCDF: http://www.unidata.ucar.edu/software/netcdf/

Pysat itself may be installed via
	``pip install pysat``

Pandas may be obtained similarly,
	``pip install pandas``

To get the forked pandas that accommodates pandas Series and DataFrames within each cell of a Series use
	``pip install git+https://github.com/rstoneback/pandas.git``

The forked pandas is required for higher dimensional data sets. A pull-request is planned.

Pysat will maintain organization of data from various platforms. Upon the first
	``import pysat``

pysat will remind you to set the top level directory that will hold the data,
	``pysat.utils.set_data_dir(path=path)``




