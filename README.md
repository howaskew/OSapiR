# OSapiR
Example R code and a work-in-progress R wrapper for Ordnance Survey APIs

## Sign Up
- Sign up at https://osdatahub.os.uk/ - free access to OS OpenData APIs.
- Create a new project 
- Add the OS Maps API to your new project
- Copy the 'Project API Key' 

## Authentication
Good practice in secure authentication is to set your API credentials as a System Environment variable.  
Replace the 'xxx...' with your copied Project API Key and run this code once.  

`Sys.setenv(OS_PROJECT_API_KEY = 'xxxxxxxxxxxxxxxxxxxxxxxx')`

## Basic example:
Leaflet is one of the most popular open-source JavaScript libraries for interactive maps.   
The 'leaflet' package makes it easy to integrate and control Leaflet maps in R.  
You can bring in OS map tiles by providing a custom URL template to the addTiles() function.  

```
library(leaflet)

m <- leaflet() %>%
  addTiles(paste0("https://api.os.uk/maps/raster/v1/zxy/Light_3857/{z}/{x}/{y}.png?key=",Sys.getenv("OS_PROJECT_API_KEY"))) %>% 
  setView(lng=-1.470770, lat=50.938039,zoom=10) %>%
  addMarkers(lng=-1.470770, lat=50.938039, popup="Explorer House, home of Ordnance Survey")
m  # Print the map
```
