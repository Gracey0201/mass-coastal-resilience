# PostGIS-Based Massachusetts Coastal Resilience Assessment
**PostGIS-Based Spatial Database and Coastal Vulnerability Analysis**

---

## Overview

This repository contains a spatial database and a GIS analysis workflow designed to examine coastal vulnerability, infrastructure exposure, and ecological habitat dynamics along the Massachusetts coastline. Using **PostGIS**, raster–vector integration, and spatial SQL, the project examines how critical infrastructure, healthcare access, land use, and sensitive ecological cores intersect within the coastal zone under climate-related stressors.

The project supports **coastal zone management, resilience planning, and environmental decision-making** by demonstrating how spatial databases can integrate heterogeneous geospatial datasets into a unified analytical framework.

---

## Project Objectives

- Assess the vulnerability of Massachusetts coastal areas to **sea level rise**, with an initial focus on a **1-meter inundation scenario**
- Identify **critical infrastructure**, residential areas, healthcare facilities, and ecological habitats located within the coastal zone
- Analyze spatial interactions between **human systems** and **natural ecosystems**
- Demonstrate the use of **PostGIS** for scalable raster–vector coastal analysis

> Due to data and performance constraints, the final analysis emphasizes **coastal infrastructure and habitat dynamics**, with limitations on inundation modeling documented below.

---

## Why This Matters

- Coastal Massachusetts supports dense development, transportation networks, healthcare services, and ecologically sensitive habitats.
- Understanding these spatial overlaps is essential for **risk reduction**, **adaptation planning**, and **equitable service access**.
- The project demonstrates how spatial databases can:
  - Quantify exposure
  - Identify vulnerable overlaps
  - Support evidence-based coastal planning

---

## Key Skills Demonstrated

- PostGIS raster and vector data management  
- Spatial indexing and query optimization (GiST)  
- Raster–vector overlay analysis  
- Buffer, proximity, and intersection analysis  
- Database normalization (1NF–4NF)  
- GIS decision-support workflow design  
- Integration of NOAA, MassGIS, and Google Earth Engine data  

---

## Data Sources

The project integrates publicly available datasets:

1. **Digital Elevation Model (DEM)** – [NOAA Sea Level Rise Viewer](https://coast.noaa.gov/slrdata/)  
2. **Aquatic Core Areas** – [MassGIS BioMap](https://www.mass.gov/info-details/massgis-data-biomap-the-future-of-conservation?)  
3. **Rare Species Core Areas** – [MassGIS BioMap](https://www.mass.gov/info-details/massgis-data-biomap-the-future-of-conservation?)  
4. **Forest Core Areas** – [MassGIS BioMap](https://www.mass.gov/info-details/massgis-data-biomap-the-future-of-conservation?)  
5. **Building Structures** – [MassGIS](https://www.mass.gov/info-details/massgis-data-building-structures-2-d?)  
6. **Cropland** – [MassGIS Land Cover](https://www.mass.gov/info-details/massgis-data-2016-land-coverland-use?)  
7. **Community Health Centers** – [MassGIS](https://www.mass.gov/info-details/massgis-data-community-health-centers)  
8. **Roads and Major Roads** – [MassDOT / MassGIS](https://www.mass.gov/info-details/massgis-data-massgis-massdot-roads)  
9. **Land Cover Raster** – [Google Earth Engine](https://code.earthengine.google.com/952954e5d84869f2683ab53aa0887724)  

Land cover data were downloaded and exported using Google Earth Engine:

![GEE code](https://github.com/Gracey0201/FinalProject/blob/main/GEE%20code.PNG)  
![GEE code2](https://github.com/Gracey0201/FinalProject/blob/main/GEE%20code2.PNG)

---

## Preprocessing

- Clipped all raster and vector datasets to the **Massachusetts coastal zone boundary** for performance optimization  
- Projected all data to **EPSG:26986 (NAD83 / Massachusetts Mainland)**  
- Preprocessing conducted in **QGIS**

![MA Coastal Map](https://github.com/Gracey0201/FinalProject/blob/main/LayersMap.png)  
![MA Coastal Map2](https://github.com/Gracey0201/FinalProject/blob/main/finapprojectmap.PNG)

---


## Methodology

The methodology for this project involved creating a PostGIS-based spatial database, preparing raster and vector datasets, normalizing tables, and performing spatial queries to analyze coastal vulnerability in Massachusetts.

### Database Setup
- Created the `Sealevelrise` database and enabled the PostGIS extension.
- Generated tables for both raster and vector datasets required for normalization and analysis.

### Shapefile Conversion to SQL
- All shapefiles were converted to SQL files using the `shp2pgsql` function.  
  Example command:

```bash
"C:\Users\default.DESKTOP-GCP9U73\OneDrive - Clark University\Documents\DATABASE MANAGEMENT\FinalProject">"C:\Program Files\PostgreSQL\16\bin\shp2pgsql" -s 4326 -I Data\Buildings.shp building_vector > building.sql

### Raster Conversion to SQL

-Raster datasets were converted to .sql files using the raster2pgsql function.
Example commands:

"C:\Users\default.DESKTOP-GCP9U73\OneDrive - Clark University\Documents\DATABASE MANAGEMENT\FinalProject">"C:\Program Files\PostgreSQL\16\bin\raster2pgsql" -s 4326 -t 1000x1000 -I -C -M  Data\Clipped_DEM.tif elevation > DEM.sql

"C:\Users\default.DESKTOP-GCP9U73\OneDrive - Clark University\Documents\DATABASE MANAGEMENT\FinalProject">"C:\Program Files\PostgreSQL\16\bin\raster2pgsql" -s 4326 -t 1000x1000 -I -C -M  Data\LULC_ClippeD.tif landcover > lulc.sql

### Importing SQL Files into PostgreSQL

-The generated vector and raster SQL files were imported into the Sealevelrise database using PgAdmin.
Example command:

pgsql -U postgres -d Flooding -f "C:\Users\rutha\OneDrive - Clark University\Documents\SpatialDatabase\FloodingProject\LocalVersion\boreholes.sql"

### Creating Cleaned Tables

-Empty tables were created for some vector layers (e.g., Aquatic Core) and populated with only the relevant columns for analysis.
Example: Aquatic Core Table

 ```SQL
CREATE TABLE aquaticcore_clean_vector(
    gid int PRIMARY KEY,
    shape_area numeric,
    shape_len numeric,
    geom GEOMETRY
);
```

-populate the new table with columns

 ```SQL
INSERT INTO floodwalls_clean_vector(gid, shape_area, shape_len,  geom)
SELECT gid, shape_area, shape_len,  geom
FROM aquaticcore_vector;
```
Rare Species Table

 ```SQL
CREATE TABLE rarespecies_clean_vector(
gid int PRIMARY KEY,
ac_ch_rare numeric,
town varchar(255),
ac_rscxtwn numeric,
shape_area numeric,
shape_len numeric,
geom GEOMETRY
);
```
- populate the new table with relevant columns

 ```SQL
INSERT INTO building_clean_vector(gid, ac_ch_rare, town, ac_rscxtwn, shape_area, shape_len, geom)
SELECT gid, ac_ch_rare, town, ac_rscxtwn, shape_area, shape_len, geom
FROM rarespecies_vector;
```

---
## Normalization of Tables

Database normalization is a systematic process for organizing data in a database to reduce redundancy, ensure data integrity, and improve query performance and maintenance efficiency.

### Reasons for Normalization

- Prevent redundancy in data storage.
- Simplify database structure for easier maintenance.
- Maintain consistent relationships between tables.
- Facilitate updates and minimize errors.

### Checking for Normalization

_All tables in this analysis are fully normalized to 1NF, 2NF, 3NF, and 4NF._

#### First Normal Form (1NF)
No table stores multiple values in a single cell, reducing complexity and improving clarity.

#### Second Normal Form (2NF)
All tables in 1NF have no partial dependencies; every non-key attribute depends on the full primary key, satisfying 2NF requirements.

#### Third Normal Form (3NF)
There are no transitive dependencies among non-prime attributes. Every non-key attribute depends directly on the primary key (`gid`), meeting 3NF criteria.

#### Fourth Normal Form (4NF)
No multi-valued dependencies exist. All attributes depend solely on the primary key (`gid`), ensuring 4NF compliance.

---

### Tables and Visuals

#### Aquatic Core Table
![Aquatic core table](https://github.com/Gracey0201/FinalProject/blob/main/Tables/Aquatic_core.PNG)

#### Coastal Zone Table
![Coastal Zone Table](https://github.com/Gracey0201/FinalProject/blob/main/Tables/Coastalzone.PNG)

#### Coastline Table
![Coastline Table](https://github.com/Gracey0201/FinalProject/blob/main/Tables/Coastline.PNG)

#### Cropland Table
![Cropland Table](https://github.com/Gracey0201/FinalProject/blob/main/Tables/Cropland.PNG)

#### Forest Core Table
![Forest Core Table](https://github.com/Gracey0201/FinalProject/blob/main/Tables/Forest_core.PNG)

#### Major Roads Table
![Major Roads Table](https://github.com/Gracey0201/FinalProject/blob/main/Tables/major_roads.PNG)

#### Roads Table
![Roads Table](https://github.com/Gracey0201/FinalProject/blob/main/Roads.PNG)

#### Roads Table 2
![Roads2 Table](https://github.com/Gracey0201/FinalProject/blob/main/Roads2.PNG)

#### Building Table
![Building Table](https://github.com/Gracey0201/FinalProject/blob/main/Cleaned%20Table/Building.PNG)

#### Community Health Center Table
![Community Health Center Table](https://github.com/Gracey0201/FinalProject/blob/main/Cleaned%20Table/Community_health_center.PNG)

#### Community Health Center Table 2
![Community Health Center2 Table](https://github.com/Gracey0201/FinalProject/blob/main/Cleaned%20Table/Community_health_center2.PNG)

#### Rare Species Core Table
![Rare Species Core Table](https://github.com/Gracey0201/FinalProject/blob/main/Cleaned%20Table/Rarespecies.PNG)

---

## Spatial Query Optimization

- **Creating Spatial Indexes**: To improve query performance on vector and raster datasets:

```sql
CREATE INDEX IF NOT EXISTS idx_coastalzone_geom ON coastalzone USING GIST (geom);
CREATE INDEX IF NOT EXISTS idx_elevation_rast ON elevation USING GIST (ST_ConvexHull(rast));
```

-Extracting Coastal Elevation Data: Attempted to create a table for only the coastal sections of the elevation raster:

```sql
CREATE TABLE elevation_coastal AS 
SELECT e.rast
FROM elevation e
WHERE EXISTS (
    SELECT 1 FROM coastalzone c
    WHERE ST_Intersects(e.rast, c.geom)
);
```
> **Note:** The query executed successfully but returned no output. Consequently, the initial objective of creating a 1-meter sea-level rise inundation map could not be achieved. These data limitations constrained the analysis of areas susceptible to inundation along the Massachusetts coast.


## Coastal Zone Analysis: Understanding Infrastructure and Habitat Dynamics in Massachusetts
_Infrastructure and Habitat Intersections_

-Identified buildings, roads, cropland, and healthcare facilities within the coastal zone.
-Examined the overlap of cropland with sensitive ecological cores.
-Assessed healthcare accessibility and proximity to transportation networks.

Example Queries

The query selects infrastructure features (buildings, cropland, community health centers, roads) that fall within the Massachusetts coastal zone boundary, thereby identifying the extent of human activities and infrastructure development within the coastal zone.

 ```SQL
SELECT 
    b.* -- columns from building data
FROM 
    building_clean_vector b,
    coastalzone_vector cz
WHERE 
    ST_Intersects(b.geom, cz.geom);
```
	
 ```SQL
SELECT	
	c.*, -- columns from cropland data
    h.* -- columns from health centers data
FROM 
    cropland_vector c,
    community_health_clean_vector h,
	coastalzone_vector cz
WHERE 
    ST_Intersects(c.geom, cz.geom)
    AND ST_Intersects(h.geom, cz.geom);
```

 ```SQL    
SELECT
   r.*,  -- columns from roads data
   m.*  -- columns from major roads
FROM 
    roads_vector r,
	majorroads_vector m,
    coastalzone_vector cz
WHERE 
    ST_Intersects(r.geom, cz.geom)
    AND ST_Intersects(m.geom, cz.geom);
```


-- Incorporating landcover layer into the analysis

_Overlay analysis_

This query is essential for understanding how different land-use/land-cover types are distributed relative to infrastructure features, and for providing insights into patterns of development, potential conflicts, and opportunities for land-use planning and management within the coastal zone.

 ```SQL
CREATE TABLE landcover_near_building AS
SELECT l.*, -- columns from land use/land cover data
       b.*  -- columns from building data
FROM 
    lulc l,
    building_clean_vector b,
    coastalzone_vector cz
WHERE 
    ST_Intersects(l.rast, b.geom)
    AND ST_Intersects(l.rast, cz.geom);
```

 ```SQL
CREATE TABLE coastal_zone_land_use AS
SELECT c.gid AS coastal_zone_id,
       c.geom AS coastal_zone_geom,
       l.rid
FROM coastalzone_vector c
JOIN lulc l ON ST_Intersects(c.geom, l.rast);
```

_Buffer Analysis_

This query creates a buffer around community health centers to assess accessibility and coverage within the coastal zone. It can can identify areas that are underserved or poorly served by healthcare facilities, informing decisions related to healthcare resource allocation and infrastructure development.

 ```SQL
CREATE TABLE community_health_centers_buffer AS
SELECT 
    h.*, -- columns from health centers data
    ST_Buffer(h.geom, 1000) AS buffer_geom -- buffer radius is 1000 meters
FROM 
    community_health_clean_vector h,
    coastalzone_vector cz
WHERE 
    ST_Intersects(h.geom, cz.geom);
```
	
_Distance Analysis_

This query calculates distances between community health centers and roads to understand accessibility within the coastal zone. It helps assess the proximity of healthcare facilities to transportation networks, which is crucial for ensuring accessibility for coastal communities. Also, it can highlight areas with limited access to healthcare services, guiding decisions on infrastructure development and service provision.

 ```SQL
CREATE TABLE chcs_in_roads AS 
SELECT 
    h.gid AS community_health_center_id, 
    r.gid AS road_id, 
    ST_Distance(h.geom, r.geom) AS distance
FROM 
    community_health_clean_vector h,
    roads_vector r,
    coastalzone_vector cz
WHERE 
    ST_Intersects(h.geom, cz.geom)
    AND ST_DWithin(h.geom, r.geom, 5000);-- considering roads within 5 km of health centers
```

_Identify Cropland within Aquatic Core Areas_

This query identifies the encroachment of cropland into aquatic and rare species cores, which can fragment these natural habitats, disrupt ecosystems, and reduce biodiversity. They can also affect water quantity and quality by contributing to pollution from fertilizers, pesticides, and herbicides, thereby degrading aquatic habitats. Further, irrigation from cropland can reduce water resources, decreasing water availability for marine species and other ecosystems.

 ```SQL
CREATE TABLE cropland_in_aquatic_core AS
SELECT c.gid AS cropland_id,
       c.geom AS cropland_geom,
       a.gid AS aquatic_core_id,
       a.geom AS aquatic_core_geom
FROM cropland_vector c
JOIN aquaticcore_clean_vector a ON ST_Intersects(c.geom, a.geom);
```

 ```SQL
CREATE TABLE cropland_in_rare_species_core AS
SELECT c.gid AS cropland_id,
       c.geom AS cropland_geom,
       ra.gid AS rare_species_core_id,
       ra.geom AS rare_species_core_geom
FROM cropland_vector c
JOIN rarespecies_clean_vector ra ON ST_Intersects(c.geom, ra.geom);
```

_Overlay Analysis_

The two queries below, which create new tables, contain major and minor roads and their corresponding coastal zones, providing spatial information on transportation infrastructure in coastal areas. This can be useful for various analyses, such as transportation planning, disaster response, and environmental impact assessments.

 ```SQL
CREATE TABLE majorroads_in_coastal AS
SELECT m.gid AS roads_id,
       m.geom AS roads_geom,
       c.gid AS coastal_zone_id,
       c.geom AS coastal_zone_geom
FROM majorroads_vector m
JOIN coastalzone_vector c ON ST_Intersects(m.geom, c.geom);
```

 ```SQL
CREATE TABLE roads_in_coastal AS
SELECT r.gid AS roads_id,
       r.geom AS roads_geom,
       c.gid AS coastal_zone_id,
       c.geom AS coastal_zone_geom
FROM roads_vector r
JOIN coastalzone_vector c ON ST_Intersects(r.geom, c.geom);
```

## Result Map
_Massachusetts Coastal Map_

![MA Coastal Map3](https://github.com/Gracey0201/FinalProject/blob/main/Updated_AnalysisMap.png)

## Challenges
-Elevation raster size prevented creation of a 1-meter coastal inundation layer.

-Some spatial joins returned empty results due to raster–vector complexity.

-Limited elevation data prevented the creation of flood inundation models and a complete analysis.

## Conclusion
This project demonstrates how PostGIS-based spatial databases can be used to analyze:

-Coastal infrastructure exposure

-Ecological habitat vulnerability

-Healthcare accessibility and land use patterns

While inundation modeling was constrained, the analysis provides actionable insights for coastal planning, conservation, and resilience strategies. The workflow is transferable to other regions with improved data.

## Future Work

-Integrate LiDAR-derived DEMs for inundation modeling

-Cloud-based PostGIS workflows

-Time-series land cover change analysis

-Scenario-based coastal risk assessment

#### Useful Resources

- [NOAA Sea Level Rise Viewer](https://coast.noaa.gov/slr/)
- [MassGIS Data Portal](https://www.mass.gov/orgs/massgis-data)
- [PostGIS Documentation](https://postgis.net/documentation/)

---

## Citation

If you use this repository in your work, please cite it as:

**Grace Nwachukwu. (2026). PostGIS-Based Massachusetts Coastal Resilience Assessment [Data set]. Zenodo. https://doi.org/10.5281/zenodo.18148415**



