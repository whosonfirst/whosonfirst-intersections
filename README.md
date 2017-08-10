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

### setup

Run the `whosonfirst_postgis::default` Chefworks recipe. If you don't work at Mapzen basically that means "install PostGIS".

```
createdb roads
psql -c "CREATE EXTENSION postgis; CREATE EXTENSION postgis_topology; CREATE EXTENSION btree_gist;" roads
```

### Los Angeles

* https://www.census.gov/cgi-bin/geo/shapefiles/index.php?year=2016&layergroup=Roads

```
shp2pgsql -s 4326 -T tl_2016_06037_roads.shp > la_roads.sql
perl -p -i -e 's/tl_2016_06037_roads/la_roads/g' la_roads.sql
psql -d roads -f la_roads.sql
```

* https://gis.stackexchange.com/questions/209713/postgis-dissolve-geometries-from-shapefiles
* https://gis.stackexchange.com/questions/17495/dissolve-or-unsplit-lines-on-common-attributes-in-postgis-or-grass

```
SELECT fullname, COUNT(gid) AS count FROM la_roads GROUP BY fullname HAVING fullname != ''
SELECT ST_AsText(ST_Union(geom)) FROM la_roads WHERE fullname = '4th St';
```

## See also

* https://github.com/whosonfirst-data/whosonfirst-data-intersection-us-ny
