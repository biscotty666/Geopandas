# Map Projections

- Depends on the __PROJ library__ via `pyproj`
- crs is stored in a `CRS` object from `pyproj`
- uses `.to_crs()` method
- must pat attention to degrees vs cartesion coordinates

- WGS84 / ESPG 4326: most common but distorts areas away from equator
- ETRS-LAEA / ESPG 3035: Lambert Azimuthal Equal Area, for country-level Europe data

__Reproject from WGS84 to Lambert Azimuthal Equal Area__

Recommended for Europe


```python
import geopandas as gpd 
fp = "data/ch6/EU_countries/eu_countries_2022.gpkg"
data = gpd.read_file(fp)

print(type(data.crs))

data.crs
```

    <class 'pyproj.crs.crs.CRS'>





    <Geographic 2D CRS: EPSG:4326>
    Name: WGS 84
    Axis Info [ellipsoidal]:
    - Lat[north]: Geodetic latitude (degree)
    - Lon[east]: Geodetic longitude (degree)
    Area of Use:
    - name: World.
    - bounds: (-180.0, -90.0, 180.0, 90.0)
    Datum: World Geodetic System 1984 ensemble
    - Ellipsoid: WGS 84
    - Prime Meridian: Greenwich




```python
data["geometry"].head()
```




    0    MULTIPOLYGON (((13.684 46.4375, 13.511 46.3484...
    1    MULTIPOLYGON (((6.3156 50.497, 6.405 50.3233, ...
    2    MULTIPOLYGON (((28.498 43.4341, 28.0602 43.316...
    3    MULTIPOLYGON (((16.9498 48.5358, 16.8511 48.43...
    4    MULTIPOLYGON (((32.9417 34.6418, 32.559 34.687...
    Name: geometry, dtype: geometry



Coordinates appear to be in decimal degrees


```python
data_wgs84 = data.copy()

data = data.to_crs(epsg=3035)
data["geometry"].head()
```




    0    MULTIPOLYGON (((4604288.477 2598607.47, 459144...
    1    MULTIPOLYGON (((4059689.242 3049361.18, 406508...
    2    MULTIPOLYGON (((5805367.757 2442801.252, 57739...
    3    MULTIPOLYGON (((4833567.363 2848881.974, 48272...
    4    MULTIPOLYGON (((6413299.362 1602181.345, 63782...
    Name: geometry, dtype: geometry




```python
data.crs
```




    <Projected CRS: EPSG:3035>
    Name: ETRS89-extended / LAEA Europe
    Axis Info [cartesian]:
    - Y[north]: Northing (metre)
    - X[east]: Easting (metre)
    Area of Use:
    - name: Europe - European Union (EU) countries and candidates. Europe - onshore and offshore: Albania; Andorra; Austria; Belgium; Bosnia and Herzegovina; Bulgaria; Croatia; Cyprus; Czechia; Denmark; Estonia; Faroe Islands; Finland; France; Germany; Gibraltar; Greece; Hungary; Iceland; Ireland; Italy; Kosovo; Latvia; Liechtenstein; Lithuania; Luxembourg; Malta; Monaco; Montenegro; Netherlands; North Macedonia; Norway including Svalbard and Jan Mayen; Poland; Portugal including Madeira and Azores; Romania; San Marino; Serbia; Slovakia; Slovenia; Spain including Canary Islands; Sweden; Switzerland; Türkiye (Turkey); United Kingdom (UK) including Channel Islands and Isle of Man; Vatican City State.
    - bounds: (-35.58, 24.6, 44.83, 84.73)
    Coordinate Operation:
    - name: Europe Equal Area 2001
    - method: Lambert Azimuthal Equal Area
    Datum: European Terrestrial Reference System 1989 ensemble
    - Ellipsoid: GRS 1980
    - Prime Meridian: Greenwich




```python
data.crs.to_epsg()
```




    3035




```python
%matplotlib inline

import matplotlib.pyplot as plt

fig, (ax1, ax2) = plt.subplots(nrows=1, ncols=2, figsize=(12, 12))

data_wgs84.plot(ax=ax1, facecolor="gray")
data.plot(ax=ax2, facecolor="blue", edgecolor="white", lw=0.5)

ax1.set_title("WGS84")
ax2.set_title("ETRS Lambert Azimuthal Equal Area projection")
ax1.set_aspect(aspect=1)
ax2.set_aspect(aspect=1)

# Remove empty white space around the plot
plt.tight_layout();
```


    
![png](MapProjections_files/MapProjections_9_0.png)
    



```python
# Ouput filepath
outfp = "data/Europe_borders_epsg3035.shp"

# Save to disk
data.to_file(outfp)
```

## Example of EPSG-3067 for Finland

Very far north, so a special projection is used


```python
finland_wgs84 = data_wgs84.loc[data_wgs84["NAME_ENGL"] == "Finland"].copy()
finland_etrs89 = finland_wgs84.to_crs(epsg=3067)

fig, (ax1, ax2) = plt.subplots(nrows=1, ncols=2, figsize=(5, 5))
finland_wgs84.plot(ax=ax1, facecolor="gray")
finland_etrs89.plot(ax=ax2, facecolor="blue", edgecolor="white", lw=0.5)

ax1.set_title("WGS84")
ax2.set_title("ETRS89 / TM35FIN")
ax1.set_aspect(aspect=1)
ax2.set_aspect(aspect=1)

plt.tight_layout();
```


    
![png](MapProjections_files/MapProjections_12_0.png)
    


# Parse, define and convert CRS information


```python
from pyproj import CRS

crs_object = CRS.from_epsg(3035)
```


```python
print(
    f"Name: {crs_object.name}\nCoord system: {crs_object.coordinate_system.name}\n\
Bounds: {crs_object.area_of_use.bounds}\nDatum: {crs_object.datum.name}\n"
)
```

    Name: ETRS89-extended / LAEA Europe
    Coord system: cartesian
    Bounds: (-35.58, 24.6, 44.83, 84.73)
    Datum: European Terrestrial Reference System 1989 ensemble
    


__export and import to wkt__


```python
crs_wkt = crs_object.to_wkt()
crs_wkt
```




    'PROJCRS["ETRS89-extended / LAEA Europe",BASEGEOGCRS["ETRS89",ENSEMBLE["European Terrestrial Reference System 1989 ensemble",MEMBER["European Terrestrial Reference Frame 1989"],MEMBER["European Terrestrial Reference Frame 1990"],MEMBER["European Terrestrial Reference Frame 1991"],MEMBER["European Terrestrial Reference Frame 1992"],MEMBER["European Terrestrial Reference Frame 1993"],MEMBER["European Terrestrial Reference Frame 1994"],MEMBER["European Terrestrial Reference Frame 1996"],MEMBER["European Terrestrial Reference Frame 1997"],MEMBER["European Terrestrial Reference Frame 2000"],MEMBER["European Terrestrial Reference Frame 2005"],MEMBER["European Terrestrial Reference Frame 2014"],MEMBER["European Terrestrial Reference Frame 2020"],ELLIPSOID["GRS 1980",6378137,298.257222101,LENGTHUNIT["metre",1]],ENSEMBLEACCURACY[0.1]],PRIMEM["Greenwich",0,ANGLEUNIT["degree",0.0174532925199433]],ID["EPSG",4258]],CONVERSION["Europe Equal Area 2001",METHOD["Lambert Azimuthal Equal Area",ID["EPSG",9820]],PARAMETER["Latitude of natural origin",52,ANGLEUNIT["degree",0.0174532925199433],ID["EPSG",8801]],PARAMETER["Longitude of natural origin",10,ANGLEUNIT["degree",0.0174532925199433],ID["EPSG",8802]],PARAMETER["False easting",4321000,LENGTHUNIT["metre",1],ID["EPSG",8806]],PARAMETER["False northing",3210000,LENGTHUNIT["metre",1],ID["EPSG",8807]]],CS[Cartesian,2],AXIS["northing (Y)",north,ORDER[1],LENGTHUNIT["metre",1]],AXIS["easting (X)",east,ORDER[2],LENGTHUNIT["metre",1]],USAGE[SCOPE["Statistical analysis."],AREA["Europe - European Union (EU) countries and candidates. Europe - onshore and offshore: Albania; Andorra; Austria; Belgium; Bosnia and Herzegovina; Bulgaria; Croatia; Cyprus; Czechia; Denmark; Estonia; Faroe Islands; Finland; France; Germany; Gibraltar; Greece; Hungary; Iceland; Ireland; Italy; Kosovo; Latvia; Liechtenstein; Lithuania; Luxembourg; Malta; Monaco; Montenegro; Netherlands; North Macedonia; Norway including Svalbard and Jan Mayen; Poland; Portugal including Madeira and Azores; Romania; San Marino; Serbia; Slovakia; Slovenia; Spain including Canary Islands; Sweden; Switzerland; Türkiye (Turkey); United Kingdom (UK) including Channel Islands and Isle of Man; Vatican City State."],BBOX[24.6,-35.58,84.73,44.83]],ID["EPSG",3035]]'



Retrieving epsg from WKT. If unable to determine automatically, can set a confidence level.


```python
epsg = CRS(crs_wkt).to_epsg()
epsg
```




    3035




```python
CRS(crs_wkt).to_epsg(min_confidence=25)
```




    3035



# CRS from scratch

- Adding a crs to a gdf
- Specifying crs on gdf creation


```python
from shapely.geometry import Point 

gdf = gpd.GeoDataFrame({"geometry": Point(35.106766, -106.629181)}, 
                      index=[0])
print(gdf.crs)
```

    None



```python
from pyproj import CRS

gdf = gdf.set_crs(CRS.from_epsg(4326))
gdf.crs
```




    <Geographic 2D CRS: EPSG:4326>
    Name: WGS 84
    Axis Info [ellipsoidal]:
    - Lat[north]: Geodetic latitude (degree)
    - Lon[east]: Geodetic longitude (degree)
    Area of Use:
    - name: World.
    - bounds: (-180.0, -90.0, 180.0, 90.0)
    Datum: World Geodetic System 1984 ensemble
    - Ellipsoid: WGS 84
    - Prime Meridian: Greenwich



Shorthand:


```python
gdf = gdf.set_crs(epsg=4326)
print(gdf.crs)
```

    EPSG:4326


or


```python
gdf = gpd.GeoDataFrame(geometry=[Point(24.950899, 60.169158)], crs="EPSG:4326")
gdf.crs
```




    <Geographic 2D CRS: EPSG:4326>
    Name: WGS 84
    Axis Info [ellipsoidal]:
    - Lat[north]: Geodetic latitude (degree)
    - Lon[east]: Geodetic longitude (degree)
    Area of Use:
    - name: World.
    - bounds: (-180.0, -90.0, 180.0, 90.0)
    Datum: World Geodetic System 1984 ensemble
    - Ellipsoid: WGS 84
    - Prime Meridian: Greenwich



# Projection examples


```python
fp = "data/ch6/Natural_Earth/ne_110m_admin_0_countries.zip"

admin = gpd.read_file(fp)
admin.crs
```




    <Geographic 2D CRS: EPSG:4326>
    Name: WGS 84
    Axis Info [ellipsoidal]:
    - Lat[north]: Geodetic latitude (degree)
    - Lon[east]: Geodetic longitude (degree)
    Area of Use:
    - name: World.
    - bounds: (-180.0, -90.0, 180.0, 90.0)
    Datum: World Geodetic System 1984 ensemble
    - Ellipsoid: WGS 84
    - Prime Meridian: Greenwich



__original crs__


```python
admin.plot(figsize=(12,6))
plt.title("WGS84");
```


    
![png](MapProjections_files/MapProjections_32_0.png)
    


__Mercator projection__


```python
admin.to_crs(epsg=3857).explore()
```




<div style="width:100%;"><div style="position:relative;width:100%;height:0;padding-bottom:60%;"><span style="color:#565656">Make this Notebook Trusted to load map: File -> Trust Notebook</span><iframe srcdoc="&lt;!DOCTYPE html&gt;
&lt;html&gt;
&lt;head&gt;

    &lt;meta http-equiv=&quot;content-type&quot; content=&quot;text/html; charset=UTF-8&quot; /&gt;

        &lt;script&gt;
            L_NO_TOUCH = false;
            L_DISABLE_3D = false;
        &lt;/script&gt;

    &lt;style&gt;html, body {width: 100%;height: 100%;margin: 0;padding: 0;}&lt;/style&gt;
    &lt;style&gt;#map {position:absolute;top:0;bottom:0;right:0;left:0;}&lt;/style&gt;
    &lt;script src=&quot;https://cdn.jsdelivr.net/npm/leaflet@1.9.3/dist/leaflet.js&quot;&gt;&lt;/script&gt;
    &lt;script src=&quot;https://code.jquery.com/jquery-3.7.1.min.js&quot;&gt;&lt;/script&gt;
    &lt;script src=&quot;https://cdn.jsdelivr.net/npm/bootstrap@5.2.2/dist/js/bootstrap.bundle.min.js&quot;&gt;&lt;/script&gt;
    &lt;script src=&quot;https://cdnjs.cloudflare.com/ajax/libs/Leaflet.awesome-markers/2.0.2/leaflet.awesome-markers.js&quot;&gt;&lt;/script&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://cdn.jsdelivr.net/npm/leaflet@1.9.3/dist/leaflet.css&quot;/&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://cdn.jsdelivr.net/npm/bootstrap@5.2.2/dist/css/bootstrap.min.css&quot;/&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://netdna.bootstrapcdn.com/bootstrap/3.0.0/css/bootstrap-glyphicons.css&quot;/&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://cdn.jsdelivr.net/npm/@fortawesome/fontawesome-free@6.2.0/css/all.min.css&quot;/&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://cdnjs.cloudflare.com/ajax/libs/Leaflet.awesome-markers/2.0.2/leaflet.awesome-markers.css&quot;/&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://cdn.jsdelivr.net/gh/python-visualization/folium/folium/templates/leaflet.awesome.rotate.min.css&quot;/&gt;

            &lt;meta name=&quot;viewport&quot; content=&quot;width=device-width,
                initial-scale=1.0, maximum-scale=1.0, user-scalable=no&quot; /&gt;
            &lt;style&gt;
                #map_98db32a608a1771dcf927666cda8eaf0 {
                    position: relative;
                    width: 100.0%;
                    height: 100.0%;
                    left: 0.0%;
                    top: 0.0%;
                }
                .leaflet-container { font-size: 1rem; }
            &lt;/style&gt;


                    &lt;style&gt;
                        .foliumtooltip {

                        }
                       .foliumtooltip table{
                            margin: auto;
                        }
                        .foliumtooltip tr{
                            text-align: left;
                        }
                        .foliumtooltip th{
                            padding: 2px; padding-right: 8px;
                        }
                    &lt;/style&gt;

&lt;/head&gt;
&lt;body&gt;


            &lt;div class=&quot;folium-map&quot; id=&quot;map_98db32a608a1771dcf927666cda8eaf0&quot; &gt;&lt;/div&gt;

&lt;/body&gt;
&lt;script&gt;


            var map_98db32a608a1771dcf927666cda8eaf0 = L.map(
                &quot;map_98db32a608a1771dcf927666cda8eaf0&quot;,
                {
                    center: [-3.1774349999999956, 1.4210854715202004e-14],
                    crs: L.CRS.EPSG3857,
                    zoom: 10,
                    zoomControl: true,
                    preferCanvas: false,
                }
            );
            L.control.scale().addTo(map_98db32a608a1771dcf927666cda8eaf0);





            var tile_layer_75fb65de18066ed1a31809fa50b54f97 = L.tileLayer(
                &quot;https://tile.openstreetmap.org/{z}/{x}/{y}.png&quot;,
                {&quot;attribution&quot;: &quot;\u0026copy; \u003ca href=\&quot;https://www.openstreetmap.org/copyright\&quot;\u003eOpenStreetMap\u003c/a\u003e contributors&quot;, &quot;detectRetina&quot;: false, &quot;maxNativeZoom&quot;: 19, &quot;maxZoom&quot;: 19, &quot;minZoom&quot;: 0, &quot;noWrap&quot;: false, &quot;opacity&quot;: 1, &quot;subdomains&quot;: &quot;abc&quot;, &quot;tms&quot;: false}
            );


            tile_layer_75fb65de18066ed1a31809fa50b54f97.addTo(map_98db32a608a1771dcf927666cda8eaf0);


            map_98db32a608a1771dcf927666cda8eaf0.fitBounds(
                [[-90.0, -180.0], [83.64513000000001, 180.00000000000003]],
                {}
            );


        function geo_json_95aaa13bc60ec901f78b88f95a399246_styler(feature) {
            switch(feature.id) {
                default:
                    return {&quot;fillOpacity&quot;: 0.5, &quot;weight&quot;: 2};
            }
        }
        function geo_json_95aaa13bc60ec901f78b88f95a399246_highlighter(feature) {
            switch(feature.id) {
                default:
                    return {&quot;fillOpacity&quot;: 0.75};
            }
        }
        function geo_json_95aaa13bc60ec901f78b88f95a399246_pointToLayer(feature, latlng) {
            var opts = {&quot;bubblingMouseEvents&quot;: true, &quot;color&quot;: &quot;#3388ff&quot;, &quot;dashArray&quot;: null, &quot;dashOffset&quot;: null, &quot;fill&quot;: true, &quot;fillColor&quot;: &quot;#3388ff&quot;, &quot;fillOpacity&quot;: 0.2, &quot;fillRule&quot;: &quot;evenodd&quot;, &quot;lineCap&quot;: &quot;round&quot;, &quot;lineJoin&quot;: &quot;round&quot;, &quot;opacity&quot;: 1.0, &quot;radius&quot;: 2, &quot;stroke&quot;: true, &quot;weight&quot;: 3};

            let style = geo_json_95aaa13bc60ec901f78b88f95a399246_styler(feature)
            Object.assign(opts, style)

            return new L.CircleMarker(latlng, opts)
        }

        function geo_json_95aaa13bc60ec901f78b88f95a399246_onEachFeature(feature, layer) {
            layer.on({
                mouseout: function(e) {
                    if(typeof e.target.setStyle === &quot;function&quot;){
                            geo_json_95aaa13bc60ec901f78b88f95a399246.resetStyle(e.target);
                    }
                },
                mouseover: function(e) {
                    if(typeof e.target.setStyle === &quot;function&quot;){
                        const highlightStyle = geo_json_95aaa13bc60ec901f78b88f95a399246_highlighter(e.target.feature)
                        e.target.setStyle(highlightStyle);
                    }
                },
            });
        };
        var geo_json_95aaa13bc60ec901f78b88f95a399246 = L.geoJson(null, {
                onEachFeature: geo_json_95aaa13bc60ec901f78b88f95a399246_onEachFeature,

                style: geo_json_95aaa13bc60ec901f78b88f95a399246_styler,
                pointToLayer: geo_json_95aaa13bc60ec901f78b88f95a399246_pointToLayer,
        });

        function geo_json_95aaa13bc60ec901f78b88f95a399246_add (data) {
            geo_json_95aaa13bc60ec901f78b88f95a399246
                .addData(data);
        }



    geo_json_95aaa13bc60ec901f78b88f95a399246.bindTooltip(
    function(layer){
    let div = L.DomUtil.create(&#x27;div&#x27;);

    let handleObject = feature=&gt;typeof(feature)==&#x27;object&#x27; ? JSON.stringify(feature) : feature;
    let fields = [&quot;featurecla&quot;, &quot;scalerank&quot;, &quot;LABELRANK&quot;, &quot;SOVEREIGNT&quot;, &quot;SOV_A3&quot;, &quot;ADM0_DIF&quot;, &quot;LEVEL&quot;, &quot;TYPE&quot;, &quot;TLC&quot;, &quot;ADMIN&quot;, &quot;ADM0_A3&quot;, &quot;GEOU_DIF&quot;, &quot;GEOUNIT&quot;, &quot;GU_A3&quot;, &quot;SU_DIF&quot;, &quot;SUBUNIT&quot;, &quot;SU_A3&quot;, &quot;BRK_DIFF&quot;, &quot;NAME&quot;, &quot;NAME_LONG&quot;, &quot;BRK_A3&quot;, &quot;BRK_NAME&quot;, &quot;BRK_GROUP&quot;, &quot;ABBREV&quot;, &quot;POSTAL&quot;, &quot;FORMAL_EN&quot;, &quot;FORMAL_FR&quot;, &quot;NAME_CIAWF&quot;, &quot;NOTE_ADM0&quot;, &quot;NOTE_BRK&quot;, &quot;NAME_SORT&quot;, &quot;NAME_ALT&quot;, &quot;MAPCOLOR7&quot;, &quot;MAPCOLOR8&quot;, &quot;MAPCOLOR9&quot;, &quot;MAPCOLOR13&quot;, &quot;POP_EST&quot;, &quot;POP_RANK&quot;, &quot;POP_YEAR&quot;, &quot;GDP_MD&quot;, &quot;GDP_YEAR&quot;, &quot;ECONOMY&quot;, &quot;INCOME_GRP&quot;, &quot;FIPS_10&quot;, &quot;ISO_A2&quot;, &quot;ISO_A2_EH&quot;, &quot;ISO_A3&quot;, &quot;ISO_A3_EH&quot;, &quot;ISO_N3&quot;, &quot;ISO_N3_EH&quot;, &quot;UN_A3&quot;, &quot;WB_A2&quot;, &quot;WB_A3&quot;, &quot;WOE_ID&quot;, &quot;WOE_ID_EH&quot;, &quot;WOE_NOTE&quot;, &quot;ADM0_ISO&quot;, &quot;ADM0_DIFF&quot;, &quot;ADM0_TLC&quot;, &quot;ADM0_A3_US&quot;, &quot;ADM0_A3_FR&quot;, &quot;ADM0_A3_RU&quot;, &quot;ADM0_A3_ES&quot;, &quot;ADM0_A3_CN&quot;, &quot;ADM0_A3_TW&quot;, &quot;ADM0_A3_IN&quot;, &quot;ADM0_A3_NP&quot;, &quot;ADM0_A3_PK&quot;, &quot;ADM0_A3_DE&quot;, &quot;ADM0_A3_GB&quot;, &quot;ADM0_A3_BR&quot;, &quot;ADM0_A3_IL&quot;, &quot;ADM0_A3_PS&quot;, &quot;ADM0_A3_SA&quot;, &quot;ADM0_A3_EG&quot;, &quot;ADM0_A3_MA&quot;, &quot;ADM0_A3_PT&quot;, &quot;ADM0_A3_AR&quot;, &quot;ADM0_A3_JP&quot;, &quot;ADM0_A3_KO&quot;, &quot;ADM0_A3_VN&quot;, &quot;ADM0_A3_TR&quot;, &quot;ADM0_A3_ID&quot;, &quot;ADM0_A3_PL&quot;, &quot;ADM0_A3_GR&quot;, &quot;ADM0_A3_IT&quot;, &quot;ADM0_A3_NL&quot;, &quot;ADM0_A3_SE&quot;, &quot;ADM0_A3_BD&quot;, &quot;ADM0_A3_UA&quot;, &quot;ADM0_A3_UN&quot;, &quot;ADM0_A3_WB&quot;, &quot;CONTINENT&quot;, &quot;REGION_UN&quot;, &quot;SUBREGION&quot;, &quot;REGION_WB&quot;, &quot;NAME_LEN&quot;, &quot;LONG_LEN&quot;, &quot;ABBREV_LEN&quot;, &quot;TINY&quot;, &quot;HOMEPART&quot;, &quot;MIN_ZOOM&quot;, &quot;MIN_LABEL&quot;, &quot;MAX_LABEL&quot;, &quot;LABEL_X&quot;, &quot;LABEL_Y&quot;, &quot;NE_ID&quot;, &quot;WIKIDATAID&quot;, &quot;NAME_AR&quot;, &quot;NAME_BN&quot;, &quot;NAME_DE&quot;, &quot;NAME_EN&quot;, &quot;NAME_ES&quot;, &quot;NAME_FA&quot;, &quot;NAME_FR&quot;, &quot;NAME_EL&quot;, &quot;NAME_HE&quot;, &quot;NAME_HI&quot;, &quot;NAME_HU&quot;, &quot;NAME_ID&quot;, &quot;NAME_IT&quot;, &quot;NAME_JA&quot;, &quot;NAME_KO&quot;, &quot;NAME_NL&quot;, &quot;NAME_PL&quot;, &quot;NAME_PT&quot;, &quot;NAME_RU&quot;, &quot;NAME_SV&quot;, &quot;NAME_TR&quot;, &quot;NAME_UK&quot;, &quot;NAME_UR&quot;, &quot;NAME_VI&quot;, &quot;NAME_ZH&quot;, &quot;NAME_ZHT&quot;, &quot;FCLASS_ISO&quot;, &quot;TLC_DIFF&quot;, &quot;FCLASS_TLC&quot;, &quot;FCLASS_US&quot;, &quot;FCLASS_FR&quot;, &quot;FCLASS_RU&quot;, &quot;FCLASS_ES&quot;, &quot;FCLASS_CN&quot;, &quot;FCLASS_TW&quot;, &quot;FCLASS_IN&quot;, &quot;FCLASS_NP&quot;, &quot;FCLASS_PK&quot;, &quot;FCLASS_DE&quot;, &quot;FCLASS_GB&quot;, &quot;FCLASS_BR&quot;, &quot;FCLASS_IL&quot;, &quot;FCLASS_PS&quot;, &quot;FCLASS_SA&quot;, &quot;FCLASS_EG&quot;, &quot;FCLASS_MA&quot;, &quot;FCLASS_PT&quot;, &quot;FCLASS_AR&quot;, &quot;FCLASS_JP&quot;, &quot;FCLASS_KO&quot;, &quot;FCLASS_VN&quot;, &quot;FCLASS_TR&quot;, &quot;FCLASS_ID&quot;, &quot;FCLASS_PL&quot;, &quot;FCLASS_GR&quot;, &quot;FCLASS_IT&quot;, &quot;FCLASS_NL&quot;, &quot;FCLASS_SE&quot;, &quot;FCLASS_BD&quot;, &quot;FCLASS_UA&quot;];
    let aliases = [&quot;featurecla&quot;, &quot;scalerank&quot;, &quot;LABELRANK&quot;, &quot;SOVEREIGNT&quot;, &quot;SOV_A3&quot;, &quot;ADM0_DIF&quot;, &quot;LEVEL&quot;, &quot;TYPE&quot;, &quot;TLC&quot;, &quot;ADMIN&quot;, &quot;ADM0_A3&quot;, &quot;GEOU_DIF&quot;, &quot;GEOUNIT&quot;, &quot;GU_A3&quot;, &quot;SU_DIF&quot;, &quot;SUBUNIT&quot;, &quot;SU_A3&quot;, &quot;BRK_DIFF&quot;, &quot;NAME&quot;, &quot;NAME_LONG&quot;, &quot;BRK_A3&quot;, &quot;BRK_NAME&quot;, &quot;BRK_GROUP&quot;, &quot;ABBREV&quot;, &quot;POSTAL&quot;, &quot;FORMAL_EN&quot;, &quot;FORMAL_FR&quot;, &quot;NAME_CIAWF&quot;, &quot;NOTE_ADM0&quot;, &quot;NOTE_BRK&quot;, &quot;NAME_SORT&quot;, &quot;NAME_ALT&quot;, &quot;MAPCOLOR7&quot;, &quot;MAPCOLOR8&quot;, &quot;MAPCOLOR9&quot;, &quot;MAPCOLOR13&quot;, &quot;POP_EST&quot;, &quot;POP_RANK&quot;, &quot;POP_YEAR&quot;, &quot;GDP_MD&quot;, &quot;GDP_YEAR&quot;, &quot;ECONOMY&quot;, &quot;INCOME_GRP&quot;, &quot;FIPS_10&quot;, &quot;ISO_A2&quot;, &quot;ISO_A2_EH&quot;, &quot;ISO_A3&quot;, &quot;ISO_A3_EH&quot;, &quot;ISO_N3&quot;, &quot;ISO_N3_EH&quot;, &quot;UN_A3&quot;, &quot;WB_A2&quot;, &quot;WB_A3&quot;, &quot;WOE_ID&quot;, &quot;WOE_ID_EH&quot;, &quot;WOE_NOTE&quot;, &quot;ADM0_ISO&quot;, &quot;ADM0_DIFF&quot;, &quot;ADM0_TLC&quot;, &quot;ADM0_A3_US&quot;, &quot;ADM0_A3_FR&quot;, &quot;ADM0_A3_RU&quot;, &quot;ADM0_A3_ES&quot;, &quot;ADM0_A3_CN&quot;, &quot;ADM0_A3_TW&quot;, &quot;ADM0_A3_IN&quot;, &quot;ADM0_A3_NP&quot;, &quot;ADM0_A3_PK&quot;, &quot;ADM0_A3_DE&quot;, &quot;ADM0_A3_GB&quot;, &quot;ADM0_A3_BR&quot;, &quot;ADM0_A3_IL&quot;, &quot;ADM0_A3_PS&quot;, &quot;ADM0_A3_SA&quot;, &quot;ADM0_A3_EG&quot;, &quot;ADM0_A3_MA&quot;, &quot;ADM0_A3_PT&quot;, &quot;ADM0_A3_AR&quot;, &quot;ADM0_A3_JP&quot;, &quot;ADM0_A3_KO&quot;, &quot;ADM0_A3_VN&quot;, &quot;ADM0_A3_TR&quot;, &quot;ADM0_A3_ID&quot;, &quot;ADM0_A3_PL&quot;, &quot;ADM0_A3_GR&quot;, &quot;ADM0_A3_IT&quot;, &quot;ADM0_A3_NL&quot;, &quot;ADM0_A3_SE&quot;, &quot;ADM0_A3_BD&quot;, &quot;ADM0_A3_UA&quot;, &quot;ADM0_A3_UN&quot;, &quot;ADM0_A3_WB&quot;, &quot;CONTINENT&quot;, &quot;REGION_UN&quot;, &quot;SUBREGION&quot;, &quot;REGION_WB&quot;, &quot;NAME_LEN&quot;, &quot;LONG_LEN&quot;, &quot;ABBREV_LEN&quot;, &quot;TINY&quot;, &quot;HOMEPART&quot;, &quot;MIN_ZOOM&quot;, &quot;MIN_LABEL&quot;, &quot;MAX_LABEL&quot;, &quot;LABEL_X&quot;, &quot;LABEL_Y&quot;, &quot;NE_ID&quot;, &quot;WIKIDATAID&quot;, &quot;NAME_AR&quot;, &quot;NAME_BN&quot;, &quot;NAME_DE&quot;, &quot;NAME_EN&quot;, &quot;NAME_ES&quot;, &quot;NAME_FA&quot;, &quot;NAME_FR&quot;, &quot;NAME_EL&quot;, &quot;NAME_HE&quot;, &quot;NAME_HI&quot;, &quot;NAME_HU&quot;, &quot;NAME_ID&quot;, &quot;NAME_IT&quot;, &quot;NAME_JA&quot;, &quot;NAME_KO&quot;, &quot;NAME_NL&quot;, &quot;NAME_PL&quot;, &quot;NAME_PT&quot;, &quot;NAME_RU&quot;, &quot;NAME_SV&quot;, &quot;NAME_TR&quot;, &quot;NAME_UK&quot;, &quot;NAME_UR&quot;, &quot;NAME_VI&quot;, &quot;NAME_ZH&quot;, &quot;NAME_ZHT&quot;, &quot;FCLASS_ISO&quot;, &quot;TLC_DIFF&quot;, &quot;FCLASS_TLC&quot;, &quot;FCLASS_US&quot;, &quot;FCLASS_FR&quot;, &quot;FCLASS_RU&quot;, &quot;FCLASS_ES&quot;, &quot;FCLASS_CN&quot;, &quot;FCLASS_TW&quot;, &quot;FCLASS_IN&quot;, &quot;FCLASS_NP&quot;, &quot;FCLASS_PK&quot;, &quot;FCLASS_DE&quot;, &quot;FCLASS_GB&quot;, &quot;FCLASS_BR&quot;, &quot;FCLASS_IL&quot;, &quot;FCLASS_PS&quot;, &quot;FCLASS_SA&quot;, &quot;FCLASS_EG&quot;, &quot;FCLASS_MA&quot;, &quot;FCLASS_PT&quot;, &quot;FCLASS_AR&quot;, &quot;FCLASS_JP&quot;, &quot;FCLASS_KO&quot;, &quot;FCLASS_VN&quot;, &quot;FCLASS_TR&quot;, &quot;FCLASS_ID&quot;, &quot;FCLASS_PL&quot;, &quot;FCLASS_GR&quot;, &quot;FCLASS_IT&quot;, &quot;FCLASS_NL&quot;, &quot;FCLASS_SE&quot;, &quot;FCLASS_BD&quot;, &quot;FCLASS_UA&quot;];
    let table = &#x27;&lt;table&gt;&#x27; +
        String(
        fields.map(
        (v,i)=&gt;
        `&lt;tr&gt;
            &lt;th&gt;${aliases[i]}&lt;/th&gt;

            &lt;td&gt;${handleObject(layer.feature.properties[v])}&lt;/td&gt;
        &lt;/tr&gt;`).join(&#x27;&#x27;))
    +&#x27;&lt;/table&gt;&#x27;;
    div.innerHTML=table;

    return div
    }
    ,{&quot;className&quot;: &quot;foliumtooltip&quot;, &quot;sticky&quot;: true});


            geo_json_95aaa13bc60ec901f78b88f95a399246.addTo(map_98db32a608a1771dcf927666cda8eaf0);

&lt;/script&gt;
&lt;/html&gt;" style="position:absolute;width:100%;height:100%;left:0;top:0;border:none !important;" allowfullscreen webkitallowfullscreen mozallowfullscreen></iframe></div></div>



__Eckert IV__

For less distortion near the poles.


```python
admin.to_crs(crs="ESRI:54012").plot(figsize=(12,12))
plt.title("Eckert IV")
plt.axis("off");
```


    
![png](MapProjections_files/MapProjections_36_0.png)
    


__Orthographic projection__

Specify with proj-string using `+lat, +lon`


```python
proj_string = "+proj=ortho +lat=60.00 +lon=24.0000"
admin.to_crs(crs=proj_string).plot() 
plt.axis("off")
plt.title("Orthographic");
```


    
![png](MapProjections_files/MapProjections_38_0.png)
    



```python

```