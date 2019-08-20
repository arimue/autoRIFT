# Autorift
ISCE module for finding pixel displacement and motion velocity between two images over both regular grid in imaging coordinates and user-defined geographic-coordinate grid

Copyright (C) 2019 California Institute of Technology.  Government Sponsorship Acknowledged.

Citation: https://github.com/leiyangleon/geoAutorift

## 1. Authors

Alex Gardner (JPL/Caltech; alex.s.gardner@jpl.nasa.gov) conceived the algorithm and developed the first version in MATLAB;
Yang Lei (GPS/Caltech; ylei@caltech.edu) translated it to Python, further optimized and incoporated to ISCE.
       
       
## 2. Features

* fast algorithm that finds displacement between the two images using sparse search and iteratively progressive chip sizes
* faster than the conventional "ampcor"/"denseampcor" algorithm in ISCE by almost an order of magnitude
* support various preprocessing modes on the given image pair, e.g. either the raw image (both texture and topography) or the texture only (high-frequency components without the topography) can be used with various choices of the high-pass filter options 
* support data format of either unsigned integer 8 (uint8; faster) or single-precision float (float32)
* user can adjust all of the relevant parameters, e.g. search limit, chip size range, etc
* a Normalized Displacement Coherence (NDC) filter is developed to filter image chip displacemnt results based on displacement difference thresholds that are scaled to the search limit
* sparse search is used to first eliminate the unreliable chip displacement results that will not be further used for fine search or following chip size iterations
* novel nested grid design that allows chip size to progress
* another chip size that progresses iteratively is used to determine the chip displacement results that have not been estimated from the previous iterations
* a light interpolation is done to fill the missing (unreliable) chip displacement results using bicubic mode (that can remove pixel discrepancy when using other modes) and an interpolation mask is returned
* the core image processing is coded by calling OpenCV's Python and/or C++ functions for efficiency 
* sub-pixel displacement estimation using the pyramid upsampling algorithm
* this sub-module is not only suitable for radar images, but also for optical, etc
* when the grid is provided in geographic coordinates, all outputs are in the format of GeoTIFF with the same EPSG code as input grid

## 3. Demo

### 3.1 Optical image over regular grid in imaging coordinates

<img src="figures/regular_grid_optical.png" width="100%">

***Output of "autorift" module for a pair of Landsat-8 images (20170708-20170724) in Greenland over a regular-spacing grid: (a) estimated x-direction pixel displacement, (b) estimated y-direction pixel displacement, (c) light interpolation mask, (b) x-direction chip size used.***

This is done by implementing the following command line:

       testAutorift.py -m I1 -s I2 -fo 1

where "I1" and "I2" are the reference and test images as defined in the instructions below. The "-fo" option indicates whether or not to read optical image data.

### 3.2 Radar image over regular grid in imaging coordinates

<img src="figures/regular_grid.png" width="100%">

***Output of "autorift" module for a pair of Sentinel-1A/B images (20170221-20170227; same as the Demo dataset at https://github.com/leiyangleon/geogrid) at Jakobshavn Glacier of Greenland over a regular-spacing grid: (a) estimated x-direction (range) pixel displacement, (b) estimated y-direction (minus azimuth) pixel displacement, (c) light interpolation mask, (b) x-direction chip size used.***


This is obtained by implementing the following command line:

       testAutorift.py -m I1 -s I2

where "I1" and "I2" are the reference and test images as defined in the instructions below. 


### 3.3 Radar image over user-defined geographic-coordinate grid

<img src="figures/autorift1.png" width="100%">

***Output of "autorift" module for a pair of Sentinel-1A/B images (20170221-20170227; same as the Demo dataset at https://github.com/leiyangleon/geogrid) at Jakobshavn Glacier of Greenland over user-defined geographic-coordinate grid (same grid used in the Demo at https://github.com/leiyangleon/geogrid): (a) estimated x-direction (range) pixel displacement, (b) estimated y-direction (minus azimuth) pixel displacement, (c) light interpolation mask, (b) x-direction chip size used. Notes: all maps are established exactly over the same geographic-coordinate grid from input.***

This is done by implementing the following command line:

       testAutorift.py -m I1 -s I2 -g winlocname -o winoffname -vx winro2vxname -vy winro2vyname

where "I1" and "I2" are the reference and test images as defined in the instructions below, and "winlocname", "winoffname", "winro2vxname", "winro2vyname" are the four outputs from running "testGeogrid.py" as defined at https://github.com/leiyangleon/geogrid.

**Runtime comparison for this test (on an OS X system with 2.9GHz Intel Core i7 processor and 16GB RAM):**
* __Autorift: 10 mins__
* __Dense ampcor from ISCE: 90 mins__



<img src="figures/autorift2.png" width="100%">

***Final motion velocity results by combining outputs from "geogrid" (i.e. matrix of conversion coefficients from the Demo at https://github.com/leiyangleon/geogrid) and "autorift" modules: (a) estimated motion velocity from Sentinel-1 data (x-direction; in m/yr), (b) coarse motion velocity from input data (x-direction; in m/yr), (c) estimated motion velocity from Sentinel-1 data (y-direction; in m/yr), (b) coarse motion velocity from input data (y-direction; in m/yr). Notes: all maps are established exactly over the same geographic-coordinate grid from input.***


## 4. Install

* First install ISCE
* Put the "geoAutorift" folder and the "Sconscript" file under the "contrib" folder that is one level down ISCE's source directory (denoted as "isce-version"; where you started installing ISCE), i.e. "isce-version/contrib/" (see the snapshot below)

<img src="figures/install_snapshot.png" width="35%">

* run "scons install" again from ISCE's source directory "isce-version" using command line


## 4. Instructions


* When the grid is provided in geographic coordinates, it is recommended to run the "geogrid" module (https://github.com/leiyangleon/geogrid) first before running "autorift". In other words, the outputs from "testGeogrid.py" (a.k.a "winlocname", "winoffname", "winro2vxname", "winro2vyname") will serve as the inputs for running "autorift" or will be required to generate the final motion velocity maps.

For quick use:
* Refer to the file "testAutorift.py" for the usage of the module and modify it for your own purpose
* Input files include the reference image (required), test image (required), and the four outputs from running "testGeogrid.py" (a.k.a "winlocname", "winoffname", "winro2vxname", "winro2vyname"). 

_Note: if the four outputs from running the "geogrid" module are not provided, a regular grid will be assigned_

* Output files include 1) estimated x-direction displacement (equivalent to range for radar), 2) estimated y-direction displacement (equivalent to minus azimuth for radar), 3) light interpolation mask, 4) iteratively progressive chip size used in x direction. 

_Note: These four output files will be stored in a file named "offset.mat" that can be viewed in Python and MATLAB. When the grid is provided in geographic coordinates, a 4-band GeoTIFF with the same EPSG code as input grid will be created as well and named "offset.tif"; a 2-band GeoTIFF consisting of the final converted motion velocity in geographic x- and y-coordinates will be created and named "velocity.tif"._

For modular use:
* In Python environment, type the following to import the "autorift" module and initialize the "autorift" object

       import isce
       from components.contrib.geoAutorift.autorift.Autorift import Autorift
       obj = Autorift()
       obj.configure()

* The "autorift" object has several inputs that have to be assigned (listed below; can also be obtained by referring to "testAutorift.py"): 
       
       ------------------input------------------
       I1:                  reference image
       I2:                  test image (displacement = motion vector of I2 relative to I1)
       xGrid:               x-direction pixel index at each grid point
       yGrid:               y-direction pixel index at each grid point
       (if xGrid and yGrid not provided, a regular grid spanning the entire image will be automatically set up, which is similar to the conventional ISCE module, "ampcor" or "denseampcor")
       Dx0:                 x-direction coarse displacement at each grid point
       Dy0:                 y-direction coarse displacement at each grid point
       (if Dx0 and Dy0 not provided, an array with zero values will be automatically assigned)

* After the inputs are specified, run the module as below
       
       obj.preprocess_filt_XXX() or obj.preprocess_db()
       obj.uniform_data_type()
       obj.runAutorift()

where "XXX" can be "wal" for the Wallis filter, "hps" for the trivial high-pass filter, "sob" for the Sobel filter, "lap" for the Laplacian filter, and also a logarithmic operator without filtering is adopted for occasions where low-frequency components (i.e. topography) are desired, i.e. "obj.preprocess_db()".

* The "autorift" object has the following four outputs: 
       
       ------------------output------------------
       Dx:                  estimated displacement in x-direction
       Dy:                  estimated displacement in y-direction
       InterpMask:          light interpolation mask
       ChipSizeX:           iteratively progressive chip size used in x-direction (different chip sizes allowed for x and y)

* The "autorift" object has many parameters that can be flexibly tweaked by the users for their own purpose (listed below; can also be obtained by referring to "geoAutorift/autorift/Autorift.py"):

       ------------------parameter list: general function------------------
       ChipSizeMinX:               Minimum size (in X direction) of the image template (chip) to correlate (default = 32; could be scalar or array with same dimension as xGrid)
       ChipSizeMaxX:               Maximum size (in X direction) of the image template (chip) to correlate (default = 64; could be scalar or array with same dimension as xGrid)
       ChipSize0X:                 Minimum acceptable size (in X direction) of the image template (chip) to correlate (default = 32)
       ScaleChipSizeY              Scaling factor to get the Y-directed chip size in reference to the X-directed sizes (default = 1)
       SearchLimitX                Range (in X direction) to search for displacement in the image (default = 25; could be scalar or array with same dimension as xGrid; when provided in array, set its elements to 0 if excluded for finding displacement)
       SearchLimitY                Range (in Y direction) to search for displacement in the image (default = 25; could be scalar or array with same dimension as xGrid; when provided in array, set its elements to 0 if excluded for finding displacement)
       SkipSampleX                 Number of samples to skip between windows in X direction for automatically creating the grid if not specified by the user (default = 32)
       SkipSampleY                 Number of lines to skip between windows in Y direction for automatically creating the grid if not specified by the user (default = 32)
       minSearch                   Minimum search limit (default = 6)
       
       ------------------parameter list: about Normalized Displacement Coherence (NDC) filter ------------------
       FracValid                   Fraction of valid displacements (default = 8/25) to be multiplied by filter window size "FiltWidth^2" and then used for thresholding the number of chip displacements that have passed the "FracSearch" disparity filtering
       FracSearch                  Fraction of search limit used as threshold for disparity filtering of the chip displacement difference that is normalized by the search limit (default = 0.25)
       FiltWidth                   Disparity filter width (default = 5)
       Iter                        Number of iterations (default = 3)
       MadScalar                   Scalar to be multiplied by Mad used as threshold for disparity filtering of the chip displacement deviation from the median (default = 4)
       
       ------------------parameter list: miscellaneous------------------
       WallisFilterWidth:          Width of the filter to be used for the preprocessing (default = 21)
       fillFiltWidth               Light interpolation filling filter width (default = 3)
       sparseSearchSampleRate      downsampling rate for sparse search  (default = 4)
       BuffDistanceC               Buffer coarse correlation mask by this many pixels for use as fine search mask (default = 8)
       CoarseCorCutoff             Coarse correlation search cutoff (default = 0.01)
       OverSampleRatio             Factor for pyramid upsampling for sub-pixel level offset refinement (default = 16)
       DataTypeInput               Image data type: 0 -> uint8, 1 -> float32 (default = 0)
       zeroMask                    Force the margin (no data) to zeros which is useful for Wallis-filter-preprocessed images (default = None; 1 for Wallis filter)
