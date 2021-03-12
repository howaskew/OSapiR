# OSapiR
Example R code and a work-in-progress R wrapper for Ordnance Survey APIs

## Sign Up
- Sign up at https://osdatahub.os.uk/ - free access to OS OpenData APIs.
- Create a new project 
- Add the OS Maps API to your new project
- Copy the 'Project API Key' 

## Authentication
Good practice in secure authentication is to set your API credentials as a System Environment variable.  
Replace the 'xxx...' with your copied Project API Key and run this code.  

`Sys.setenv(OS_PROJECT_API_KEY = 'xxxxxxxxxxxxxxxxxxxxxxxx')`

Or, add OS_PROJECT_API_KEY=xxxxxxxxxxxxxxxxxxxxxxxx (note no quotes) to your .Renviron file to persist the key between sessions.  
See https://bookdown.org/csgillespie/efficientR/set-up.html#r-startup for more info on .Renviron

## Basic example:
Leaflet is one of the most popular open-source JavaScript libraries for interactive maps.   
The 'leaflet' package makes it easy to integrate and control Leaflet maps in R.  
You can bring in OS map tiles by providing a custom URL template to the addTiles() function.  

```
library(leaflet)

m <- leaflet() %>%
  addTiles(paste0("https://api.os.uk/maps/raster/v1/zxy/Light_3857/{z}/{x}/{y}.png?key=",Sys.getenv("OS_PROJECT_API_KEY"))) %>% 
  addMarkers(lng=-1.470770, lat=50.938039, popup="Explorer House, home of Ordnance Survey")
m  # Print the map
```

## Varying map tile styles

```
#Define url templates for the different tile styles
OSLight_3857 <- paste0("https://api.os.uk/maps/raster/v1/zxy/Light_3857/{z}/{x}/{y}.png?key=",Sys.getenv("OS_PROJECT_API_KEY"))
OSRoad_3857 <- paste0("https://api.os.uk/maps/raster/v1/zxy/Road_3857/{z}/{x}/{y}.png?key=",Sys.getenv("OS_PROJECT_API_KEY"))
OSOutdoor_3857 <- paste0("https://api.os.uk/maps/raster/v1/zxy/Outdoor_3857/{z}/{x}/{y}.png?key=",Sys.getenv("OS_PROJECT_API_KEY"))

#Pass these templates to the addTiles() function
m <- leaflet() %>%
  addTiles(OSLight_3857) %>% 
  addMarkers(lng=-1.470770, lat=50.938039, popup="Explorer House, home of Ordnance Survey")
m  # Print the map

m <- leaflet() %>%
  addTiles(OSRoad_3857) %>% 
  addMarkers(lng=-1.470770, lat=50.938039, popup="Explorer House, home of Ordnance Survey")
m  # Print the map

m <- leaflet() %>%
  addTiles(OSOutdoor_3857) %>% 
  addMarkers(lng=-1.470770, lat=50.938039, popup="Explorer House, home of Ordnance Survey")
m  # Print the map
```


## Changing the map projection
Leaflet expects all point, line, and shape data to be specified in latitude and longitude using WGS 84 (a.k.a. EPSG:4326).   
By default, when displaying this data it projects everything to EPSG:3857 and expects that any map tiles are also displayed in EPSG:3857.  
However, you can use custom projections via the integrated Proj4Leaflet plugin.  
For more detail on custom projections, see https://rstudio.github.io/leaflet/projections.html 

```
#Define a Proj4Leaflet coordinate reference system (crs) instance configured for British National Grid (EPSG:27700) and the resolutions of our base map
crs <- leafletCRS(crsClass = "L.Proj.CRS", code = "EPSG:27700",
                  proj4def = '+proj=tmerc +lat_0=49 +lon_0=-2 +k=0.9996012717 +x_0=400000 +y_0=-100000 +ellps=airy +towgs84=446.448,-125.157,542.06,0.15,0.247,0.842,-20.489 +units=m +no_defs',
                  resolutions = c(896.0, 448.0, 224.0, 112.0, 56.0, 28.0, 14.0, 7.0, 3.5, 1.75 ),
                  origin = c(-238375.0, 1376256.0)
                  )

#Define url templates for the different tile styles in EPSG:27700 
OSRoad_27700 <- paste0("https://api.os.uk/maps/raster/v1/zxy/Road_27700/{z}/{x}/{y}.png?key=",Sys.getenv("OS_PROJECT_API_KEY"))
OSLight_27700 <- paste0("https://api.os.uk/maps/raster/v1/zxy/Light_27700/{z}/{x}/{y}.png?key=",Sys.getenv("OS_PROJECT_API_KEY"))
OSOutdoor_27700 <- paste0("https://api.os.uk/maps/raster/v1/zxy/Outdoor_27700/{z}/{x}/{y}.png?key=",Sys.getenv("OS_PROJECT_API_KEY"))
OSLeisure_27700 <- paste0("https://api.os.uk/maps/raster/v1/zxy/Leisure_27700/{z}/{x}/{y}.png?key=",Sys.getenv("OS_PROJECT_API_KEY"))

#Pass these templates to the addTiles() function, along with the new crs
m <- leaflet(options=leafletOptions(crs=crs)) %>%
  addTiles(OSRoad_27700) %>% 
  addMarkers(lng=-1.470770, lat=50.938039, popup="Explorer House, home of Ordnance Survey")
m  # Print the map

m <- leaflet(options=leafletOptions(crs=crs)) %>%
  addTiles(OSOutdoor_27700) %>% 
  addMarkers(lng=-1.470770, lat=50.938039, popup="Explorer House, home of Ordnance Survey")
m  # Print the map

m <- leaflet(options=leafletOptions(crs=crs)) %>%
  addTiles(OSLight_27700) %>% 
  addMarkers(lng=-1.470770, lat=50.938039, popup="Explorer House, home of Ordnance Survey")
m  # Print the map

#Add a max zoom parameter, given the scale limitation for the Leisure tiles 
m <- leaflet(options=leafletOptions(crs=crs)) %>%
  addTiles(OSLeisure_27700,options=tileOptions(maxZoom = 5)) %>% 
  addMarkers(lng=-1.470770, lat=50.938039, popup="Explorer House, home of Ordnance Survey")
m  # Print the map
```
