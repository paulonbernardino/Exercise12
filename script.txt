###### WUR Geo-scripting course
### Paulo Bernardino
### February 8th 2017
### Exercise 12

# Import modules
from osgeo import gdal
from osgeo.gdalconst import GA_ReadOnly, GDT_Float32
import numpy as np
import os
import matplotlib.pyplot as plt

# Set WD
os.chdir('/home/ubuntu/Exercise12')

# Merge bands into a file
shellCommand='gdal_merge.py -o /home/ubuntu/Exercise12/data/out.tif -separate /home/ubuntu/Exercise12/data/LC81980242014260LGN00_sr_band3.tif /home/ubuntu/Exercise12/data/LC81980242014260LGN00_sr_band5.tif'
os.system(shellCommand)

# Load data
dataSource=gdal.Open('data/out.tif')

# NDWI function
def ndwiCalc(x):
    # Read data into an array
    band3Arr = x.GetRasterBand(1).ReadAsArray(0,0,x.RasterXSize, x.RasterYSize)
    band5Arr = x.GetRasterBand(2).ReadAsArray(0,0,x.RasterXSize, x.RasterYSize)                                                   
    # Set the data type
    band3Arr=band3Arr.astype(np.float32)
    band5Arr=band5Arr.astype(np.float32)
    # Derive the NDWI
    mask = np.greater(band5Arr+band3Arr,0)
    with np.errstate(invalid='ignore'):
        ndwi = np.choose(mask,(-99,(band3Arr-band5Arr)/(band3Arr+band5Arr)))
    return ndwi    

# Run the function with the provided data
ndwi=ndwiCalc(dataSource)

# Write the result to disk
driver = gdal.GetDriverByName('GTiff')
outDataSet=driver.Create('data/ndwi.tif', dataSource.RasterXSize, dataSource.RasterYSize, 1, GDT_Float32)
outBand = outDataSet.GetRasterBand(1)
outBand.WriteArray(ndwi,0,0)
outBand.SetNoDataValue(-99)

# Set the projection and extent information of the dataset
outDataSet.SetProjection(dataSource.GetProjection())
outDataSet.SetGeoTransform(dataSource.GetGeoTransform())

# Save it
outBand.FlushCache()
outDataSet.FlushCache()

# Via the Shell
shellCommand= 'gdalwarp -t_srs "EPSG:4326" /home/ubuntu/Exercise12/data/ndwi.tif /home/ubuntu/Exercise12/data/ndwi_ll.tif'
os.system(shellCommand)

# Open image
dsll = gdal.Open("data/ndwi_ll.tif")

# Read raster data
ndwi = dsll.ReadAsArray(0, 0, dsll.RasterXSize, dsll.RasterYSize)

# Now plot the raster data using gist_earth palette
plt.imshow(ndwi, interpolation='nearest', vmin=0, cmap=plt.cm.gist_earth)