# whosonfirst-intersections

How we generate intersections for Who's On First

## Important

This is work in progress. Specifically we are trying to figure out to operationalize generating intersections. In other words this is wet paint. Proceed with caution.

## Workflow

1. Find source data
2. Prepare and import source data into Postgres with `shp2pgsql`
3a. Find the list of unique street names
3b. Dissolve line segments for each street name in to a single linestring
3c. Store linestrings in a new table
4. Find all the intersections for the set of linestrings as points
5. Geneate Voronoi polygons from the set of resulting points
6. Clip polygons to a container Who's On First polygon (typically a locality)

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
