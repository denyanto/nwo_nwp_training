# WRF-Python Tutorial 2024
### Danang Eko Nuryanto 
#### Indonesian Agency for Meteorology Climatology and Geopgysics (BMKG)
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
...

```

### Dimensions
* WRF-ARW uses an Arakawa C-grid staggered grid [(taken from mmm website)] 1
* Mass related quantities (pressure, temperature, etc) are computed at the center of a grid cell.
* The u-component of the horizontal wind is calculated at the left and right edges of a grid cell. It has one more point in the x direction than the mass grid.
* The v-component of the horizontal wind is calculated at the bottom and top edges of a grid cell. It has one more point in the y direction than the mass grid.
* The corners of each grid box are know as the 'staggered' grid, and it has one additional point in both the x and y direction.
```console

```


```console

```


```console

```


```console

```


```console

```
