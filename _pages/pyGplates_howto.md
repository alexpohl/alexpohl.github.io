---
layout: smallfont_page
permalink: /pyGplates_howto/
title: Calculating paleocoordinates with pyGplates
description: Programmatically rotate present-day coordinates back in geological time
nav: false
---

# Overview

This is a very short tutorial showing how to rotate data points back in time based on (1) present-day lon-lat data and (2)  a rotational model (here using the model of Scotese and Wright, 2018).

__Requirements__<br><br>You'll need a python 3.8 install. Tested on MacOS 12.1 only.<br><br>
{: .notice--info}

- Minimum example can be downloaded as a .zip archive [here](../../assets/data/pyGPlates_minimum_example.zip)

- All requested files are available in the archive, including files that should normally be downloaded from the dedicated websites: 
    - the MacOS pyGPlates distribution;
    - Scotese and Right (2018) rotation and polygon files.

To execute the script on MacOS, just open your favorite terminal and type in:
```bash
python3 rotate.py
```

# Python code

Provided in the archive, shown here for illustration.

```python
# python3 rotate.py
# https://pip.pypa.io/en/stable/getting-started/
# python3 -m pip install pandas

# takes around 5 mins to rotate all > 700 000 PBDB entries individually on MacBook Pro i5 2.4GhZ 16Go DDR3
# Dropped 40416/741860 (5.4%) entries that were not correctly rotated
# output: 'Used_pbdbdata_simplified_rotatedScotese.csv'

# https://discourse.gplates.org/t/pygplates-restore-points-to-paleomap/339/6

import sys
sys.path.insert(1, './pygplates_rev28_python38_MacOS64')
import pygplates
from pandas import read_csv
import numpy as np
import os

######################
inPBDB = 'indata.csv'
######################

# reading PBDB data
PBDB_table = read_csv(inPBDB, sep=';', header='infer')
age_min = np.squeeze(np.array(PBDB_table['min_ma'].to_frame()))
age_max = np.squeeze(np.array(PBDB_table['max_ma'].to_frame()))
age_mid = (age_min + age_max) / 2. # each point rotated using mid age
PDlon = np.squeeze(np.array(PBDB_table['lng'].to_frame())) # -180:180 (741865,)
PDlat = np.squeeze(np.array(PBDB_table['lat'].to_frame()))

nPBDB = len(age_min)

# Load one or more rotation files into a rotation model.
rotation_model = pygplates.RotationModel('./PALEOMAP_PlateModel.rot')

# The static polygons file will be used to assign plate IDs and valid time periods.
static_polygons_filename = './PALEOMAP_PlatePolygons.gpml'

# Filename of the output points file that we will write.
# Note that it is a GPML file (has extension '.gpml').
# This enables it to be read into GPlates.
output_points_filename = 'output_points.gpml' # tmp file, removed at the end

input_points = []
for i in np.arange(nPBDB): # nPBDB (replace with e.g. 10000 to test script on a subset of data)
    # Create a pyGPlates point from the latitude and longitude, and add it to our list of points.
    input_points.append(pygplates.PointOnSphere(PDlat[i], PDlon[i]))

# debugging/testing
#input_points = []
#input_points.append(pygplates.PointOnSphere(0, 0))
#input_points.append(pygplates.PointOnSphere(0, 0))
#age_mid = [50, 0]

# Create a feature for each point we read from the input file.
point_features = []
for point in input_points:
    # Create an unclassified feature.
    point_feature = pygplates.Feature()
    # Set the feature's geometry to the input point read from the text file.
    point_feature.set_geometry(point)
    point_features.append(point_feature)

# Use the static polygons to assign plate IDs and valid time periods.
# Each point feature is partitioned into one of the static polygons and assigned its
# reconstruction plate ID and valid time period.
assigned_point_features = pygplates.partition_into_plates(
    static_polygons_filename,
    rotation_model,
    point_features,
    properties_to_copy = [
        pygplates.PartitionProperty.reconstruction_plate_id,
        pygplates.PartitionProperty.valid_time_period])

output_points_filename = 'output_points.gpmlz'
assigned_point_feature_collection = pygplates.FeatureCollection(assigned_point_features)
assigned_point_feature_collection.write(output_points_filename)

# Restore points to a geological time based on rotation model

features = pygplates.FeatureCollection(output_points_filename)

paleolat = np.full((nPBDB),np.nan)
paleolon = np.full((nPBDB),np.nan)

counter = 0
for feature in features:
    reconstruction_time = age_mid[counter]
    paleocoords = [] # possibility to use a list or a string (to save to file)
    pygplates.reconstruct(feature, rotation_model, paleocoords, reconstruction_time)
    if len(paleocoords)>0: # some data points disappear because of rotational model
        # https://www.gplates.org/docs/pygplates/pygplates_reference.html#geometry
        # https://www.gplates.org/docs/pygplates/generated/pygplates.reconstructedfeaturegeometry
        # paleocoords[0].get_present_day_geometry().to_lat_lon_list()
        paleolat[counter] = paleocoords[0].get_reconstructed_geometry().to_lat_lon_list()[0][0]
        paleolon[counter] = paleocoords[0].get_reconstructed_geometry().to_lat_lon_list()[0][1]
    else:
        paleolat[counter] = np.nan
        paleolon[counter] = np.nan
    counter += 1

# adding columns to PBDB table
PBDB_table['paleolon'] = paleolon
PBDB_table['paleolat'] = paleolat

# removing rows with no data
PBDB_table_noNA = PBDB_table.dropna(axis=0, how = 'any', inplace=False)
n_droppedrows = nPBDB - len(PBDB_table_noNA)
percent_droppedrows = np.around((n_droppedrows)/nPBDB*100.,1)
print('Dropped ' + str(int(n_droppedrows)) + '/' + str(int(nPBDB)) + ' (' + str(percent_droppedrows)+ '%) entries that were not correctly rotated')

# writing to csv
outname = 'Used_pbdbdata_simplified_rotatedScotese.csv'
PBDB_table_noNA.to_csv(outname, sep=';')
print('Saving to file: ' + outname)

# cleaning
os.remove(output_points_filename)

print('done')
```
