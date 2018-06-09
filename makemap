'''
Create the map! 
'''
import os
import math

import geopandas as gpd
from scipy.spatial import Voronoi
from shapely.geometry import Point, Polygon, mapping
from shapely.geometry.base import BaseGeometry
import matplotlib.pyplot as plt
from PIL import Image

X_RANGE = (-130, -60)
Y_RANGE = (25, 50)
current_folder = os.getcwd()

#read location data from FBSdata.txt, return it into a tuple
#not used, but left in since it was used to find mercator coords
def read_locations(file = 'FBSdata.txt'):
    team_names, stadium_names, stadium_latlon, = [], [], []
    with open(file, 'r') as f:
        index = 0
        for line in f:
            if index != 0:
                lsplit = line.split(',', 2)
                team_names.append(lsplit[0])
                stadium_names.append(lsplit[1])
                lat = float(lsplit[2].split(',')[0][2:])
                lon = float(lsplit[2].split(',')[1][1:-1])
                stadium_latlon.append((lat, lon))
            index += 1

    all_data = [team_names, stadium_names, stadium_latlon]

    return all_data
        
#use mercator proj to since it should preserve straight lines
def read_gis(file = 'cb_2017_us_state_5m.shp'):
    current_folder = os.getcwd()
    os.chdir(current_folder + '/' +file[:-4])
    data = gpd.read_file(file).to_crs(epsg = 3395)
    #only want lower 48 states + hawaii, plot for hawaii and alaska seperately
    data = data.drop([1,44,45,46,49,50])
    os.chdir(current_folder)
    return data


#used https://mygeodata.cloud/cs2cs/ to convert from lat/lon to mercator
#create the trusty voronoi tesselation
def voronoi_mercator(file = 'mercator_coords.txt'):

    vframe = gpd.GeoDataFrame()
    vframe['school'] = None
    vframe['points'] = None

    with open(file) as f:
        merc_data = []
        for line in f:
            line_data = line.split(',')
            if line_data[0] != 'TeamName':
                school = line_data[0]
                x = float(line_data[1][1:])
                y = float(line_data[2][1:-2])
                merc_data.append((school, (x, y)))

    points = [item[1] for item in merc_data]
    #need to add the four extra points far away to deal with points at infinity
    #created in scipy.spatial.Voronoi
    points.append((-100000000,-100000000))
    points.append((-100000000,100000000))
    points.append((100000000,-100000000))
    points.append((100000000,100000000))

    vor = Voronoi(points)
    regions = vor.regions
    pts = vor.vertices

    i = 0
    for part in regions:
        loop_points = []
        #ignore any shape with a -1 (pt at infinity) since those are from the 4 extra pts
        if -1 not in part:
            loop_points = [pts[part[index]] for index in range(len(part))]
            try:
                poly = Polygon(loop_points)
                vframe.loc[i, 'points'] = poly
            except ValueError:
                vframe.loc[i, 'points'] = None
                
            nested_index = 0
            for point in points:
                if poly.contains(Point(point)):
                    vframe.loc[i, 'school'] = merc_data[nested_index][0]
                    break
                nested_index += 1
        i += 1
        
    return vframe
    
#geo_frame is the frame from the US
def make_geoms(vframe, geo_frame):
    vframe['geometry'] = None
    poly = geo_frame.unary_union

    for index in range(135):
        try:
            loop_geom = BaseGeometry.intersection(vframe.loc[index]['points'].buffer(0), poly.buffer(0))
            vframe.loc[index, 'geometry'] = loop_geom
        except:
            True

    #set projection as mercator for the new data frame
    vframe.crs = {'init':'epsg = 3395'}

    return vframe

#search thru txt file to find color (hex)
def get_colors(file = 'colors.txt'):
    team_colors = []
    with open(file,'r') as data:
        for line in data:
            line_data = line.split(',')
            if line_data[0] != '':
                current_team = line_data[0]
            if line_data[-1] != '' or '\n':
                current_color = '#' + line_data[-1][:-1]
                team_colors.append((current_team, current_color))
    return team_colors


def dist(x1, x2):
    return math.sqrt((x1[0]-x2[0])**2 + (x1[1]-x2[1])**2)

#return something close to span of polygon, doesnt have to be exact
def poly_span(poly):
    mapper = mapping(poly)
    if mapper['type'] == 'Polygon':
        points = mapper['coordinates'][0]
    else:
        pt_list = mapper['coordinates']
        points = [item for sublist in pt_list for item in sublist[0]]
        
    start_point = poly.centroid.coords[0]
    largest_dist = 0
    for point in points:
        loop_dist = dist(start_point, point)
        if loop_dist > largest_dist:
            largest_dist = loop_dist
            
    return largest_dist

#make a plot of just alaska
def alaska(file = 'cb_2017_us_state_5m.shp', owned_by = 'Washington', return_data = False):
    current_folder = os.getcwd()
    os.chdir(current_folder + '/' +file[:-4])
    data = gpd.read_file(file).to_crs(epsg = 4326)
    #only want Alaska (2)
    data = data.drop([0, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20,
                      21, 22, 23, 24, 25, 26, 27, 28, 29, 30, 31, 32, 33, 34, 35,
                      36, 37, 38, 39, 40, 41, 42, 43, 44, 45, 46, 47, 48, 49, 50,
                      51, 52, 53, 54, 55])
    os.chdir(current_folder)
    data = data.to_crs(epsg = 3395)

    #make a plot of alaska, with proper color and logo
    folder = os.getcwd() + '/team logos'
    os.chdir(folder)
    fname = owned_by + '.png'
    img = Image.open(fname)
    os.chdir(current_folder)
    colors = get_colors()
    color = ''
    for team in colors:
        if owned_by == team[0]:
            color = team[1]
            break
    if color == '':
        print('Unable to find color...')
        color = '#ffffff'
       
    x, y = (-17000000, 9500000)
    ax = plt.gca()
    ax.set_xlim((-20000000,-14200000))
    ax.set_ylim((6000000,12000000))
    ax.set_aspect('equal')
    ax.axis('off')
    xscale, yscale = 1000000, 1000000
    gpd.plotting.plot_series(data, ax = ax, color = color, zorder = 0)
    plt.imshow(img, extent = (x - xscale, x + xscale, y - yscale, y + yscale),
                       interpolation = 'nearest', zorder = 10)
    plt.savefig('alaska.png',transparent = True)
    
    if return_data == True:
        return data

#make a plot of just hawaii
def hawaii(file = 'cb_2017_us_state_5m.shp', owned_by = 'Hawaii'):
    current_folder = os.getcwd()
    os.chdir(current_folder + '/' +file[:-4])
    data = gpd.read_file(file).to_crs(epsg = 4326)
    #only want Hawaii (28)
    data = data.drop([0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20,
                      21, 22, 23, 24, 25, 26, 27, 29, 30, 31, 32, 33, 34, 35,
                      36, 37, 38, 39, 40, 41, 42, 43, 44, 45, 46, 47, 48, 49, 50,
                      51, 52, 53, 54, 55])
    os.chdir(current_folder)
    data = data.to_crs(epsg = 3395)

    #create a plot
    folder = os.getcwd() + '/team logos'
    os.chdir(folder)
    fname = owned_by + '.png'
    img = Image.open(fname)
    os.chdir(current_folder)
    colors = get_colors()
    color = ''
    for team in colors:
        if owned_by == team[0]:
            color = team[1]
            break
    if color == '':
        print('Unable to find color...')
        color = '#ffffff'
      
    x, y = (-17400000, 2300000)
    ax = plt.gca()
    ax.set_xlim((-17900000,-17200000))
    ax.set_ylim((2100000,2600000))
    ax.set_aspect('equal')
    ax.axis('off')
    xscale, yscale = 100000, 100000
    gpd.plotting.plot_series(data, ax = ax, color = color, zorder = 0)
    plt.imshow(img, extent = (x - xscale, x + xscale, y - yscale, y + yscale),
                      interpolation = 'nearest', zorder = 10)
    plt.savefig('hawaii.png',transparent = True)
    
    return

#create plot for rest of the states
def plot_merc(vframe, save_plot = True, ak_owner = 'Washington', hi_owner = 'Hawaii'):
    vframe['color'] = None
    vframe['center'] = None
    vframe['logo'] = None
    vframe['owner'] = None
    
    #set owner
    alaska(owned_by = ak_owner)
    hawaii(owned_by = hi_owner)
    center_x, center_y = [],[]
    #find the center of each shape, to place the image of team logo
    for item in range(135):
        try:
            center = vframe.loc[item]['geometry'].centroid.coords[0]
            vframe.loc[item, 'center'] = center
            center_x.append(center[0])
            center_y.append(center[1])
        except KeyError:
            True
        except IndexError:
            True
            
    #add logo image to vframe
    folder = os.getcwd() + '/team logos'
    os.chdir(folder)
    for item in range(135):
        try:
            team_name = vframe.loc[item]['school']
            fname = team_name + '.png'
            loop_image = Image.open(folder + '/' + fname)
            loop_image.thumbnail((512,512), Image.ANTIALIAS)
            vframe.loc[item, 'logo'] = loop_image
        except KeyError:
            True
        except IndexError:
            True
        except TypeError:
            True
    os.chdir(current_folder)
        

    #set range, create outline of states
    X_RANGE = (-14000000,-7240000)
    Y_RANGE = (2650000, 6500000)
    us_data = read_gis().to_crs(epsg=3395)

    #add colors to vframe from txt file
    for index in range(135):
        color_data = get_colors()
        for color in color_data:
            try:
                if vframe.loc[index]['school'] == color[0]:
                    vframe.loc[index, 'color'] = color[1]
                    break
            except:
                True

    #if any school isn't found in colors.txt, assign a color
    for index in range(135):
        try:
            if vframe.loc[index]['color'] == None:
                vframe.loc[index]['color'] = '#f47321'
        except:
            True

    #set axes, add images
    fig, ax = plt.subplots()
    plt.figure(dpi = 1000)
    ax = plt.gca()
    ax.set_xlim(X_RANGE)
    ax.set_ylim(Y_RANGE)
    ax.set_aspect('equal')
    ax.axis('off')
    for index in range(135):
        try:
            shape = vframe.loc[index]['geometry']
            span = poly_span(shape) / 2
            #ugly logos that needed to be enlarged
            if vframe.loc[index]['school'] == 'Penn State':
                span *= 2
            elif vframe.loc[index]['school'] == 'Pittsburgh':
                span *= 2
            elif vframe.loc[index]['school'] == 'Maryland':
                span *= 2
            elif vframe.loc[index]['school'] == 'Akron':
                span *= 2
            elif vframe.loc[index]['school'] == 'Wake Forest':
                span *= 2
            elif vframe.loc[index]['school'] == 'Georgia Tech':
                span *= 2
            elif vframe.loc[index]['school'] == 'Virginia':
                span *= 2
            elif vframe.loc[index]['school'] == 'East Carolina':
                span *= 2
            elif vframe.loc[index]['school'] == 'Coastal Carolina':
                span *= 2
            elif vframe.loc[index]['school'] == 'Vanderbilt':
                span *= 2
            elif vframe.loc[index]['school'] == 'Notre Dame':
                span *= 2
            elif vframe.loc[index]['school'] == 'Purdue':
                span *= 2
            elif vframe.loc[index]['school'] == 'Missouri':
                span *= 2
            elif vframe.loc[index]['school'] == 'Tennessee':
                span *= 2
            elif vframe.loc[index]['school'] == 'Marshall':
                span *= 2
            elif vframe.loc[index]['school'] == 'Kentucky':
                span *= 2
                
            image = vframe.loc[index]['logo']
            x, y = vframe.loc[index]['center'][0], vframe.loc[index]['center'][1]
            width, height = image.size
            xscale, yscale = width*span/700, height*span/700
            plt.imshow(image, extent = (x - xscale, x + xscale, y - yscale, y + yscale),
                       interpolation = 'nearest', zorder = 10)
        except:
            True
    
    #add each shape to the plot -> individually so color can be fully customized
    for index in range(len(vframe)):
        gpd.plotting.plot_series(vframe[index:index+1], ax=ax, color = vframe[index:index+1]['color'], zorder = 0)

    #add frame of US states
    us_data.plot(ax=ax, edgecolor = 'black', facecolor = 'None', linewidth = 0.1)

    #add alaska and hawaii to map

    ak_img = Image.open('alaska.png')
    hi_img = Image.open('hawaii.png')
    ak_extent = (-14000000, -14000000 + 2500*ak_img.size[0], 2500000, 2500000 + 2500*ak_img.size[0])
    hi_extent = (-12800000, -12800000 + 1250*hi_img.size[0], 2800000, 2800000 + 1250*hi_img.size[0])

    plt.imshow(ak_img, extent = ak_extent, interpolation = 'nearest', zorder = 11)
    plt.imshow(hi_img, extent = hi_extent, interpolation = 'nearest', zorder = 11)

    if save_plot:
        plt.savefig('FBSvoronoi.png',bbox_inches = 'tight')

    return vframe
    
def main():
    
    vm = voronoi_mercator()
    geoms = make_geoms(vm, read_gis().to_crs(epsg=3395))
    vm = plot_merc(geoms)
