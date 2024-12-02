# WRF-Python Tutorial 2024
#
## What is WRF-Python?
WRF-Python is a post-processing tool similar to NCL's WRF Package.
Contains over 30 diagnostic routines (CAPE, storm relative helicity, cloud top temperature, etc).
Interpolation routines (level, cross section, surface).
Utilities to help with plotting via cartopy, basemap, PyNGL.
WRF-ARW only.
WRF-Python is NOT a tool for running the WRF-ARW model with Python.
#
## Topics
1. Introduction to jupyter notebooks, numpy, xarray
2. Overview of WRF-ARW Output Data
3. WRF-Python Functions
4. Plotting
#
## 1 Introduction to jupyter, numpy, xarray
## What is Jupyter Notebook?
* Originally IPython Notebook (Python, R)
* The Jupyter Notebook is an open-source web application that allows you to create and share documents that contain live code, equations, visualizations and explanatory text.
* The Jupyter Notebook actually consists of an application and a document.
* The Jupyter Notebook application is a web browser application that allows editing and executing of jupyter notebook documents.
* Jupyter Notebook documents (usually ending with a .ipynb extension), are really just JSON-formatted text files that contain the code and rich text elements that will be rendered by the jupyter notebook application.
* Jupyter notebook documents are NOT Python scripts, so do not try to run them via the 'python' command. They need to be converted first.
* For this tutorial, when we refer to jupyter notebook, we're referring to both the application and document.
* Example: Google Colaboratory. Colab is a hosted Jupyter Notebook service that requires no setup to use and provides free access to computing resources, including GPUs and TPUs. Colab is especially well suited to machine learning, data science, and education.
# 
### Starting Google Colab
Open a web browser and type https://colab.google/ and then sign in with your google account.
![image](https://github.com/user-attachments/assets/4a938390-6fb4-4ffb-9e8e-676e3c79f1d6)
#
### Cells
* A google colab is a collection of cells, similar to jupyter notebook.
* Cells can be either executable code or text (markdown).
* Cells can also be specified as slides, which is how this slide show was made (along with the Rise plugin).
* Entering and executing code in cells is the same as having typed it in to the Python shell program.
* The order of execution of the cells can have impacts on variables that are used across the cells, so be careful when re-running cells.
* Aside from the first cell you run, the cells used in this tutorial should be more like independent scripts.
#
### Executing Cells
1. Click on the desired cell.
2. Press **CTRL + RETURN** to execute the cell or press **SHIFT + RETURN** to execute the cell and advance to the next cell.
3. Alternatively, you can use the Cell dropdown menu
#
### Restarting the Notebook
If your notebook crashes for some reason:
1. Use the Runtime dropdown menu at the top.
2. Execute Runtime -> Restart session.
#
### Shutting down the notebook
On your web browser, select the Home tab.
If you want to shutting down your notebook:
1. Use the Runtime dropdown menu at the top.
2. Execute Runtime -> Disconnect and delete runtime.
#
### Install important module
On your google colab cell, type the following sintax, and click running:
```console
!pip install cartopy
!pip install netCDF4
!pip install -q condacolab
import condacolab
condacolab.install()
!conda install -c conda-forge wrf-python
```
### Example 1.3: Verifying your Jupyter Environment
To set up the tutorial to work with your files, modify the **WRF_DIRECTORY** and **WRF_FILES** variables to point to your WRF files.
**IMPORTANT**: If for some reason your workbook crashes, you need to run this cell again before running the later examples.
```console
from __future__ import print_function

# This jupyter notebook command inserts matplotlib graphics in 
# to the workbook
%matplotlib inline

# Modify these to point to your own files
WRF_DIRECTORY = "../wrf_output"
WRF_FILES = ["wrfout_d01_2023-05-19_02%3A00%3A00",
             "wrfout_d01_2023-05-19_01%3A00%3A00",
             "wrfout_d01_2023-05-19_00%3A00%3A00"]


# Do not modify the code below this line
#------------------------------------------------------
# Turn off annoying warnings
import warnings
warnings.filterwarnings('ignore')

# Make sure the environment is good
import numpy
import cartopy
import matplotlib
from netCDF4 import Dataset
from xarray import DataArray
from wrf import (getvar, interplevel, vertcross, 
                 vinterp, ALL_TIMES)
import os

_WRF_FILES = [os.path.abspath(
    os.path.join(WRF_DIRECTORY, f)) for f in WRF_FILES]

# Check that the WRF files exist
try:
    for f in _WRF_FILES:
        if not os.path.exists(f):
            raise ValueError("{} does not exist. "
                "Check for typos or incorrect directory.".format(f))
except ValueError as e:
    # Try downloading then check again
    os.system("git submodule init")
    os.system("git submodule update")
    os.system("GIT_DIR={}/.git git checkout -- .".format(WRF_DIRECTORY))
    for f in _WRF_FILES:
        if not os.path.exists(f):
             raise e


# Create functions so that the WRF files only need
# to be specified using the WRF_FILES global above
def single_wrf_file():
    global _WRF_FILES
    return _WRF_FILES[0]

def multiple_wrf_files():
    global _WRF_FILES
    return _WRF_FILES

print("All tests passed!")
```
#
## Numpy
* Numpy is a Python package for performing array based operations, similar to Matlab and NCL.
* Numpy arrays can be created for the common types in C (named "dtype" in numpy)
  * int8, int16, int32, int64 (and unsigned versions)
  * float16, float32, float64 [default]
  * bool
  * complex64, complex128
* Arrays can be C-ordered (fastest on right) or Fortran-ordered (fastest on left). C-ordered by default.
### Numpy Basics
#### Array Creation
In this example, we're going to create an array of all zeros with 3x3x3 shape.
Here is how to create an array of floats and then integers.
```console
import numpy
array_float32 = numpy.zeros((3,3,3), "float32")
array_int32 = numpy.zeros((3,3,3), "int32")
```
#### Accessing Elements
To access elements in numpy, you use the bracket "[ ]" syntax.
Supply each desired index separated by commas (this is really a tuple).
You can use also negative indexes to pick indexes from the end.
```console
import numpy
my_array = numpy.zeros((3,3,3), "float32")

# Accessing elements
first_element = my_array[0,0,0]
last_element = my_array[-1,-1,-1]
mid_element = my_array[1,1,1]

# Setting an element
my_array[1,1,1] = 10.0
```
#### Slices
* Slices are a way to extract array subsets from an array.
* The syntax for a slice is the ':' character.
* Specifying the ':' for a dimension will return all values along that dimension.
* Specifying 'start : end' will take a subset of that dimension. Also, either start or end can be left blank.
* The end index is NOT included in the sliced data (opposite of NCL).
* Can also use a step value with 'start : end : step' if you want to increment values with something other than 1.
```console
import numpy
my_array = numpy.zeros((3,3,3), "float32")
first_row = my_array[0,0,:]
first_column = my_array[0,:,0]
first_z = my_array[:,0,0]
subset = my_array[:, :, 1:3]
reverse_z = my_array[::-1, :, :]
```

Also, slices are implicitly applied from left to right for unspecified dimensions.
```console
import numpy
my_array = numpy.zeros((3,3,3), "float32")
first_plane = my_array[0,:,:]

# This is the same as first_plane
first_plane2 = my_array[0]

# A short way to get everything
# Same as my_array[:,:,:]
all_elements = my_array[:]
```
#### Masked Arrays
* numpy uses a numpy array subclass called a MaskedArray.
* MaskedArrays contain a data array and a boolean mask array (True/False).
* Usually a fill_value is set in the data array at each location where the mask array is True.
* Numerous ways to convert a regular numpy array to a MaskedArray:
  * masked_equal
  * masked_greater
  * masked_where
#### Creating a MaskedArray
```console
import numpy
import numpy.ma

my_array = numpy.zeros((3,3,3), "float32")

# Now all the array elements are masked values
my_masked = numpy.ma.masked_equal(my_array, 0)
```
### Example 1.2: Numpy Basics
```console
import numpy
import numpy.ma

my_array = numpy.zeros((3,3,3), "float32")

print("my_array")
print(my_array)
print("\n")

# Setting an element
my_array[1,1,1] = 10.0

# Getting an element
mid = my_array[1,1,1]

print("Mid element set")
print(my_array)
print("\n")

# Getting a slice
my_slice = my_array[1,:,:]

print("my_slice")
print(my_slice)
print("\n")

# Masking the zeros
my_masked = numpy.ma.masked_equal(my_array, 0)

print("my_masked")
print(my_masked)
print("\n")
```
#
## xarray
* xarray expands upon numpy by adding dimension names, coordinate variables, and metadata.
* xarray array (DataArray) objects wrap around a numpy array (NOT a numpy array subclasses).
  * xarray HAS A numpy array (it is not an "IS A" relationship)
  * Often have to extract the numpy array from the xarray array before passing it to extension modules
  * Most numpy methods are available in xarray, but not all.
### Creating an xarray Array from a numpy Array
```console
import numpy
import xarray

my_array = numpy.zeros((3,3,3), "float32")

# Making up dimension names and 
# coordinates.
my_name = "my_xarray"

my_dims = ["bottom_top", "south_north", "west_east"]

my_coords = {"bottom_top" : [100., 200., 300.],
             "south_north": [40., 50., 60.],
             "west_east" : [-120., -110., -100.]
            }

my_attrs = {"info" : "This is my xarray"}

my_xarray = xarray.DataArray(my_array,
                             name=my_name,
                             dims=my_dims, 
                             coords=my_coords, 
                             attrs=my_attrs)
```
### Example 1.3: Creating an xarray DataArray
```console
import numpy
import xarray

my_array = numpy.zeros((3,3,3), "float32")

# Making up dimension names and 
# coordinates.
my_name = "my_xarray"

my_dims = ["bottom_top", "south_north", "west_east"]

my_coords = {"bottom_top" : [100., 200., 300.],
          "south_north": [40., 50., 60.],
          "west_east" : [-120., -110., -100.]
         }

my_attrs = {"info" : "This is my xarray"}

my_xarray = xarray.DataArray(my_array,
                           name=my_name,
                           dims=my_dims, 
                           coords=my_coords, 
                           attrs=my_attrs)

print(my_xarray)
```
### xarray and Missing Data Values
* xarray always uses IEEE NaN for missing data values.
  * Can cause problems with compiled numerical routines.
  * Can cause problems for algorithms expecting MaskedArrays.
* wrf-python includes the fill value information in the attribute section of the metadata (_FillValue).
* The to_np routine can be used to convert xarray arrays to numpy/masked arrays.
### Example 1.4: xarray and Missing Values
```console
import numpy
import numpy.ma
import xarray

from wrf import to_np

# Create a MaskedArray with 10.0 in the center
my_array = numpy.zeros((3,3,3), "float32")

my_array[1,1,1] = 10.0

my_masked = numpy.ma.masked_equal(my_array, 0)

# Making up dimension names and 
# coordinates.
my_name = "my_masked_xarray"

my_dims = ["bottom_top", "south_north", "west_east"]

my_coords = {"bottom_top" : [100., 200., 300.],
          "south_north": [40., 50., 60.],
          "west_east" : [-120., -110., -100.]
         }

my_attrs = {"info" : "This is my masked xarray",
           "_FillValue" : -999.0}

# Create the xarray DataArray
my_xarray = xarray.DataArray(my_masked,
                           name=my_name,
                           dims=my_dims, 
                           coords=my_coords, 
                           attrs=my_attrs)

print("xarray Array with Missing Values")
print(my_xarray)
print("\n")

# Covert back to a MaskedArray
converted = to_np(my_xarray)

print("Converted to a MaskedArray with to_np")
print(converted)
```
#
## 2.0 Overview of WRF Output Data
The first rule of data processing:
**"ALWAYS LOOK AT YOUR DATA"**

Why Look At WRF Data? Isn't It All the Same?
WRF can be configured in various ways and can have variables turned on and off.
If you run in to problems, it could be due to a variable missing.
Some users intentionally move the coordinate variables to a separate file to save space (not supported by NCL or wrf-python).
If your plot doesn't look right, there could be a map projection issue.
### Data Viewing Tools
There are numerous tools available to examine NetCDF data, from both outside and inside of Python.
* ncdump (used for this example)
* ncl_filedump
* netcdf4-python
* PyNIO
* xarray
### ncdump
ncdump is a program included with the NetCDF libraries that can be used to examine NetCDF data.
By supplying the '-h' option, only the data descriptions are returned. Otherwise, you'll get all of the data values, which can span miles.
To run:
```console
$ ncdump -h wrfout_d01_2023-05-19_00%3A00%3A00
```
### ncdump Output
```console
dimensions:
        Time = UNLIMITED ; // (1 currently)
        DateStrLen = 19 ;
        west_east = 89 ;
        south_north = 89 ;
        bottom_top = 34 ;
        bio_emissions_dimension_stag = UNLIMITED ; // (0 currently)
        klevs_for_dvel = 1 ;
        bottom_top_stag = 35 ;
        soil_layers_stag = 4 ;
        west_east_stag = 90 ;
        south_north_stag = 90 ;
        seed_dim_stag = 8 ;
variables:
        char Times(Time, DateStrLen) ;
.
.
.
```

### Dimensions
* WRF-ARW uses an Arakawa C-grid staggered grid [(taken from mmm website)] 1
* Mass related quantities (pressure, temperature, etc) are computed at the center of a grid cell.
* The u-component of the horizontal wind is calculated at the left and right edges of a grid cell. It has one more point in the x direction than the mass grid.
* The v-component of the horizontal wind is calculated at the bottom and top edges of a grid cell. It has one more point in the y direction than the mass grid.
* The corners of each grid box are know as the 'staggered' grid, and it has one additional point in both the x and y direction.
![image](https://github.com/user-attachments/assets/ce789085-5c2d-48e3-ad7f-157acd68d935)

```console
dimensions:
        Time = UNLIMITED ; // (1 currently)
        DateStrLen = 19 ;
        west_east = 89 ;
        south_north = 89 ;
        bottom_top = 34 ;
        bio_emissions_dimension_stag = UNLIMITED ; // (0 currently)
        klevs_for_dvel = 1 ;
        bottom_top_stag = 35 ;   <-- Extra grid point
        soil_layers_stag = 4 ;
        west_east_stag = 90 ;    <-- Extra grid point
        south_north_stag = 90 ;  <-- Extra grid point
        seed_dim_stag = 8 ;
.
.
.
```
### Variables
* Each variable is made up of dimensions, attributes, and data values.
* Pay special attention to the units and coordinates attribute.
* The coordinates attribute specifies the variables that contain the latitude and longitude information for each grid box (XLONG, XLAT).
* More recent versions of WRF include an XTIME coordinate.
* The coordinates are named with Fortran ordering, so they'll be listed in reverse.

```console
.
.
.
        float LAKEMASK(Time, south_north, west_east) ;
                LAKEMASK:FieldType = 104 ;
                LAKEMASK:MemoryOrder = "XY " ;
                LAKEMASK:description = "LAKE MASK (1 FOR LAND, 0 FOR WATER)" ;
                LAKEMASK:units = "" ;
                LAKEMASK:stagger = "" ;
                LAKEMASK:coordinates = "XLONG XLAT XTIME" ;
        float SST(Time, south_north, west_east) ;
                SST:FieldType = 104 ;
                SST:MemoryOrder = "XY " ;
                SST:description = "SEA SURFACE TEMPERATURE" ;
                SST:units = "K" ;
                SST:stagger = "" ;
                SST:coordinates = "XLONG XLAT XTIME" ;
.
.
.
```
### Global Attributes
* Provide a description of how the model was set up (resolution, map projection, microphysics, etc)
* For plotting, the map projection parameters will be the most important.
* wrf-python uses this information to build the mapping object in your plotting system of choice - basemap, cartopy, pyngl.

```console
.
.
.
                :CEN_LAT = -0.5240097f ;
                :CEN_LON = 116.637f ;
                :TRUELAT1 = -0.524f ;
                :TRUELAT2 = 0.f ;
                :MOAD_CEN_LAT = -0.5240097f ;
                :STAND_LON = 116.637f ;
                :POLE_LAT = 90.f ;
                :POLE_LON = 0.f ;
                :GMT = 0.f ;
                :JULYR = 2023 ;
                :JULDAY = 139 ;
                :MAP_PROJ = 3 ;
                :MAP_PROJ_CHAR = "Mercator" ;

.
.
.
```

### Reading a WRF File in Python
You have several options to read a WRF NetCDF file in Python.
* netcdf4-python
* PyNIO (Python 3.x available on conda-forge)
* xarray (xarray.Dataset type not currently supported in wrf-python)

### Example 2.1: Using netcdf4-python
```console
from netCDF4 import Dataset

file_path = "/content/drive/MyDrive/Colab Notebooks/WRF_train/wrf_output/wrfout_d01_2023-05-19_00%3A00%3A00"
wrf_file = Dataset(file_path)
print(wrf_file)
```

### Getting Variables and Attributes
* netcdf4-python uses an old API that was originally created for a package called Scientific.IO.NetCDF.
* PyNIO also uses this API.
* xarray does not use this API.
* The API may look a little dated.

### Getting global attributes
The get the full dictionary of global attributes, use the __dict__ attribute.
To work with one attribute at a time, you can use the getncattr and setncattr methods.
```console
global_attrs = wrf_file.__dict__

# To get the value for MAP_PROJ, you can do:
map_proj = wrf_file.__dict__["MAP_PROJ"]

# Or more cleanly
map_proj = wrf_file.getncattr("MAP_PROJ")

# Or for those that know __dict__ is where the class members are stored
map_proj = wrf_file.MAP_PROJ
```
### Getting Variables, Variable Attributes, and Variable Data
All variables are stored in a dictionary attribute called variables.
Let's get the perturbation pressure "P" variable.
```console
# This will return a netCDF4.Variable object
p = wrf_file.variables["P"]
```
To get the variable attributes, you can use the __dict__ attribute to get a dictionary of all attributes.
Use the getncattr function if you already know the attribute name.
```console
# Return a dictionary of all of P's 
# attributes
p_attrs = p.__dict__

# Let's just get the 'coordinates' attribute
p_coords = p.getncattr("coordinates")

# Or using the class attribute directly
p_coords = p.coordinates
```
To get the variable's data as a numpy array, you need to use Python's bracket "[ ]" syntax.
```console
# Get a numpy array for all times
p_all_data = p[:,:,:,:]

# A shorthand version of the above.
p_all_data = p[:]

# This will extract the numpy array for 
# time index 0.
p_t0_data = p[0,:]
```
### Example 2.2: Variables, Attributes, and Data with netcd4-python
```console
from netCDF4 import Dataset

file_path = "/content/drive/MyDrive/Colab Notebooks/WRF_train/wrf_output/wrfout_d01_2023-05-19_00%3A00%3A00"

# Create the netCDF4.Dataset object
wrf_file = Dataset(file_path)

# Get the global attribute dict
global_attrs = wrf_file.__dict__
print ("Global attributes for the file")
print(global_attrs)
print ("\n")

# Just get the 'MAP_PROJ' attribute
map_proj = wrf_file.getncattr("MAP_PROJ")
print ("The MAP_PROJ attribute:")
print (map_proj)
print("\n")

# Get the perturbation pressure variable
p = wrf_file.variables["P"]
print ("The P variable: ")
print(p)
print ("\n")

# Get the P attributes
p_attrs = p.__dict__
print ("The attribute dict for P")
print (p_attrs)
print ("\n")

# Get the 'coordinates' attribute for P
coords = p.getncattr("coordinates")
print ("Coordinates for P:")
print (coords)
print ("\n")

# Get the P numpy array for all times
p_all_data = p[:]
print ("The P numpy array: ")
print (p_all_data)
print ("\n")

# Get the P numpy array for time 0
p_t0_data = p[0,:]
print ("P array at time 0:")
print (p_t0_data)
print ("\n")
```
#
## 3 WRF-Python Functions
WRF-Python provides functionality similar to what is found in the NCL-WRF package:
* Over 30 diagnostics calculations.
* Several interpolation routines (horizontal level, vertical cross section, horizontal "surface").
* Plot helper utilities for cartopy, basemap, and PyNGL.
* WRF-ARW only.

The most commonly used functions:
* getvar: Extracts variables and diagnostic variables.
* interplevel: Linearly interpolates a 3D variable to a horizontal plane at a specified vertical level.
* vertcross: Interpolates a 3D variable to a vertical cross section.
* vinterp: Interpolates a 3D variable to a new surface (e.g. theta-e).

### The getvar function
The getvar function can be used to:
* Extract NetCDF variables from a file, similar to netcdf4-python or PyNIO.
* Compute diagnostic variables.
* Concatenate a variable (either NetCDF or diagnostic) across multiple files.

### Getting the NetCDF HGT Variable with getvar
```console
from netCDF4 import Dataset
from wrf import getvar

file_path = "/content/drive/MyDrive/Colab Notebooks/WRF_train/wrf_output/wrfout_d01_2023-05-19_00%3A00%3A00"
wrf_file = Dataset(file_path)
hgt = getvar(wrf_file, "HGT", timeidx=0)
```
### Example 3.1: Using getvar to Extract a WRF NetCDF Variable
```console
from netCDF4 import Dataset
from wrf import getvar

file_path = "/content/drive/MyDrive/Colab Notebooks/WRF_train/wrf_output/wrfout_d01_2023-05-19_00%3A00%3A00"
wrf_file = Dataset(file_path)
hgt = getvar(wrf_file, "HGT", timeidx=0)
print(hgt)
```
### Computing a Diagnostic Variable with getvar
In this example, we're going to compute sea level pressure.
```console
from netCDF4 import Dataset
from wrf import getvar

file_path = "/content/drive/MyDrive/Colab Notebooks/WRF_train/wrf_output/wrfout_d01_2023-05-19_00%3A00%3A00"
wrf_file = Dataset(file_path)
slp = getvar(wrf_file, "slp", timeidx=0, units="hPa")
```
### Example 3.2: Using getvar to compute Sea Level Pressure (SLP)
Also try changing the units for by specifiying the following values: 'hPa', 'Pa', 'atm', 'mmhg'
```console
from netCDF4 import Dataset
from wrf import getvar

file_path = "/content/drive/MyDrive/Colab Notebooks/WRF_train/wrf_output/wrfout_d01_2023-05-19_00%3A00%3A00"
wrf_file = Dataset(file_path)
slp = getvar(wrf_file, "slp", timeidx=0, units="mmhg")
print(slp)
```
### Combining Across Multiple Files
WRF-Python has two methods for combining a variable across multiple files:
* cat - combines the the variable along the Time dimension (Note: you must order the files yourself)
* join - creates a new left-most dimension for each file
To extract all times in to a single array, set timeidx to wrf.ALL_TIMES (an alias for None).
In this example, we're using the 'cat' method, which is the default.
```console
from netCDF4 import Dataset
from wrf import getvar, ALL_TIMES

file_paths = file_path = ["/content/drive/MyDrive/Colab Notebooks/WRF_train/wrf_output/wrfout_d01_2023-05-19_00%3A00%3A00", "/content/drive/MyDrive/Colab Notebooks/WRF_train/wrf_output/wrfout_d01_2023-05-19_01%3A00%3A00"]
wrf_files = [Dataset(file_paths[0]), Dataset(file_paths[1])]

# Explicitly specifying 'cat', but this is the default
slp = getvar(wrf_files, "slp", timeidx=ALL_TIMES, method="cat")
```
### Example 3.3: Combining Files Using the 'cat' Method
```console
from netCDF4 import Dataset
from wrf import getvar, ALL_TIMES

file_paths = file_path = ["/content/drive/MyDrive/Colab Notebooks/WRF_train/wrf_output/wrfout_d01_2023-05-19_00%3A00%3A00", "/content/drive/MyDrive/Colab Notebooks/WRF_train/wrf_output/wrfout_d01_2023-05-19_01%3A00%3A00"]
wrf_files = [Dataset(file_paths[0]), Dataset(file_paths[1])]

# Explicitly specifying 'cat', but this is the default
slp = getvar(wrf_files, "slp", timeidx=ALL_TIMES, method="cat")
print(slp)
```
### Example 3.4: Combining Files Using the 'join' Method
```console
from netCDF4 import Dataset
from wrf import getvar, ALL_TIMES

file_paths = file_path = ["/content/drive/MyDrive/Colab Notebooks/WRF_train/wrf_output/wrfout_d01_2023-05-19_00%3A00%3A00", "/content/drive/MyDrive/Colab Notebooks/WRF_train/wrf_output/wrfout_d01_2023-05-19_01%3A00%3A00"]
wrf_files = [Dataset(file_paths[0]), Dataset(file_paths[1])]

# Explicitly specifying 'cat', but this is the default
slp = getvar(wrf_files, "slp", timeidx=ALL_TIMES, method="join")
print(slp)
```
### Interpolation Routines
* interplevel - linear interpolation to a horizontal plane at a specified height or pressure level.
* vertcross - vertical cross section interpolation to a vertical plane through two specified points (or a pivot point and angle).
* vinterp - interpolates to a "surface", which could be pressure levels or temperature levels like theta-e. A smarter version of interplevel.
### The interplevel function
* The easiest way to get a field at a specified height or pressure vertical level (500 mb, 5000 m, etc).
* Uses linear interpolation, which is fast and generally good enough for plotting.
* You should use vinterp for geopotential height if you want more accuracy, since the interpolation is done with the decaying exponential pressure profile.
#### interplevel Example
Let's get the 500 hPa geopotential height in decameters
```console
from netCDF4 import Dataset
from wrf import getvar, interplevel

file_path = "/content/drive/MyDrive/Colab Notebooks/WRF_train/wrf_output/wrfout_d01_2023-05-19_00%3A00%3A00"
wrf_file = Dataset(file_path)
pres = getvar(wrf_file, "pressure", timeidx=0)
ht = getvar(wrf_file, "z", timeidx=0, units="dm")
ht_500 = interplevel(ht, pres, 500.0)
```
### Example 3.5: Interpolate to 500 hPa Using interplevel
```console
from netCDF4 import Dataset
from wrf import getvar, interplevel

file_path = "/content/drive/MyDrive/Colab Notebooks/WRF_train/wrf_output/wrfout_d01_2023-05-19_00%3A00%3A00"
wrf_file = Dataset(file_path)
pres = getvar(wrf_file, "pressure", timeidx=0)
ht = getvar(wrf_file, "z", timeidx=0, units="dm")
ht_500 = interplevel(ht, pres, 500.0)
print(ht_500)
```
### The vertcross function
* The idea is to draw a horizontal line at the surface, and the cross section is defined as a vertical plane extending up from this line.
* The new x-axis for a cross section plot is the points along the line you made. The line can be defined by:
  1. defining a start point and an end point by using (x,y) grid coordinates or (latitude, longitude) coordinates.
  2. defining a pivot point and an angle, which is useful for cross sections that will span most of the domain.
* The new y-axis will be a set of vertical levels at default intervals (1% increments), or you can choose them yourself.
### Introducing the CoordPair Class
A _CoordPair_ is simply used to store (x,y) coordinates, or (lat,lon) coordinates. It is also possible to have (x, y, lat, lon), but that's mostly used for metadata.
The _CoordPair_ will be used to define your cross section line.
```console
from wrf import CoordPair

# Creating an x,y pair
x_y_pair = CoordPair(x=10, y=20)

# Creating a lat,lon pair
lat_lon_pair = CoordPair(lat=30.0, lon=-120.0)
```
### vertcross Example
In this example, we're going to define the cross section using a start point and and end point.
We're going to let the algorithm pick the levels, which are at ~1% increments by default.
```console
from netCDF4 import Dataset
from wrf import getvar, vertcross, CoordPair

file_path = "/content/drive/MyDrive/Colab Notebooks/WRF_train/wrf_output/wrfout_d01_2023-05-19_00%3A00%3A00"
wrf_file = Dataset(file_path)

# Making a diagonal cross section line from 
# bottom left to top right.
bottom_left = CoordPair(x=0, y=0)
top_right = CoordPair(x=-1, y=-1)

# Let's get wind speed in kts
wspd_wdir = getvar(wrf_file, "wspd_wdir", timeidx=0, units="kt")           
wspd = wspd_wdir[0,:]

# Get the height levels
ht = getvar(wrf_file, "z", timeidx=0)

# Compute the wind speed cross section
wspd_cross = vertcross(wspd, ht, start_point=bottom_left, end_point=top_right)
```
### Example 3.6: Interpolate to a Vertical Cross Section with vertcross
```console
from netCDF4 import Dataset
from wrf import getvar, vertcross, CoordPair

file_path = "/content/drive/MyDrive/Colab Notebooks/WRF_train/wrf_output/wrfout_d01_2023-05-19_00%3A00%3A00"
wrf_file = Dataset(file_path)

# Making a diagonal cross section line from 
# bottom left to top right.
bottom_left = CoordPair(x=0, y=0)
top_right = CoordPair(x=-1, y=-1)

# Let's get wind speed in kts
wspd_wdir = getvar(wrf_file, "wspd_wdir", timeidx=0, units="kt")           
wspd = wspd_wdir[0,:]

# Get the height levels
ht = getvar(wrf_file, "z", timeidx=0)

# Compute the wind speed cross section
wspd_cross = vertcross(wspd, ht, start_point=bottom_left, end_point=top_right)
print(wspd_cross)
```
### The vinterp function
* Used for interpolating a field to a type of surface:
  * pressure
  * geopotential height
  * theta
  * theta-e
* User must specify the interpolation level(s) on the new surface.
* A smarter, albeit slower and more complicated, version of interplevel.
### vinterp Example
In this example, we're going to interpolate pressure to theta-e levels.
```console
from netCDF4 import Dataset
from wrf import getvar, vinterp 

file_path = "/content/drive/MyDrive/Colab Notebooks/WRF_train/wrf_output/wrfout_d01_2023-05-19_00%3A00%3A00"
wrf_file = Dataset(file_path) 

pres = getvar(wrf_file, "pressure", timeidx=0)

# Interpolate pressure to theta-e levels                 
interp_levels = [280, 285, 290, 292, 294, 
                 296, 298, 300, 305, 310]

pres_eth = vinterp(wrf_file, 
                   field=pres, 
                   vert_coord="theta-e", 
                   interp_levels=interp_levels, 
                   extrapolate=False, 
                   field_type="pressure", 
                   log_p=False)
```
### Example 3.7: Interpolate to Theta-e Levels with vinterp
```console
from netCDF4 import Dataset
from wrf import getvar, vinterp 

file_path = "/content/drive/MyDrive/Colab Notebooks/WRF_train/wrf_output/wrfout_d01_2023-05-19_00%3A00%3A00"
wrf_file = Dataset(file_path) 

pres = getvar(wrf_file, "pressure", timeidx=0)

# Interpolate pressure to theta-e levels                 
interp_levels = [280, 285, 290, 292, 294, 
                 296, 298, 300, 305, 310]

pres_eth = vinterp(wrf_file, 
                   field=pres, 
                   vert_coord="theta-e", 
                   interp_levels=interp_levels, 
                   extrapolate=False, 
                   field_type="pressure", 
                   log_p=False)
print(pres_eth)
```
### Other Useful Functions
### to_np
Converts an xarray.DataArray to a numpy array.
This is often necessary when passing wrf-python variables to the plotting functions or to a compiled extension module.
This routine does the following:
* If no missing/fill values, then it simply calls the xarray.DataArray.values property.
* If missing/fill values are present, then it replaces the NaN values with the fill value found in the attributes, and returns a MaskedArray.
### to_np Example
```console
from netCDF4 import Dataset
from wrf import getvar, to_np 

file_path = "/content/drive/MyDrive/Colab Notebooks/WRF_train/wrf_output/wrfout_d01_2023-05-19_00%3A00%3A00"
wrf_file = Dataset(file_path) 
pres_xarray = getvar(wrf_file, "pressure", timeidx=0)
pres_numpy = to_np(pres_xarray)
```
### xy_to_ll and ll_to_xy
These routines convert to/from grid (x,y) coordinates to/from (lat,lon) coordinates.
Works with a single point or sequences of points.

### xy_to_ll and ll_to_xy Example
```console
from netCDF4 import Dataset
from wrf import getvar, xy_to_ll, ll_to_xy 

file_path = "/content/drive/MyDrive/Colab Notebooks/WRF_train/wrf_output/wrfout_d01_2023-05-19_00%3A00%3A00"
wrf_file = Dataset(file_path)

# Convert (x=20,y=50) and (x=30,y=75) to latitude,longitude
lat_lon = xy_to_ll(wrf_file, [20,30], [50,75])

# Convert back to x,y
x_y = ll_to_xy(wrf_file, lat_lon[0,:], lat_lon[1,:])
```
### Example 3.8: xy_to_ll and ll_to_xy
```console
from netCDF4 import Dataset
from wrf import getvar, xy_to_ll, ll_to_xy 

file_path = "/content/drive/MyDrive/Colab Notebooks/WRF_train/wrf_output/wrfout_d01_2023-05-19_00%3A00%3A00"
wrf_file = Dataset(file_path)

lat_lon = xy_to_ll(wrf_file, [20, 30], [50,75])
print("lat,lon values")
print(lat_lon)
print("\n")

x_y = ll_to_xy(wrf_file, lat_lon[0,:], lat_lon[1,:])
print("x,y values")
print(x_y)
```
#
## 4 Plotting
### Plotting
* Plotting wrf-python variables in Python can be done using either matplotlib or PyNGL.
* For this tutorial, we're going to focus on matplotlib (using cartopy for the mapping).
* As of March 2018, cartopy supports matplotlib 2.x.

### Extremely Brief Overview of Matplotlib
* Under the hood, matplotlib uses an object oriented API.
* In simplest terms, a matplotlib plot consists of Figure object that contains an Axes object (or multiple Axes objects).
* The Axes object is where the action is.
* The Axes object contains the methods for contouring, making histograms, adding colorbar, etc.

### The pyplot API
* Most matplotlib users do no need to know much about the underlying object oriented API.
* Matplotlib includes a series of standalone functions that wrap around the object oriented API.
* These standalone functions are in the matplotlib.pyplot package and it was designed to look similar to the Matlab API.
* New users should start here.
### Example 4.1: Single Wind Barb Example with pyplot
In this example, we are going to plot a single wind barb in the center of the domain using the pyplot API.
```console
from matplotlib import pyplot
import numpy as np

# Make a 5x5 grid of missing u,v values
u = np.ma.masked_equal(np.zeros((5,5)), 0)
v = np.ma.masked_equal(np.zeros((5,5)), 0)

# Add u,v winds to center of domain
u[2,2] = 10.0
v[2,2] = 10.0

# Draw a single wind barb in the middle using pyplot API
# Note:  the axes objects are "hidden" in these functions
fig = pyplot.figure()
pyplot.barbs(u, v)

# Set the x and y ranges so the barb is in the middle
pyplot.xlim(0, 4)
pyplot.ylim(0, 4)

pyplot.show()
```
### Mixing the APIs
* Often you will find yourself mixing the object oriented API with the pyplot API.
* This is required when making subplots, but that is beyond the scope of this tutorial.
* The next example shows how to make the single wind barb using the axes object directly.
```console
from matplotlib import pyplot
import numpy as np

# Make a 5x5 grid of missing u,v values
u = np.ma.masked_equal(np.zeros((5,5)), 0)
v = np.ma.masked_equal(np.zeros((5,5)), 0)

# Add u,v winds to center of domain
u[2,2] = 10.0
v[2,2] = 10.0

# We'll use pyplot to create the figure and 
# get the axes
fig = pyplot.figure()
ax = pyplot.axes() # <- Now we're using the Axes object directly

# Now use the axes directly to create the barbs
ax.barbs(u, v)

# Set the x and y ranges using the axes directly
ax.set_xlim(0, 4)
ax.set_ylim(0, 4)

pyplot.show()
```
### WRF-Python Plotting Helper Functions
wrf-python has several functions to help with plotting when using cartopy, basemap, or PyNGL.
* get_cartopy, get_basemap, get_pyngl: Returns the mapping object used by the plotting system.
* latlon_coords: Returns the latitude and longitude coordinate variables.
* get_bounds: Returns the geographic boundaries for the variable.
### Plotting with cartopy
Cartopy uses the same API as matplotlib by returning a matplotlib.axes.Axes subclass (cartopy.mpl.geoaxes.GeoAxes) when a projection keyword argument is passed to the matplotlib.pyplot.axes function.

### Getting the GeoAxes object
```console
import matplotlib.pyplot
import cartopy.crs

# Create a lat/lon map projection object.
latlon = cartopy.crs.PlateCarree()

geo_axes = pyplot.axes(projection=latlon)
```
### Getting the cartopy Projection Object and Lat/Lon Coordinates Using WRF-Python
* When xarray is installed and enabled, wrf-python carries the projection information around in the metadata of a variable.
* You can use the get_cartopy function to extract the cartopy projection object from a variable.
* You can use the latlon_coords function to get the latitude and longitude points.
```console
from netCDF4 import Dataset
from wrf import (getvar, get_cartopy, latlon_coords)

file_path = "/content/drive/MyDrive/Colab Notebooks/WRF_train/wrf_output/wrfout_d01_2023-05-19_00%3A00%3A00"
wrf_file = Dataset(file_path)

terrain = getvar(wrf_file, "ter", timeidx=0)
cart_proj = get_cartopy(terrain)
lats, lons = latlon_coords(terrain)
```
### Example 4.3: Making a Plot of Terrain
Let's make a plot of terrain. It's the easiest way to check if your map is correct.
```console
import numpy
from matplotlib import pyplot
from matplotlib.cm import get_cmap
from cartopy import crs
from cartopy.feature import NaturalEarthFeature
from netCDF4 import Dataset
from wrf import getvar, to_np, get_cartopy, latlon_coords

file_path = "/content/drive/MyDrive/Colab Notebooks/WRF_train/wrf_output/wrfout_d01_2023-05-19_00%3A00%3A00"
wrf_file = Dataset(file_path)

# Get the terrain height
terrain = getvar(wrf_file, "ter", timeidx=0)

# Get the cartopy object and the lat,lon coords
cart_proj = get_cartopy(terrain)
lats, lons = latlon_coords(terrain)

# Create a figure and get the GetAxes object
fig = pyplot.figure(figsize=(10, 7.5))
geo_axes = pyplot.axes(projection=cart_proj)

# Download and add the states and coastlines
# See the cartopy documentation for more on this.
states = NaturalEarthFeature(category='cultural', 
                             scale='50m', 
                             facecolor='none',
                             name='admin_1_states_provinces_shp')
geo_axes.add_feature(states, linewidth=.5)
geo_axes.coastlines('50m', linewidth=0.8)

# Set the contour levels
levels = numpy.arange(250., 5000., 250.)

# Make the contour lines and fill them.
pyplot.contour(to_np(lons), to_np(lats), 
               to_np(terrain), levels=levels, 
               colors="black",
               transform=crs.PlateCarree())
pyplot.contourf(to_np(lons), to_np(lats), 
                to_np(terrain), levels=levels,
                transform=crs.PlateCarree(),
                cmap=get_cmap("terrain"))
             
# Add a color bar. The shrink often needs to be set 
# by trial and error.
pyplot.colorbar(ax=geo_axes, shrink=.99)

pyplot.show()
```
### Cropping
Sometimes WRF domains are much larger than what you care about.

Plots can be cropped in two ways using wrf-python:
1. Crop the data before plotting.
  * Less data to process = faster!
  * There's a slight risk of issues at borders, but matplotlib seems pretty smart about this.
2. Crop the domain with matplotlib using x,y axis limits.
  * Runs slower since all of the domain is contoured.
  * Should always be correct.
### Method 1: Cropping then Plotting
If you are cropping the data, then cartopy should just work without worrying about setting the axis limits, as long as xarray is installed and enabled.

**Let's start by taking a quick look at the full plot of sea level pressure.**
### First, let's start with a full plot of sea level pressure [no cropping].
### Example 4.4: Full Plot of Sea Level Pressure
```console
import numpy
from matplotlib import pyplot
from matplotlib.cm import get_cmap
from cartopy import crs
from cartopy.feature import NaturalEarthFeature
from netCDF4 import Dataset
from wrf import getvar, to_np, get_cartopy, latlon_coords

file_path = "/content/drive/MyDrive/Colab Notebooks/WRF_train/wrf_output/wrfout_d01_2023-05-19_00%3A00%3A00"
wrf_file = Dataset(file_path)

# Get the terrain height
slp = getvar(wrf_file, "slp", timeidx=0)

# Get the cartopy object and the lat,lon coords
cart_proj = get_cartopy(slp)
lats, lons = latlon_coords(slp)

# Create a figure and get the GetAxes object
fig = pyplot.figure(figsize=(10, 7.5))
geo_axes = pyplot.axes(projection=cart_proj)

# Download and add the states and coastlines
# See the cartopy documentation for more on this.
states = NaturalEarthFeature(category='cultural', 
                             scale='50m', 
                             facecolor='none',
                             name='admin_1_states_provinces_shp')
geo_axes.add_feature(states, linewidth=.5)
geo_axes.coastlines('50m', linewidth=0.8)

# Set the contour levels so that all plots match
levels = numpy.arange(980.,1030.,2.5)

# Make the contour lines and fill them.
pyplot.contour(to_np(lons), to_np(lats), 
               to_np(slp), levels=levels, colors="black",
               transform=crs.PlateCarree())
pyplot.contourf(to_np(lons), to_np(lats), 
                to_np(slp), levels=levels, 
                transform=crs.PlateCarree(),
                cmap=get_cmap("jet"))
             
# Add a color bar. The shrink often needs to be set 
# by trial and error.
pyplot.colorbar(ax=geo_axes, shrink=.86)

pyplot.show()
```
### Example 4.5: Cropping by Slicing the Data
Let's crop the data to the lower right quadrant.
```console
import numpy
from matplotlib import pyplot
from matplotlib.cm import get_cmap
from cartopy import crs
from cartopy.feature import NaturalEarthFeature
from netCDF4 import Dataset
from wrf import getvar, to_np, get_cartopy, latlon_coords

file_path = "/content/drive/MyDrive/Colab Notebooks/WRF_train/wrf_output/wrfout_d01_2023-05-19_00%3A00%3A00"
wrf_file = Dataset(file_path)

# Get the terrain height
slp = getvar(wrf_file, "slp", timeidx=0)

# Determine the center of the domain in grid coordinates
slp_shape = slp.shape
center_y = int(slp_shape[-2]/2.) - 1
center_x = int(slp_shape[-1]/2.) - 1

# Slice from bottom to middle for y
# Slice from middle to right for x
slp_quad = slp[..., 0:center_y+1, center_x:]

# Get the cartopy object and the lat,lon coords
cart_proj = get_cartopy(slp_quad)
lats, lons = latlon_coords(slp_quad)

# Create a figure and get the GetAxes object
fig = pyplot.figure(figsize=(10, 7.5))
geo_axes = pyplot.axes(projection=cart_proj)

# Download and add the states and coastlines
# See the cartopy documentation for more on this.
states = NaturalEarthFeature(category='cultural', 
                             scale='50m', 
                             facecolor='none',
                             name='admin_1_states_provinces_shp')
geo_axes.add_feature(states, linewidth=.5)
geo_axes.coastlines('50m', linewidth=0.8)

# Set the contour levels so that all plots match
levels = numpy.arange(980.,1030.,2.5)

# Make the contour lines and fill them.
pyplot.contour(to_np(lons), to_np(lats), 
               to_np(slp_quad), levels=levels, colors="black",
               transform=crs.PlateCarree())
pyplot.contourf(to_np(lons), to_np(lats), 
                to_np(slp_quad), levels=levels, 
                transform=crs.PlateCarree(),
                cmap=get_cmap("jet"))
             
# Add a color bar. The shrink often needs to be set 
# by trial and error.
pyplot.colorbar(ax=geo_axes, shrink=.83)

pyplot.show()
```
### Method 2: Cropping by Setting x and y Extents
This time, let's crop the domain by using the x and y extents in matplotlib.
Also, we're going to crop the domain using lat,lon geographic boundaries.

### Introducing the GeoBounds class
To create geographic boundaries, you supply a GeoBounds object constructed with set of bottom_left and top_right CoordPair objects.
The CoordPair objects need to use the lat and lon arguments.
```console
from wrf import CoordPair, GeoBounds

bottom_left = CoordPair(lat=29.5, lon=-110)
top_right = CoordPair(lat=30.0, lon=-109.3)

geo_bounds = GeoBounds(bottom_left, top_right)
```
### Setting the Cartopy Extents
After setting up the GeoBounds objects, you can use cartopy_xlim and cartopy_ylim functions to set the extents.
Note: Cartopy also has an API for doing this, but it doesn't work correctly for some projections like RotatedPole.
```console
from wrf import (CoordPair, GeoBounds, getvar, 
                 cartopy_xlim, cartopy_ylim)

file_path = single_wrf_file()
wrf_file = Dataset(file_path)

slp = getvar(wrf_file, "slp", timeidx=0)

bottom_left = CoordPair(lat=29.5, lon=-110)
top_right = CoordPair(lat=30.0, lon=-109.3)

geo_bounds = GeoBounds(bottom_left, top_right)

fig = pyplot.figure(figsize=(10, 7.5))
geo_axes = pyplot.axes(projection=cart_proj)

#. 
#. Draw contours, add geographic features, etc.
#. 

xlim = cartopy_xlim(slp, geobounds=geo_bounds)               
geo_axes.set_xlim(xlim)
         
ylim = cartopy_ylim(slp, geobounds=geo_bounds)               
geo_axes.set_ylim(ylim)
```
### Example 4.6: Cropping by Setting the X,Y Extents
```console
import numpy
from matplotlib import pyplot
from matplotlib.cm import get_cmap
from cartopy import crs
from cartopy.feature import NaturalEarthFeature
from netCDF4 import Dataset
from wrf import getvar, to_np, get_cartopy, latlon_coords
from wrf import xy_to_ll, cartopy_xlim, cartopy_ylim
from wrf import CoordPair, GeoBounds

file_path = "/content/drive/MyDrive/Colab Notebooks/WRF_train/wrf_output/wrfout_d01_2023-05-19_00%3A00%3A00"
wrf_file = Dataset(file_path)

# Get the terrain height
slp = getvar(wrf_file, "slp", timeidx=0)

# Get the cartopy object and the lat,lon coords
cart_proj = get_cartopy(slp)
lats, lons = latlon_coords(slp)

# Create a figure and get the GetAxes object
fig = pyplot.figure(figsize=(10, 7.5))
geo_axes = pyplot.axes(projection=cart_proj)

# Download and add the states and coastlines
# See the cartopy documentation for more on this.
states = NaturalEarthFeature(category='cultural', 
                             scale='50m', 
                             facecolor='none',
                             name='admin_1_states_provinces_shp')
geo_axes.add_feature(states, linewidth=.5)
geo_axes.coastlines('50m', linewidth=0.8)

# Set the contour levels so that all plots match
levels = numpy.arange(980.,1030.,2.5)

# Make the contour lines and fill them.
pyplot.contour(to_np(lons), to_np(lats), 
               to_np(slp), levels=levels, colors="black",
               transform=crs.PlateCarree())
pyplot.contourf(to_np(lons), to_np(lats), 
                to_np(slp), levels=levels, 
                transform=crs.PlateCarree(),
                cmap=get_cmap("jet"))
             
# Add a color bar. The shrink often needs to be set 
# by trial and error.
pyplot.colorbar(ax=geo_axes, shrink=.83)

# Set up the x, y extents

# Determine the center of the domain in grid coordinates
slp_shape = slp.shape
start_y = 0
center_y = int(slp_shape[-2]/2.) - 1
center_x = int(slp_shape[-1]/2.) - 1
end_x = int(slp_shape[-1]) - 1

# Get the lats and lons for the start, center, and end points
# (Normally you would just set these yourself)
center_latlon = xy_to_ll(wrf_file, 
                         [center_x, end_x], 
                         [start_y, center_y])

start_lat = center_latlon[0,0]
end_lat = center_latlon[0,1]
start_lon = center_latlon[1,0]
end_lon = center_latlon[1,1]

# Set the extents
geo_bounds = GeoBounds(CoordPair(lat=start_lat, lon=start_lon),
                       CoordPair(lat=end_lat, lon=end_lon))
geo_axes.set_xlim(cartopy_xlim(slp, geobounds=geo_bounds))
geo_axes.set_ylim(cartopy_ylim(slp, geobounds=geo_bounds))

pyplot.show()
```
