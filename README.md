# Geospatial learning

**Geospatial** - Locations on Earth

## GeoJSON Data Format

GeoJSON unsurprisingly is a standard for representing locations in JSON.

Each location is called a feature.

A feature has a key/value collection of properties, which can be used to describe what is at the location (e.g. name, identifier, age, categorisation).

Location information is defined using Longitude and Latitude coordinates.

```geojson
{
  "type": "FeatureCollection",
  "features": [{
    "type": "Feature",
    "properties": {
        "name": "Really important parking space",
        "year_constructed": "1882",
        "favourite_cake": "carrot",
        "type": "parking_space"
    },
    "geometry": {
        "type": "Polygon",
        "coordinates": [
            [
                [
                    -0.1086403,
                    51.5341233
                ],
                [
                    -0.1086753,
                    51.5341219
                ],
                [
                    -0.1086705,
                    51.5340754
                ],
                [
                    -0.1086354,
                    51.5340768
                ],
                [
                    -0.1086403,
                    51.5341233
                ]
            ]
        ]
    },
    "id": "22212"
  }]
}
```

The https://geojson.io website has functionality to create your own files, and visualise existing GeoJSON.

## Web Mapping Libraries

Leaflet is an open source framework written in JS and CSS for adding maps to your website.

A map on a website will usually be a combination of a baselayer layered with some features mapped on top.
The baselayer provides a visual reference allowing users to understand where features are.

OpenMapTiles, an open source map server, has a [gallery of different styles](https://openmaptiles.org/styles/).
Matching the design of your website to the baselayer helps to create a consistent look and feel.


### Examples of web maps

- [Croydon parks and playgrounds directory](https://www.croydon.gov.uk/libraries-leisure-and-culture/parks-and-open-spaces/parks-and-playgrounds/parks-and-playgrounds-directory) - Individual points
- [Wildfire map](https://www.mapofire.com/#5/43.67/-117.15) - Areas affected by fire

### Practise

Create a map showing a rectangle around the City of Winchester using [this Codepen as a starting point](https://codepen.io/adrianclay/pen/XJdbxxE?editors=0010).
[geojson.io](https://geojson.io) can help with creating the rectangle location coordinates.

## Coordinates

The World Geodetic System 1984 (WGS 84) is a 3-dimensional coordinate reference against which latitude, longitude and altitude are plotted.

[Geographic Coordinate Systems 101: A Primer for Software Generalists](https://8thlight.com/insights/geographic-coordinate-systems-101) provides some useful explanation of why we need standards for coordinate references.

Another coordinate system you may be familiar with is the Ordnance Survey National Grid reference system, [e.g. The Made Tech office at TQ33545819](https://britishnationalgrid.uk/#TQ335819).

## OpenStreetMap

Creating your own map from scratch would take a very very very long time.
OpenStreetMap is a source of free Open Map Data, with the only restriction that attribution is provided.
The $1.8 trillion company [Meta use this free data to power maps](https://developers.meta.com/maps/) on their website and apps.

OpenStreetMap can also be used to search for points of interest, for example this [list of parking spaces in London was extracted from OpenStreetMap](./parking_spaces.geojson) by querying the Overpass turbo wizard `amenity=parking_space in London`.

### Practise

Search for all [memorial=blue_plaque](https://wiki.openstreetmap.org/wiki/Tag:memorial%3Dblue_plaque) in London (or your preferred city) using [Overpass turbo](https://overpass-turbo.eu/).

## Geometries inside GeoJSON

We've explored points and polygons, but there are [lots of other geometries](https://en.wikipedia.org/wiki/GeoJSON#Geometries).

### Practise

TODO: Quiz using markdown GeoJSON Support.

## Geospatial database support

Lots of databases have built-in support for geospatial querying, but don't assume each implementation is equivilent in functionality.

- PostgreSQL has an [extension called PostGIS](https://postgis.net/)
- OpenSearch has [Geographic queries](https://docs.opensearch.org/latest/query-dsl/geo-and-xy/index/#geographic-queries)

### Practise

Using the [DuckDB Shell](https://shell.duckdb.org/) search for the parking spaces closest to Made Techs London office.

```sql

INSTALL spatial;
LOAD spatial;

.files add

SET VARIABLE made_tech_office = ST_Point(-0.07621863117158646, 51.520877510209274);


-- Closest parking spaces to the Made Tech office.
SELECT
    id,
    CEIL(ST_Distance_Sphere(ST_Centroid(geom), getvariable('made_tech_office'))) AS distance
    FROM ST_Read('parking_spaces.geojson')
    ORDER BY distance ASC
    LIMIT 10;

-- Total area (square meters) of parking spaces
SELECT
    SUM(ST_Area_Spheroid(geom)) AS total_area
    FROM ST_Read('parking_spaces.geojson');
```

## Geocoding and reverse geocoding

Geocoding is converting a human readable location into a latitude/longitude.
OpenStreetMap provide a free to use HTTP API for low usage scenarios.

```shell
curl "https://nominatim.openstreetmap.org/search?q=E1+6BX&limit=2&format=json" | json_pp
```

```json
[
   {
      "addresstype" : "postcode",
      "boundingbox" : [
         "51.5190523",
         "51.5230523",
         "-0.0797462",
         "-0.0757461"
      ],
      "class" : "place",
      "display_name" : "E1 6BX, London Borough of Tower Hamlets, London, Greater London, England, United Kingdom",
      "importance" : 0.0666766666666667,
      "lat" : "51.5210523",
      "licence" : "Data © OpenStreetMap contributors, ODbL 1.0. http://osm.org/copyright",
      "lon" : "-0.0777461",
      "name" : "E1 6BX",
      "place_id" : 350605768,
      "place_rank" : 25,
      "type" : "postcode"
   }
]
```

Reverse geocoding is the process of converting a latitude/longitude into a human readable location.

```shell
curl "https://nominatim.openstreetmap.org/reverse?lat=51.503375&lon=-0.127679&zoom=19&format=json" | json_pp
```

```json
{
   "address" : {
      "ISO3166-2-lvl4" : "GB-ENG",
      "city" : "London",
      "country" : "United Kingdom",
      "country_code" : "gb",
      "house_number" : "10",
      "office" : "10 Downing Street",
      "postcode" : "SW1A 2AA",
      "quarter" : "Westminster",
      "road" : "Downing Street",
      "state" : "England",
      "state_district" : "Greater London",
      "suburb" : "Millbank"
   },
   "addresstype" : "office",
   "boundingbox" : [
      "51.5033074",
      "51.5036913",
      "-0.1277991",
      "-0.1273088"
   ],
   "class" : "office",
   "display_name" : "10 Downing Street, 10, Downing Street, Westminster, Millbank, London, Greater London, England, SW1A 2AA, United Kingdom",
   "importance" : 0.545464825148363,
   "lat" : "51.5034878",
   "licence" : "Data © OpenStreetMap contributors, ODbL 1.0. http://osm.org/copyright",
   "lon" : "-0.1276965",
   "name" : "10 Downing Street",
   "osm_id" : 1879842,
   "osm_type" : "relation",
   "place_id" : 259858018,
   "place_rank" : 30,
   "type" : "government"
}
```

## Routing algorthims

Routing algorithms can take:
- a network of road locations
- a starting location
- an ending location

And generate a set of directions of how to get from the start to end via the road network.

```shell
curl "https://routing.openstreetmap.de/routed-foot/route/v1/driving/0.01908,51.47981;0.0248199,51.4719?overview=false&geometries=polyline&steps=true" | json_pp
```