# OSapiR
Example R code and a work-in-progress R wrapper for Ordnance Survey APIs

## Sign Up
- Sign up at https://osdatahub.os.uk/ - free access to OS OpenData APIs and up to £1000 of premium data per calendar month.
- Create a new project 
- Add the OS Maps API to your new project (for the basic and advanced examples below)
- Add the OS Features API to your project (for the advanced example below - requires access to premium data)
- Copy the 'Project API Key' 

## Authentication
Good practice in secure authentication is to set your API credentials as a System Environment variable.  
Replace the 'xxx...' with your copied Project API Key and run this code.  

`Sys.setenv(OS_PROJECT_API_KEY = 'xxxxxxxxxxxxxxxxxxxxxxxx')`

Or, add OS_PROJECT_API_KEY=xxxxxxxxxxxxxxxxxxxxxxxx (note no quotes) to your .Renviron file to persist the key between sessions.  
See https://bookdown.org/csgillespie/efficientR/set-up.html#r-startup for more info on .Renviron

## Basic example: Adding backdrops or base map tiles to leaflet maps 
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

### Varying map tile styles

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

### Changing the map projection
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

#I've added a max zoom parameter here, given the scale limitation for the OpenData Leisure tiles   
m <- leaflet(options=leafletOptions(crs=crs)) %>%
  addTiles(OSLeisure_27700,options=tileOptions(maxZoom = 5)) %>% 
  addMarkers(lng=-1.470770, lat=50.938039, popup="Explorer House, home of Ordnance Survey")
m  # Print the map
```
  
## More advanced example: Show features of a given type within the map bounds 
The code below creates a simple Shiny web application with an interactive leaflet map.  
As you move the map around, the app queries the OS Features API for a specific feature type - Airports.  
These are added to the map as polygons.
```
library(shiny) #Interactive R applications
library(leaflet) #Wrapper for the popular javascript mapping library 
library(httr) #Functions for working with http
library(sf) #Functions for handling geospatial data - simple features

#Set custom url template for OS map for backdrop
OSRoad_3857 <- paste0("https://api.os.uk/maps/raster/v1/zxy/Road_3857/{z}/{x}/{y}.png?key=",Sys.getenv("OS_PROJECT_API_KEY"))
#Base URL for web feature service
wfsServiceUrl <- "https://api.os.uk/features/v1/wfs"

#Define app front-end
ui <- fluidPage(
  #Style map to fill the screen
  tags$style(type = "text/css", "#map {height: calc(100vh) !important;}"),
  #Show the map
  leafletOutput("map")
)

#Define app server functions
server <- function(input, output, session) {
  
  #Show the base map
  output$map <- renderLeaflet({
    leaflet() %>%
      addTiles(OSRoad_3857) %>% setView(lng=-1.470770, lat=50.938039,zoom=12)
  })

  #Create a reactive element that updates when the map is moved
  coords <- reactive(coordinates <- paste0(input$map_bounds$south,",",input$map_bounds$west," ",
                            input$map_bounds$north,",",input$map_bounds$east))  
  observe({
    #When map moves, update the boundary coordinates
    #Could add a strategy to only search in new bounds areas, but keeping it simple for now
    if(coords()!=", ,") {
    #Create an OGC XML filter parameter value which will select features intersecting the bounds coordinates
    # where the site function equals "Airport" 
    xml <- paste0('<ogc:Filter>',
                  '<ogc:And>',
                  '<ogc:BBOX>',
                  '<ogc:PropertyName>SHAPE</ogc:PropertyName>',
                  '<gml:Box srsName="urn:ogc:def:crs:EPSG::4326">',
                  '<gml:coordinates>',coords(),'</gml:coordinates>',
                  '</gml:Box>',
                  '</ogc:BBOX>',
                  '<ogc:PropertyIsEqualTo>',
                  '<ogc:PropertyName>SiteFunction</ogc:PropertyName>',
                  '<ogc:Literal>Airport</ogc:Literal>',
                  '</ogc:PropertyIsEqualTo>',
                  '</ogc:And>',
                  '</ogc:Filter>')
    #Define (WFS) parameters object.
    wfsParams <- paste0("key=",Sys.getenv("OS_PROJECT_API_KEY"),
                        "&service=WFS&request=GetFeature&version=2.0.0&typeNames=Sites_FunctionalSite&outputFormat=GeoJSON&srsName=EPSG:27700",
                        "&filter=",xml)
    #Define query url
    url <-  paste0(wfsServiceUrl,"?",wfsParams)
    #Query api and convert geojson to spatial features 
    sf <- read_sf(GET(url),crs=27700,as_tibble=F)
    #Set CRS for standard web mapping (convert from national grid to degrees)
    sfw <- st_transform(sf,4326)
    #Finally update map
    leafletProxy("map") %>% clearShapes() %>% addPolygons(data=sfw)
    }
  })
}

#Launch the app
shinyApp(ui, server)
```
