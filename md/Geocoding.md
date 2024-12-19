# Geocoding


```python
import pandas as pd 
import geopandas as gpd 
from shapely.geometry import Point

fp = "data/ch6/Helsinki/addresses.txt"
data = pd.read_csv(fp, sep=";")
data.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>id</th>
      <th>addr</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1000</td>
      <td>Itämerenkatu 14, 00101 Helsinki, Finland</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1001</td>
      <td>Kampinkuja 1, 00100 Helsinki, Finland</td>
    </tr>
    <tr>
      <th>2</th>
      <td>1002</td>
      <td>Kaivokatu 8, 00101 Helsinki, Finland</td>
    </tr>
    <tr>
      <th>3</th>
      <td>1003</td>
      <td>Hermannin rantatie 1, 00580 Helsinki, Finland</td>
    </tr>
    <tr>
      <th>4</th>
      <td>1005</td>
      <td>Tyynenmerenkatu 9, 00220 Helsinki, Finland</td>
    </tr>
  </tbody>
</table>
</div>




```python
from geopandas.tools import geocode

geo = geocode(
    data["addr"], provider="nominatim", 
    user_agent="biscotty", timeout=10
)
geo.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>geometry</th>
      <th>address</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>POINT (24.91556 60.1632)</td>
      <td>Ruoholahti, 14, Itämerenkatu, Ruoholahti, Läns...</td>
    </tr>
    <tr>
      <th>1</th>
      <td>POINT (24.93009 60.16846)</td>
      <td>1, Kampinkuja, Kamppi, Eteläinen suurpiiri, He...</td>
    </tr>
    <tr>
      <th>2</th>
      <td>POINT (24.94153 60.17016)</td>
      <td>Espresso House, 8, Kaivokatu, Keskusta, Kluuvi...</td>
    </tr>
    <tr>
      <th>3</th>
      <td>POINT (24.97675 60.19438)</td>
      <td>Hermannin rantatie, Hermanninranta, Hermanni, ...</td>
    </tr>
    <tr>
      <th>4</th>
      <td>POINT (24.92151 60.15662)</td>
      <td>9, Tyynenmerenkatu, Jätkäsaari, Länsisatama, E...</td>
    </tr>
  </tbody>
</table>
</div>




```python
geo.shape
```




    (34, 2)




```python
join = geo.join(data)
join.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>geometry</th>
      <th>address</th>
      <th>id</th>
      <th>addr</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>POINT (24.91556 60.1632)</td>
      <td>Ruoholahti, 14, Itämerenkatu, Ruoholahti, Läns...</td>
      <td>1000</td>
      <td>Itämerenkatu 14, 00101 Helsinki, Finland</td>
    </tr>
    <tr>
      <th>1</th>
      <td>POINT (24.93009 60.16846)</td>
      <td>1, Kampinkuja, Kamppi, Eteläinen suurpiiri, He...</td>
      <td>1001</td>
      <td>Kampinkuja 1, 00100 Helsinki, Finland</td>
    </tr>
    <tr>
      <th>2</th>
      <td>POINT (24.94153 60.17016)</td>
      <td>Espresso House, 8, Kaivokatu, Keskusta, Kluuvi...</td>
      <td>1002</td>
      <td>Kaivokatu 8, 00101 Helsinki, Finland</td>
    </tr>
    <tr>
      <th>3</th>
      <td>POINT (24.97675 60.19438)</td>
      <td>Hermannin rantatie, Hermanninranta, Hermanni, ...</td>
      <td>1003</td>
      <td>Hermannin rantatie 1, 00580 Helsinki, Finland</td>
    </tr>
    <tr>
      <th>4</th>
      <td>POINT (24.92151 60.15662)</td>
      <td>9, Tyynenmerenkatu, Jätkäsaari, Länsisatama, E...</td>
      <td>1005</td>
      <td>Tyynenmerenkatu 9, 00220 Helsinki, Finland</td>
    </tr>
  </tbody>
</table>
</div>




```python
outfp = "data/addresses.gpkg"
join.to_file(outfp)
```


```python
address_list = [
    "6119 Mustang Ln NW, Albuquerque, NM 87120",
    "105 S State Street, Ann Arbor, MI 48109",
    "5 Leona Terrace, Mahwah, NJ 07430"
]

geo2 = geocode(
    address_list, provider="nominatim", user_agent="biscotty", timeout=10
)

geo2.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>geometry</th>
      <th>address</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>POINT (-106.70813 35.14749)</td>
      <td>6119, Mustang Lane Northwest, Albuquerque, Ber...</td>
    </tr>
    <tr>
      <th>1</th>
      <td>POINT (-83.7399 42.28055)</td>
      <td>North Quad Dining Hall, 105, South State Stree...</td>
    </tr>
    <tr>
      <th>2</th>
      <td>POINT (-74.15647 41.03768)</td>
      <td>5, Leona Terrace, Mahwah, Bergen County, New J...</td>
    </tr>
  </tbody>
</table>
</div>




```python
geo2.explore(
    color="red", max_zoom=12, marker_kwds=dict(radius=8), 
    tiles="CartoDB Positron"
)
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
                #map_82d83b7b94c97141fc8cc282179155ea {
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


            &lt;div class=&quot;folium-map&quot; id=&quot;map_82d83b7b94c97141fc8cc282179155ea&quot; &gt;&lt;/div&gt;

&lt;/body&gt;
&lt;script&gt;


            var map_82d83b7b94c97141fc8cc282179155ea = L.map(
                &quot;map_82d83b7b94c97141fc8cc282179155ea&quot;,
                {
                    center: [38.714023998979584, -90.43230165108558],
                    crs: L.CRS.EPSG3857,
                    zoom: 10,
                    zoomControl: true,
                    preferCanvas: false,
                }
            );
            L.control.scale().addTo(map_82d83b7b94c97141fc8cc282179155ea);





            var tile_layer_7092da23acb04c9ea7ecf2284a7bc501 = L.tileLayer(
                &quot;https://a.basemaps.cartocdn.com/light_all/{z}/{x}/{y}{r}.png&quot;,
                {&quot;attribution&quot;: &quot;\u0026copy; \u003ca href=\&quot;https://www.openstreetmap.org/copyright\&quot;\u003eOpenStreetMap\u003c/a\u003e contributors \u0026copy; \u003ca href=\&quot;https://carto.com/attributions\&quot;\u003eCARTO\u003c/a\u003e&quot;, &quot;detectRetina&quot;: false, &quot;maxZoom&quot;: 12, &quot;minZoom&quot;: 0, &quot;noWrap&quot;: false, &quot;opacity&quot;: 1, &quot;subdomains&quot;: &quot;abc&quot;, &quot;tms&quot;: false}
            );


            tile_layer_7092da23acb04c9ea7ecf2284a7bc501.addTo(map_82d83b7b94c97141fc8cc282179155ea);


            map_82d83b7b94c97141fc8cc282179155ea.fitBounds(
                [[35.14749489795918, -106.70813463265306], [42.2805531, -74.1564686695181]],
                {}
            );


        function geo_json_b2ca56ab53121084c630d9e84f3024b7_styler(feature) {
            switch(feature.id) {
                default:
                    return {&quot;color&quot;: &quot;red&quot;, &quot;fillColor&quot;: &quot;red&quot;, &quot;fillOpacity&quot;: 0.5, &quot;weight&quot;: 2};
            }
        }
        function geo_json_b2ca56ab53121084c630d9e84f3024b7_highlighter(feature) {
            switch(feature.id) {
                default:
                    return {&quot;fillOpacity&quot;: 0.75};
            }
        }
        function geo_json_b2ca56ab53121084c630d9e84f3024b7_pointToLayer(feature, latlng) {
            var opts = {&quot;bubblingMouseEvents&quot;: true, &quot;color&quot;: &quot;#3388ff&quot;, &quot;dashArray&quot;: null, &quot;dashOffset&quot;: null, &quot;fill&quot;: true, &quot;fillColor&quot;: &quot;#3388ff&quot;, &quot;fillOpacity&quot;: 0.2, &quot;fillRule&quot;: &quot;evenodd&quot;, &quot;lineCap&quot;: &quot;round&quot;, &quot;lineJoin&quot;: &quot;round&quot;, &quot;opacity&quot;: 1.0, &quot;radius&quot;: 8, &quot;stroke&quot;: true, &quot;weight&quot;: 3};

            let style = geo_json_b2ca56ab53121084c630d9e84f3024b7_styler(feature)
            Object.assign(opts, style)

            return new L.CircleMarker(latlng, opts)
        }

        function geo_json_b2ca56ab53121084c630d9e84f3024b7_onEachFeature(feature, layer) {
            layer.on({
                mouseout: function(e) {
                    if(typeof e.target.setStyle === &quot;function&quot;){
                            geo_json_b2ca56ab53121084c630d9e84f3024b7.resetStyle(e.target);
                    }
                },
                mouseover: function(e) {
                    if(typeof e.target.setStyle === &quot;function&quot;){
                        const highlightStyle = geo_json_b2ca56ab53121084c630d9e84f3024b7_highlighter(e.target.feature)
                        e.target.setStyle(highlightStyle);
                    }
                },
            });
        };
        var geo_json_b2ca56ab53121084c630d9e84f3024b7 = L.geoJson(null, {
                onEachFeature: geo_json_b2ca56ab53121084c630d9e84f3024b7_onEachFeature,

                style: geo_json_b2ca56ab53121084c630d9e84f3024b7_styler,
                pointToLayer: geo_json_b2ca56ab53121084c630d9e84f3024b7_pointToLayer,
        });

        function geo_json_b2ca56ab53121084c630d9e84f3024b7_add (data) {
            geo_json_b2ca56ab53121084c630d9e84f3024b7
                .addData(data);
        }
            geo_json_b2ca56ab53121084c630d9e84f3024b7_add({&quot;bbox&quot;: [-106.70813463265306, 35.14749489795918, -74.1564686695181, 42.2805531], &quot;features&quot;: [{&quot;bbox&quot;: [-106.70813463265306, 35.14749489795918, -106.70813463265306, 35.14749489795918], &quot;geometry&quot;: {&quot;coordinates&quot;: [-106.70813463265306, 35.14749489795918], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;0&quot;, &quot;properties&quot;: {&quot;__folium_color&quot;: &quot;red&quot;, &quot;address&quot;: &quot;6119, Mustang Lane Northwest, Albuquerque, Bernalillo County, New Mexico, 87120, United States&quot;}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [-83.7399009, 42.2805531, -83.7399009, 42.2805531], &quot;geometry&quot;: {&quot;coordinates&quot;: [-83.7399009, 42.2805531], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;1&quot;, &quot;properties&quot;: {&quot;__folium_color&quot;: &quot;red&quot;, &quot;address&quot;: &quot;North Quad Dining Hall, 105, South State Street, Old Fourth Ward, Ann Arbor, Washtenaw County, Michigan, 48109, United States&quot;}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [-74.1564686695181, 41.03767750396952, -74.1564686695181, 41.03767750396952], &quot;geometry&quot;: {&quot;coordinates&quot;: [-74.1564686695181, 41.03767750396952], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;2&quot;, &quot;properties&quot;: {&quot;__folium_color&quot;: &quot;red&quot;, &quot;address&quot;: &quot;5, Leona Terrace, Mahwah, Bergen County, New Jersey, 07430, United States&quot;}, &quot;type&quot;: &quot;Feature&quot;}], &quot;type&quot;: &quot;FeatureCollection&quot;});



    geo_json_b2ca56ab53121084c630d9e84f3024b7.bindTooltip(
    function(layer){
    let div = L.DomUtil.create(&#x27;div&#x27;);

    let handleObject = feature=&gt;typeof(feature)==&#x27;object&#x27; ? JSON.stringify(feature) : feature;
    let fields = [&quot;address&quot;];
    let aliases = [&quot;address&quot;];
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


            geo_json_b2ca56ab53121084c630d9e84f3024b7.addTo(map_82d83b7b94c97141fc8cc282179155ea);

&lt;/script&gt;
&lt;/html&gt;" style="position:absolute;width:100%;height:100%;left:0;top:0;border:none !important;" allowfullscreen webkitallowfullscreen mozallowfullscreen></iframe></div></div>



# Reverse geocoding


```python
points = geo[["geometry"]].copy()
points.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>geometry</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>POINT (24.91556 60.1632)</td>
    </tr>
    <tr>
      <th>1</th>
      <td>POINT (24.93009 60.16846)</td>
    </tr>
    <tr>
      <th>2</th>
      <td>POINT (24.94153 60.17016)</td>
    </tr>
    <tr>
      <th>3</th>
      <td>POINT (24.97675 60.19438)</td>
    </tr>
    <tr>
      <th>4</th>
      <td>POINT (24.92151 60.15662)</td>
    </tr>
  </tbody>
</table>
</div>




```python
points2 = geo2[["geometry"]].copy()
points2.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>geometry</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>POINT (-106.70813 35.14749)</td>
    </tr>
    <tr>
      <th>1</th>
      <td>POINT (-83.7399 42.28055)</td>
    </tr>
    <tr>
      <th>2</th>
      <td>POINT (-74.15647 41.03768)</td>
    </tr>
  </tbody>
</table>
</div>




```python
from geopandas.tools import reverse_geocode

reverse_geocoded = reverse_geocode(
    points2.geometry, provider="nominatim", user_agent="biscotty", timeout=10
)
reverse_geocoded.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>geometry</th>
      <th>address</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>POINT (-106.70813 35.14749)</td>
      <td>6119, Mustang Lane Northwest, Albuquerque, Ber...</td>
    </tr>
    <tr>
      <th>1</th>
      <td>POINT (-83.7399 42.28055)</td>
      <td>North Quad Dining Hall, 105, South State Stree...</td>
    </tr>
    <tr>
      <th>2</th>
      <td>POINT (-74.15647 41.03768)</td>
      <td>5, Leona Terrace, Mahwah, Bergen County, New J...</td>
    </tr>
  </tbody>
</table>
</div>




```python

```
