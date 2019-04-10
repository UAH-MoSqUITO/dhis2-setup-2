
# featuretype by string manipulation

```shell
psql -h localhost -U dhis -d dhis2 -v ON_ERROR_STOP=1 -c "SELECT jsonb_build_object('type','Feature','geometry',jsonb_build_object('type',array_to_string(array(SELECT initcap(unnest(regexp_split_to_array(featuretype,'_')))),''),'coordinates',coordinates::jsonb),'properties',jsonb_build_object('name',name))::text FROM organisationunit WHERE featuretype <> 'NONE';" -A -t > organisationunits.geojsonlines
```

```sql
SELECT jsonb_build_object(
    'type', 'Feature',
    'geometry', jsonb_build_object(
        'type', array_to_string(
            array(
                SELECT initcap(
                    unnest(
                        regexp_split_to_array(featuretype,'_')
                    )
                )
            ),
            ''
        ),
        'coordinates', coordinates::jsonb
    ),
    'properties', jsonb_build_object(
        'name',
        name
    )
)::text
FROM organisationunit
WHERE featuretype <> 'NONE';
```

# featuretype by condition

```shell
psql -h localhost -U dhis -d dhis2 -v ON_ERROR_STOP=1 -c "SELECT jsonb_build_object('type','Feature','geometry',jsonb_build_object('type',CASE featuretype WHEN 'MULTI_POLYGON' THEN 'MultiPolygon' WHEN 'POLYGON' THEN 'MultiPolygon' WHEN 'POINT' THEN 'Point' END,'coordinates',coordinates::jsonb),'properties',jsonb_build_object('name',name))::text FROM organisationunit WHERE featuretype <> 'NONE';" -A -t > organisationunits.geojsonlines
```

```sql
SELECT jsonb_build_object(
    'type', 'Feature',
    'geometry', jsonb_build_object(
        'type', CASE featuretype
			WHEN 'MULTI_POLYGON' THEN 'MultiPolygon'
			WHEN 'POLYGON' THEN 'MultiPolygon'
			WHEN 'POINT' THEN 'Point'
		END,
        'coordinates', coordinates::jsonb
    ),
    'properties', jsonb_build_object(
        'name',
        name
    )
)::text
FROM organisationunit
WHERE featuretype <> 'NONE';
```


# split into separate files based on depth of JSON array in coordinates

Note: GDAL 2.2.3 (Ubuntu 18.04) does not add all entries from the geojsonlines files into the other types. GDAL 2.4.1 does.

```shell
sudo apt install gdal-bin
```

```shell
(mark='' ; for t in Point skip Polygon MultiPolygon ; do mark=$mark'[' ; test "$t" = skip && continue ; echo "$t  $mark" ; psql -h localhost -U dhis -d dhis2 -v ON_ERROR_STOP=1 -c "SELECT jsonb_build_object('type','Feature','geometry',jsonb_build_object('type','$t','coordinates',coordinates::jsonb),'properties',jsonb_build_object('name',name))::text FROM organisationunit WHERE regexp_match(coordinates,'^([[]*)') = ARRAY['$mark'];" -A -t > organisationunits.$t.geojsonlines ; done)
```

## Example for multipolygon

```sql
SELECT jsonb_build_object(
    'type', 'Feature',
    'geometry', jsonb_build_object(
        'type', 'MultiPolygon',
        'coordinates', coordinates::jsonb
    ),
    'properties', jsonb_build_object(
        'name', name
    )
)::text
FROM organisationunit
WHERE regexp_match(coordinates,'^([[]*)') = ARRAY['[[[['];
```

_Log_

```console
vagrant@vagrant:/vagrant$ ( table=organisationunit ; mark= ; for featuretype in Point skip Polygon MultiPolygon ; do mark=$mark\[ ; test "$featuretype" = skip && continue ; name=$table.$featuretype ; gjl=$name.geojsonlines ; echo "$featuretype  $mark" ; psql -h localhost -U dhis -d dhis2 -v ON_ERROR_STOP=1 -c "SELECT jsonb_build_object('type','Feature','geometry',jsonb_build_object('type','$featuretype','coordinates',coordinates::jsonb),'properties',jsonb_build_object('name',name))::text FROM $table WHERE regexp_match(coordinates,'^([[]*)') = ARRAY['$mark'];" -A -t > "$gjl" ; test -s "$gjl" || continue ; ogr2ogr -skipfailures -f "ESRI Shapefile" "$name.shp" "$gjl" ; ogr2ogr -skipfailures -f "GPKG" "$name.gpkg" "$gjl" ; done )
Point  [
Polygon  [[[
MultiPolygon  [[[[
vagrant@vagrant:/vagrant$ wc -l organisationunit.*.geojsonlines
    165 organisationunit.MultiPolygon.geojsonlines
    601 organisationunit.Point.geojsonlines
      0 organisationunit.Polygon.geojsonlines
    766 total
vagrant@vagrant:/vagrant$ ll -d organisationunit.*
-rw-r--r-- 1 vagrant vagrant    147 Apr 10 01:21 organisationunit.MultiPolygon.dbf
-rw-r--r-- 1 vagrant vagrant 962529 Apr 10 01:21 organisationunit.MultiPolygon.geojsonlines
-rw-r--r-- 1 vagrant vagrant 135168 Apr 10 01:21 organisationunit.MultiPolygon.gpkg
-rw-r--r-- 1 vagrant vagrant    143 Apr 10 01:21 organisationunit.MultiPolygon.prj
-rw-r--r-- 1 vagrant vagrant  19344 Apr 10 01:21 organisationunit.MultiPolygon.shp
-rw-r--r-- 1 vagrant vagrant    108 Apr 10 01:21 organisationunit.MultiPolygon.shx
-rw-r--r-- 1 vagrant vagrant    147 Apr 10 01:21 organisationunit.Point.dbf
-rw-r--r-- 1 vagrant vagrant  76443 Apr 10 01:21 organisationunit.Point.geojsonlines
-rw-r--r-- 1 vagrant vagrant 118784 Apr 10 01:21 organisationunit.Point.gpkg
-rw-r--r-- 1 vagrant vagrant    143 Apr 10 01:21 organisationunit.Point.prj
-rw-r--r-- 1 vagrant vagrant    128 Apr 10 01:21 organisationunit.Point.shp
-rw-r--r-- 1 vagrant vagrant    108 Apr 10 01:21 organisationunit.Point.shx
-rw-r--r-- 1 vagrant vagrant      0 Apr 10 01:21 organisationunit.Polygon.geojsonlines
```



```console
dhis2-setup-2 $ rm organisationunit.Polygon.geojsonlines
dhis2-setup-2 $ for x in organisationunit.*.geojsonlines ; do n=${x%.*} ; for y in $n.shp $n.gpkg ; do ogr2ogr -skipfailures $y $x ; done ; done
dhis2-setup-2 $ gdalinfo --version
GDAL 2.4.1, released 2019/03/15
dhis2-setup-2 $ for x in organisationunit.*.shp ; do n=${x%.*} ; zip $n.zip $x $n.shx $n.prj $n.dbf ; done
  adding: organisationunit.MultiPolygon.shp (deflated 61%)
  adding: organisationunit.MultiPolygon.shx (deflated 31%)
  adding: organisationunit.MultiPolygon.prj (deflated 15%)
  adding: organisationunit.MultiPolygon.dbf (deflated 91%)
  adding: organisationunit.Point.shp (deflated 52%)
  adding: organisationunit.Point.shx (deflated 73%)
  adding: organisationunit.Point.prj (deflated 15%)
  adding: organisationunit.Point.dbf (deflated 91%)
```
