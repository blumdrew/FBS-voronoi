"""
Various stats about map made from makemap.py
Just supports area at the moment
"""


import os
import math

import geopandas as gpd
from shapely.geometry.base import BaseGeometry

import makemap

current_folder = os.getcwd()
vm = makemap.voronoi_mercator()
geoms = makemap.make_geoms(vm, makemap.read_gis())

data = makemap.plot_merc(geoms, save_plot = False)
data = data.drop(0)
print('Succesfully imported data')
alaska_owner = 'Washington'

def set_owners(file = ''):
    #a function to set which school owns which 
    return

def get_us_area(file = 'cb_2017_us_state_5m.shp'):
    os.chdir(current_folder + '/' +file[:-4])
    data = gpd.read_file(file).to_crs(epsg = 3395)
    os.chdir(current_folder)

    return data.unary_union.area

def area(data_frame, include_alaska = False):
    data_frame['area km2'] = None
    data_frame['area sqmile'] = None
    ak = makemap.alaska(return_data = True)
    usa_area = get_us_area()
    all_area = []
    usa_km2, usa_sqmile = 9834000, 3797000

    for index in range(135):
        try:
            school = data_frame.loc[index]['school']
            geom_area = data_frame.loc[index]['geometry'].area
            if school == alaska_owner and include_alaska == True:
                geom_area += ak.loc[1]['geometry'].area
            area = geom_area/usa_area
            data_frame.loc[index, 'area km2'] = area * usa_km2
            data_frame.loc[index, 'area sqmile'] = area * usa_sqmile
            all_area.append((int(area * usa_sqmile), school))
        except:
            True

    #sort by area
    all_area = sorted(all_area, key = lambda x: x[0], reverse = True)
    

    return all_area
