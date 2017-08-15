# whosonfirst-intersections

How we generate intersections for Who's On First

## Important

This is work in progress. Specifically we are trying to figure out to operationalize generating intersections. In other words this is wet paint. Proceed with caution.

## Workflow

1. Find source data
2. Prepare and import source data into Postgres with `shp2pgsql`
3. Convert all the road segments to unified line strings
* Find the list of unique street names
* Dissolve line segments for each street name in to a single linestring
* Store linestrings in a new table
4. Find all the intersections for the set of linestrings as points
5. Geneate Voronoi polygons from the set of resulting points
6. Clip polygons to a container Who's On First polygon (typically a locality)

## Databases

Something something something. Databases. Something something something. Schemas need to be written. Something something something.

### Database A

Database `A` is where source data is stored. It contains a `geom` field that is whatever `shp2pgsql` decides it needs and an arbitrary number of properties.

### Database B

Database `B` is where the union of line segments (for roads) derived from database `A` is stored. It contains three fields: a WOF ID representing the locality for roads; a street name; a `geom` field that is a `LineString`.

### Database C

Database `C` is where all the intersections derived from database `B` are stored. It contains four fields: a WOF ID representing the locality for each intersections; street name `X` and street name `Y` (the intersection); a `geom` field that is a `Point`.

### Database D

Database `D` is where all the Voronoi polygons derived from the points in database `C` are stored. It contains four fields: a WOF ID representing the locality for each intersections; street name `X` and street name `Y` (the intersection); a `geom` field that is a `Polygon`.

_TBD: Whether database `D` contains only the raw Voronoi polygons or the clipped (to a WOF locality) polygons. Probably the latter._

## Example

### Setup

Run the `whosonfirst_postgis::default` Chefworks recipe. If you don't work at Mapzen basically that means "install PostGIS".

```
createdb roads
psql -c "CREATE EXTENSION postgis; CREATE EXTENSION postgis_topology; CREATE EXTENSION btree_gist;" roads
```

### Los Angeles

#### Step 1

Download https://www.census.gov/cgi-bin/geo/shapefiles/index.php?year=2016&layergroup=Roads

#### Step 2

```
shp2pgsql -s 4326 tl_2016_06037_roads.shp la_roads > la_roads.sql
psql -d roads -f la_roads.sql
```

#### Step 3

```
SELECT fullname, COUNT(gid) AS count FROM la_roads GROUP BY fullname HAVING fullname != ''
SELECT ST_AsText(ST_Union(geom)) FROM la_roads WHERE fullname = '{FULLNAME}';
```

_This is as far as we've gotten..._

## See also

* https://github.com/whosonfirst-data/whosonfirst-data-intersection-us-ny
