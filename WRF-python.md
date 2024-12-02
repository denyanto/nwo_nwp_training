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
## 1.1 Introduction to jupyter, numpy, xarray
### What is Jupyter Notebook?
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
## 1.1 Verifying your Jupyter Environment
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
