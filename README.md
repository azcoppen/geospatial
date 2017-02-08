Storing & Searching Geometric Spatial Data
===================

By Alex Coppen

Background on Cartography
-------------
The Earth is not 2D flat, or even a 3D sphere. It is commonly referred to as a oblate spheroid, which is like a sphere that has been squashed at the top and bottom. When squashed flat onto a piece of paper on a desk (i.e. a map), it requires a 2D system to identify points on its surface, which appears as a grid. That grid is represented in data the same as any other you would draw.

The system of using **longitude** and **latitude** creates an X,Y grid over a flattened map of the world. These are just ordinary floating point decimal numbers, but measuring the journey between two sets typically uses the *Haversine Formula* (https://en.wikipedia.org/wiki/Haversine_formula) to determine the approximate *Great Circle Distance* (https://en.wikipedia.org/wiki/Great-circle_distance) "as the crow flies".

![enter image description here](http://www.colorado.edu/geography/gcraft/notes/coordsys/gif/georef.gif)

Longitude is the horizontal or X axis that runs from east to west, from 180 degrees to -180 degrees, through the Greenwich (Prime) Meridian at 0 degrees.

Latitude is the vertical or Y axis that runs from north to south, from 90 degrees to -90 degrees, through the Equator at 0 degrees.

- Longitude: **X** grid axis, 180 (map right) to -180 (map left)
- Latitude: **Y** grid axis, 90 (map top) to -90 (map bottom)

![enter image description here](https://camo.githubusercontent.com/42e804c7b47100b68746f5966defc85d406bb5d2/68747470733a2f2f7261772e6769746875622e636f6d2f76656c746d616e2f6c6561726e696e676c756e636865732f6d61737465722f6d6170732f696d616765732f6c61746c6e672e706e67)

![enter image description here](http://freegeographytools.com/wp-content/uploads/2010/09/nyc.jpg)

####Example Coordinates
- New York City: *-74.005941, 40.712784* (X/Lng, Y/Lat)
- London: *-0.127758, 51.507351* (X/Lng, Y/Lat)
- Beijing: *116.407395, 39.904211* (X/Lng, Y/Lat)

https://en.wikipedia.org/wiki/Geographic_coordinate_system

> **Note:**

> - The ellipsoid geography of the Earth obviously doesn't make any of this as simple as it is presented here for real navigational purposes, or the relationship of planetary orbit with time. The explanation above is simply to relate the geometry to how it is stored in a database.

Background On Postal Areas
-------------
Postal districts (or zip codes in the US) are typically area measurements of distributed human population used by postal services for categorising mail for delivery to inhabited zones. The information is reviewed from a periodic national census.

Governments and postal services refer to these areas in technical terms, rather than "postal" shapes.

####United States
In the US, these zones are called **Zip Code Tabulation Areas** (ZCTAs). A zipcode has 5 numeric digits. There are approximately 45,000.

In the US, a zipcode has one shape.

![enter image description here](https://tableaumapping.files.wordpress.com/2013/11/usgs-tiger-zcta-2010.png)

####Canada
In Canada, postal codes follow the British system, and can be either 3 characters, or 6 characters (e.g A0A, or A0A 1AF). These zones are called **Forward Sortation Areas** (FSAs). There are approximately 1,700 3-digit wider codes, which dissolve into over 830,000 6-digit variants for specificity.

In Canada, a postal code can be a *collection* of neighbouring shapes, or polygons. For example, the postcode *A0A* has 5+ separate geographic areas.

![enter image description here](https://www.maptechnica.com/img/carousel/ca-fsa-area-tornto.jpg)

####Centres & Boundaries
Each postal or zip code is given a centre point of longitude and latitude.

Each postal or zip code has an approximate shape or border (or set of them), known as its **boundaries**, which can be simplistically represented on a map as a **polygon** (or set of them). These can overlap, and should never be considered forensically accurate.

New postal areas are added every quarter, with the best records being held by the postal services of each country.

Sourcing the Data
-------------
Getting hold of postal data can be extremely difficult, because a) in many cases it is subject to postal service copyright, and b) it is referred to in technical terms (e.g. *"FSA boundaries"*).

it is important to understand that the data is generated, stored, and manipulated for **Geographic Information System** (GIS) programs, which use both compiled and textual formats, such as shape files (.shp), and geography markup (.gml). Newer formats include Keyhole (.kml) and GeoJSON (.json).

![enter image description here](https://infogeoblog.files.wordpress.com/2012/10/qgis-introduction.png)

Well-known platforms for opening, viewing, and converting layers of these data files are ArcGIS, OpenGIS, and qGIS (Mac).

More: https://en.wikipedia.org/wiki/Geographic_information_system

####Free Government Data
Most governments publish geographic area data after they take a periodic census. This data has been imported and adapted frequently into Google FusionTables.

**United States ZCTAs **:

 - http://www.census.gov/geo/maps-data/data/tiger-kml.html
 - https://www.google.com/fusiontables/DataSource?docid=1Lae-86jeUDLmA6-8APDDqazlTOy1GsTXh28DAkw#rows:id=1

**Canada FSAs**:

- https://www12.statcan.gc.ca/census-recensement/2011/geo/bound-limit/bound-limit-2011-eng.cfm
- https://www.google.com/fusiontables/DataSource?docid=1H_cl-oyeG4FDwqJUTeI_aGKmmkJdPDzRNccp96M&hl=en_US&pli=1

####Private Paid Data
Many companies offer much more detailed and curated information that is updated monthly, and can be licensed annually for a cost of $2-7000 USD.

- http://www.maponics.com/international-postal-boundaries
- http://www.geoplan.com/product/data/index.html
- http://www.mbi-geodata.com/en/area-boundaries/postal-zip/

####Importing GIS data
In many cases, the easiest way to manipulate the data is from XML formats that can deserialized into programmatically-accessible objects (e.g. traversible DOM). Several of both these input and output files can be **hundreds of megabytes** in size,  with **over 5000** lng/lat points that comprise a single boundary.

The simplest route to an XML-based source is to open the data file as a layer in qGIS (Mac), and export/convert it to either KML or GeoJSON for parsing inside your application.

Storing Spatial Data In MySQL
-------------
MySQL has a sweet GIS extension for storing information about points, lines, and shapes that need to be represented on a grid, and searched. The database engine stores the data natively, but has functions to retrieve it as Well-Known Text (WKT).

![enter image description here](http://paulbourke.net/miscellaneous/povexamples/grid.gif)

The grid being search through runs from 90 to -90 on the Y axis, and -180 to 180 on the X axis. Most MySQL documentation tends to refer to arbitrary values such as 1-10, or 0,0, and so on. There is no difference between 0,0 or 2,5, or -90 to +180 - they are just X and Y axis values for a grid.

The most used of these is the "Geometry-From-Text" function:

```sql
SELECT ST_AsText(field_name) FROM table;
INSERT INTO table (field_name) VALUES (GeomFromText("POINT(X,Y)"))

```

#### Points
The simplest form of location that can be stored is the POINT field, which is expressed as POINT(X,Y), or in geographic terms: POINT(LNG, LAT).

![enter image description here](http://sphinxsearch.com/blog/wp-content/uploads/2013/07/poly2d_large_polygon.png)

```sql
POINT(X,Y)
POINT(LNG,LAT)
```

```sql
INSERT INTO table (city, coords) 
VALUES (
  'New York', 
  GeomFromText("POINT(-74.005941, 40.712784)")
);
```
Retrieving the data is equally simple.

```sql
SELECT ST_AsText(coords) FROM table;
SELECT X(coords) FROM table AS longitude;
SELECT Y(coords) FROM table AS latitude;
```

#### Polygons
Polygons are a little more tricky, as they need to close themselves. A polygon comprises a list of X,Y grid values. The last X,Y values of the list *must* be the same as the first in order to self-close the shape. If not, the shape will evaluate to NULL.

![enter image description here](http://sphinxsearch.com/blog/wp-content/uploads/2013/07/geopoly2d-small.png)

A polygon has an inner and outer ring (two parentheses), and can be expressed simply as:

```sql
POLYGON(( X1 Y1, X2 Y2, X3 Y3, X4 Y4, X1 Y1 ))
```

```sql
INSERT INTO table (area_name, shape_field) VALUES (
   'My Area Name',
   PolyFromText(" 
     POLYGON((LON1 LAT1, LON2 LAT2, LON3 LAT3, LON4 LAT4,     LON1 LAT1)) ")
);
```

Retrieving the data again requires the function to turn it back into text:

```sql
SELECT ST_AsText(shape_field) FROM table;
```

#### Multi-Polygons
Multi-polygons are essentially a list of polygons to store grouped in one field, which makes them **ideally suited for postal areas**.

![enter image description here](http://i.stack.imgur.com/BCbCy.png)

Polygons are simply separated by commas. but eventually have **three** parentheses.

```sql
MULTIPOLYGON(( 
   (X1 Y1, X2 Y2, X3 Y3, X4 Y4, X1 Y1),
   (X1 Y1, X2 Y2, X3 Y3, X4 Y4, X1 Y1),
   (X1 Y1, X2 Y2, X3 Y3, X4 Y4, X1 Y1)
))
```
You can store 1 polygon in a multi-polygon field, or many.

```sql
INSERT INTO table (area_name, shape_field) VALUES (
   'My Area Name',
   GeomFromText(" 
        MULTIPOLYGON((
       (LON1 LAT1, LON2 LAT2, LON3 LAT3, LON4 LAT4, LON1 LAT1),
       (LON1 LAT1, LON2 LAT2, LON3 LAT3, LON4 LAT4, LON1 LAT1),
       (LON1 LAT1, LON2 LAT2, LON3 LAT3, LON4 LAT4, LON1 LAT1)
        ))
     ")
);
```
Retrieving the data again requires the function to turn it back into text:

```sql
SELECT ST_AsText(shape_field) FROM table;
```

#### Others (Lines etc)
MySQL also allows you store many different types that can be useful for mapping and journeying applications:

- Arbitrary shapes (GEOMETRY)
- Multiple arbitrary shapes (GEOMETRYCOLLECTION)
- Lines from one place to another (LINESTRING)
- Multiple points (MULTIPOINT)
- Multiple lines (MULTILINESTRING)

Many of these are useful for determining whether 2 lines intersect/cross, or simply storing drawings set on a grid for a floor of a house.

> **Note:**

> - MySQL uses the X,Y format (Lng/Lat). Google Maps uses the opposite, or Y,X (Lat/Lng).
> - The first and last co-ordinates (X/Y, or Lng/Lat) of polygons must be the same, or they will evaluate to NULL.
> - Note the double parantheses: polygons require an **inner** and **outer** ring. Most often, these are the same.
> - Serializing raw database results into JSON or other formats will inject **illegal characters** into your HTML string output. You must use ST_AsText (field_name) format to retrieve and parse the Well-Known Text (WKT) value.

Spatially Querying Geometric Data 
-------------
All of this is well and good, but why would you want to go through all of this to store the information in the first place?

Almost all applications tend to use the same approach when it comes to geometry calculations with database resultsets: the Haversine Formula as a function or stored procedure for working out *distance*, and row-by-row *Point-In-Polygon* formulas for searching areas. 

![enter image description here](http://i.stack.imgur.com/1Gvlq.jpg)

This often comes from Google's own guide, which is frequently cited:
https://developers.google.com/maps/articles/phpsqlsearch_v3

```sql
SELECT id, 
( 3959 * acos( cos( radians(37) ) * cos( radians( lat ) ) * cos( radians( lng ) - radians(-122) ) + sin( radians(37) ) * sin( radians( lat ) ) ) ) AS distance 
FROM table 
HAVING distance < 25 
ORDER BY distance 
LIMIT 0, 20;
```

![enter image description here](https://coronalabs.com/wp-content/uploads/2015/01/Law-of-haversines.png)

This is generally bearable for store locators with less than 5000 rows, taking around 500ms. It's slightly different when it goes over 10,000 (without indexing FLOAT columns).

It also gets difficult when it comes to a table with 100+ polygon areas, stored assumedly as text or JSON.  For the latter, you are forced to retrieve your entire result base, and then programatically interrogate whether the specified lat/lng is geometrically contained in the polygon, vertex by terrible vertex. Just a single postal boundary can contain 1000 - 8000 vertices.

```php
function contains($point, $polygon)
{
    if($polygon[0] != $polygon[count($polygon)-1])
        $polygon[count($polygon)] = $polygon[0];
    $j = 0;
    $oddNodes = false;
    $x = $point[1];
    $y = $point[0];
    $n = count($polygon);
    for ($i = 0; $i < $n; $i++)
    {
        $j++;
        if ($j == $n)
        {
            $j = 0;
        }
        if ((($polygon[$i][0] < $y) && ($polygon[$j][0] >= $y)) || (($polygon[$j][0] < $y) && ($polygon[$i][0] >=
            $y)))
        {
            if ($polygon[$i][1] + ($y - $polygon[$i][0]) / ($polygon[$j][0] - $polygon[$i][0]) * ($polygon[$j][1] -
                $polygon[$i][1]) < $x)
            {
                $oddNodes = !$oddNodes;
            }
        }
    }
    return $oddNodes;
}
```


![enter image description here](http://idav.ucdavis.edu/~okreylos/TAship/Spring2000/PointInPolygon1.gif)

More: https://en.wikipedia.org/wiki/Point_in_polygon

Why write this when MySQL can do this natively, and much more - with indexing, for 500,000+ records, with all GIS functionality, in less than 50ms?

#### What's the distance between two points?

```sql
SELECT ST_DISTANCE(
    POINT(LNG1, LAT1), 
    POINT(LNG2, LAT2)
) AS map_degrees;
```
ST_DISTANCE measures the difference between the points in **map degrees**.  To get the value in actual length units, you need to multiple the result by the amount of degrees in that unit.

Each degree on a map is roughly 69.047 miles, or 111.32 km.

```sql
SELECT ST_DISTANCE(
    POINT(LNG1, LAT1), 
    POINT(LNG2, LAT2)
) * 69.047 AS miles;

SELECT ST_DISTANCE(
    POINT(LNG1, LAT1), 
    POINT(LNG2, LAT2)
) * 111.32 AS km;
```

#### What's the distance between a lat/lng and every row?

```sql
SELECT ST_DISTANCE(
    point_field, 
    POINT(-74.005941, 40.712784)
)
FROM table;
```

#### Which rows are within 50 miles nearby of a lat/lng?
```sql
SELECT *,
ST_DISTANCE(POINT(LON, LAT), point_field) * 69.047 AS miles
FROM table
WHERE ST_DISTANCE(POINT(LON, LAT), point_field) <= (50 / 69.047);
```

#### Is a lat/lng inside a polygon field value?

```sql
SELECT * FROM table
WHERE ST_CONTAINS(
    polygon_or_multipolygon_field, 
    POINT(-74.005941, 40.712784)
);
```

This function asks if argument 2 is within argument 1. The inverse function ST_WITHIN does the opposite.

#### How big is a polygon?
```sql
SELECT ST_AREA(polygon_field)
FROM table
WHERE id = 200
```

#### What's the 20th point of a polygon?
```sql
SELECT ST_AsText(
    ST_GEOMETRYN(ST_GEOMFROMTEXT(polygon_field), 20)
)
FROM table;
```

#### Is a lat/lng within a 50 unit radius of any row point field?

```sql
SELECT *, 
ST_DISTANCE(POINT(-74.005941, 40.712784), point_field) AS map_degrees
FROM table
WHERE ST_CONTAINS( 
    ST_BUFFER(point_field, 50), /* temporary area value */
    POINT(-74.005941, 40.712784) 
);
```

#### Which point rows are within the polygon area i specify?
```sql
SELECT *
FROM table
WHERE ST_WITHIN(
    point_field,
    PolyFromText("POLYGON((LON1 LAT1, LON2 LAT2, LON3 LAT3, LON4 LAT4,LON1 LAT1)) ")
);
```

#### Other GIS functions
There are dozens of useful ST_ functions available inside MySQL that allow you to query your world map grid in any way you like.

Examples include:

 - ST_AsGeoJSON()
 - ST_GeomFromGeoJSON()
 - ST_CENTROID
 - ST_CROSSES()
 - ST_DIFFERENCE()
 - ST_DIMENSION()
 - ST_DISJOINT()
 - ST_DISTANCE_SPHERE()
 - ST_EQUALS()
 - ST_GEOMETRYN()
 - ST_INTERSECTS()
 - ST_LENGTH()
 - ST_OVERLAPS()
 - ST_SIMPLIFY()
 - ST_TOUCHES()
 - ST_UNION()
 - 

See: 
 - https://dev.mysql.com/doc/refman/5.7/en/spatial-function-reference.html
 - https://dev.mysql.com/doc/refman/5.6/en/spatial-relation-functions.html
 - https://dev.mysql.com/doc/refman/5.7/en/spatial-function-reference.html

Speed With Indexing
-------------
Indexing any data inside an RDBMS is necessary for datasets over 1000 rows, and can massively accelerate queries. Spatial indexing is only currently supported on **MyISAM** tables, and not available with InnoDB.

To create a table with a spatial index:

```sql
CREATE TABLE geometries 
(
   postal_boundaries MULTIPOLYGON NOT NULL, 
   SPATIAL INDEX (postal_boundaries)
) 
ENGINE=MyISAM;
```

```sql
CREATE SPATIAL INDEX boundaries 
ON geometries (postal_boundaries);
```

> **Note:**

> - Spatial indexes in MySQL require that no rows are empty or NULL. You cannot create an index if one or more of your rows has a NULL value, or insert a new row with an empty spatial field.

Packages can Deserialize Spatial Values
-------------
One of the most useful PHP packages for loading textual/WKT MySQL values into an object-orientated programming environment is **GeoPHP**:

https://github.com/phayes/geoPHP

To include in your project:

```bash
composer require phayes/geoPHP dev-master
```

Or adding it to composer.json raw:

```json
"require": {
    "phayes/geoPHP": "dev-master"
}
```

Using it is also straightforward:

```php
$geo_field = \geoPHP::load($row->db_column, 'wkt');
$polygons  = $geo_field->getComponents()->asArray();
$area      = $geo_field->getArea();
$centroid  = $geo_field->getCentroid();
```
