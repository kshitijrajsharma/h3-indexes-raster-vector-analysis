# h3-indexes-raster-vector-analysis
Using h3 indexes for raster and vector analysis, State of the Map 2025


## Resources 

- h3geo : https://h3geo.org/ 
- Performance Comparison : https://github.com/kshitijrajsharma/pg_benchmark_A4/blob/master/example.ipynb
- DB Schema used from : https://github.com/hotosm/raw-data-api 
- Raster Analysis to get h3 cells from COG : https://github.com/kshitijrajsharma/cog2h3/tree/main 
- Read Blog : https://dev.to/krschap/raster-analysis-using-uber-h3-indexes-and-postgresql-57g9
- Osm2pgsql : https://osm2pgsql.org/ 
- Slides : Coming Up 

## Setup 

1. Install osm2pgsql

```bash
sudo apt install osm2pgsql
```

2. Get raw data api

You only need the .lua script though 

```bash
git clone https://github.com/hotosm/raw-data-api.git
```

3. Download OSM Data
```bash
wget https://planet.openstreetmap.org/pbf/planet-latest.osm.pbf
```

4. Install h3 deps

```bash
sudo apt install pgxnclient cmake
```

6. Fire up the data loading

```log
(env) krschap@krs-tuxedo:~/hotosm/raw-data-api/backend$ osm2pgsql \
  --create \
  --slim \
  --flat-nodes=/var/tmp/flat-nodes.bin \
  --extra-attributes \
  --output=flex \
  --style /home/krschap/hotosm/raw-data-api/backend/raw.lua \
  --cache=20000 \
  --number-processes=8 \
  ./data/planet-latest.osm.pbf
2025-06-16 15:40:17  osm2pgsql version 1.11.0
2025-06-16 15:40:17  Database version: 16.9 (Ubuntu 16.9-0ubuntu0.24.04.1)
2025-06-16 15:40:17  PostGIS version: 3.4
2025-06-16 15:40:17  Storing properties to table '"public"."osm2pgsql_properties"'.
Processing: Node(9964518k 1815.4k/s) Way(1110570k 94.71k/s) Relation(13410520 1069.8/s)
2025-06-16 23:56:10  Writing 1169438 entries to table 'planet_osm_users'...
2025-06-16 23:56:11  Reading input files done in 29753s (8h 15m 53s).                     
2025-06-16 23:56:11    Processed 9964518268 nodes in 5489s (1h 31m 29s) - 1815k/s
2025-06-16 23:56:11    Processed 1110570899 ways in 11726s (3h 15m 26s) - 95k/s
2025-06-16 23:56:11    Processed 13411680 relations in 12538s (3h 28m 58s) - 1k/s
2025-06-16 23:56:11  No marked ways (Skipping stage 2).
2025-06-16 23:56:13  Done postprocessing on table 'planet_osm_nodes' in 0s
2025-06-16 23:56:13  Building index on table 'planet_osm_ways'
2025-06-16 23:56:13  Clustering table 'nodes' by geometry...
2025-06-16 23:56:13  Building index on table 'planet_osm_rels'
2025-06-16 23:56:13  Clustering table 'ways_poly' by geometry...
2025-06-16 23:56:13  Clustering table 'relations' by geometry...
2025-06-16 23:56:13  Clustering table 'ways_line' by geometry...
2025-06-17 00:30:22  Creating index on table 'nodes' ("geom")...
2025-06-17 00:41:02  Creating id index on table 'nodes'...
2025-06-17 00:43:32  Analyzing table 'nodes'...
2025-06-17 01:02:09  Creating index on table 'relations' ("geom")...
2025-06-17 01:05:50  Creating id index on table 'relations'...
2025-06-17 01:06:01  Creating index on table 'ways_line' ("geom")...
2025-06-17 01:06:10  Analyzing table 'relations'...
2025-06-17 01:19:00  Creating id index on table 'ways_line'...
2025-06-17 01:22:16  Analyzing table 'ways_line'...
2025-06-17 01:56:23  Creating index on table 'ways_poly' ("geom")...
2025-06-17 02:21:11  Creating id index on table 'ways_poly'...
2025-06-17 02:26:52  Analyzing table 'ways_poly'...
2025-06-17 03:20:17  Done postprocessing on table 'planet_osm_ways' in 12243s (3h 24m 3s)
2025-06-17 03:20:17  Done postprocessing on table 'planet_osm_rels' in 667s (11m 7s)
2025-06-17 03:20:17  All postprocessing on table 'nodes' done in 2841s (47m 21s).
2025-06-17 03:20:17  All postprocessing on table 'ways_line' done in 5165s (1h 26m 5s).
2025-06-17 03:20:17  All postprocessing on table 'ways_poly' done in 9041s (2h 30m 41s).
2025-06-17 03:20:17  All postprocessing on table 'relations' done in 4199s (1h 9m 59s).
2025-06-17 03:20:17  Storing properties to table '"public"."osm2pgsql_properties"'.
2025-06-17 03:20:17  osm2pgsql took 42000s (11h 40m 0s) overall.
```
6. Enable POSTGIS and h3 extension
```sql
Create extension postgis;
create extension h3;
create extension h3_postgis CASCADE;
```


7. Update the h3 column in your table

You can use this script for large data https://github.com/hotosm/raw-data-api/blob/develop/backend/field_update or simply fire up the query to populate them

```bash
python field_update --target ways_poly
```
or 
```sql
ALTER TABLE buildings ADD COLUMN h3 h3index GENERATED ALWAYS AS (h3_lat_lng_to_cell(ST_Centroid(geom), 6)) STORED;
```

8. Enjoy 

## Connect 

Connect with Speakers : 
Deepak Pradhan 
Kshitij Raj Sharma 
