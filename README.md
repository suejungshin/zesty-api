# Zesty.ai Backend Project

## Summary
Returns basic information about a given property and its neighbors

## How to Run
`docker-compose -p project up -d` ("-p project" so container name is predictable for steps below, "-d" for detatched mode)


## How To Test
`docker exec -it project_zesty_app_1 pytest`<br>
(or grab CONTAINER ID for the zesty_app at port 8080 printed from `docker ps -a` and subsitute, e.g., `docker exec -it 9c270f7860ac pytest`)

Or, can go to links at localhost below to check it out.<br>
Here are the valid propertyIDs for the 5 properties in the test database, for reference:
- f1650f2a99824f349643ad234abff6a2
- f853874999424ad2a5b6f37af6b56610
- 3290ec7dd190478aab124f6f2f32bdd7
- 5e25c841f0ca47ac8215b5fd0076259a
- 622088210a6f43fca2a1824e8610df03


## API endpoints

Endpoints are exposed at http://localhost:8080/{endpoint}

  ### GET /api/display/:id(?overlay=True)

  Returns the image of the property by ID

  Path parameter - property ID (e.g., f853874999424ad2a5b6f37af6b56610)<br>
  Query parameter (optional) - overlay=True or overlay=False (case insensitive)

  Examples -
  http://localhost:8080/api/display/f853874999424ad2a5b6f37af6b56610
  http://localhost:8080/api/display/f853874999424ad2a5b6f37af6b56610?overlay=true
  http://localhost:8080/api/display/f853874999424ad2a5b6f37af6b56610?overlay=false

  Example response:<br>
  JPEG image as displayed in browser<br>
  If overlay is true, image has overlayed building and parcel polygons


  ### POST /api/find

  Returns list of property IDs that are within search distance radius (in METERS) of the target property. Note that distances are based on distance between each property's POINT geocode_code which is at the center of each property.

  POST a geojson object specifying [longitude, latitude] (in DEGREES) of target property, as well as search distance radius in METERS<br>

  Example- http://localhost:8080/api/find  (plus the POST data below)<br>
  POST request data must be in format:
   ```python
  {
    "type": "Feature",
    "geometry": {
        "type": "Point",
        "coordinates": [-80, 26]
    },
    "distance": 1000000
  }
  ```

  Example response:
  ```python
  ['f853874999424ad2a5b6f37af6b56610', '3290ec7dd190478aab124f6f2f32bdd7', '622088210a6f43fca2a1824e8610df03']
  ```

  Note that the PostGIS query seems to break down at distances too large, in which case the API response will be "Search distance too large"

  ### GET /api/statistics

  Returns basic statistics about the target property's neighbors, where neighbors are those properties whose POINT geocode_code are within target distance (in METERS) of the target property's POINT geocode_code:
  - Total area (in SQUARE METERS) of all neighboring parcels, summed
  - List of areas (in SQUARE METERS) of neighboring buildings
  - List of distances (in METERS) between each neighboring building (POLYGON) and target property (POINT geocode_geo). The ST_Distance function used finds the minimum 2D Cartesian distance, so looks to the closest part on the edge of the building polygon.
  - Zone density (in PERCENT, so a result of 100 means 100% and a result of 0.1 means 0.1%) - percent of buffer of target distance around the target property that is occupied by a building

  Path parameter - property ID (e.g., f853874999424ad2a5b6f37af6b56610)<br>
  Query parameter - distance in METERS

  Example -
  http://localhost:8080/api/statistics/622088210a6f43fca2a1824e8610df03?distance=10

  Example response:
```python
  {
    total_parcel_area_in_radius: 2330.55982263293,
    buildings_areas: [
      981.199068874121
    ],
    buildings_dists_to_center: [
      0
    ],
    zone_density: 97.49789007556822
  }
```


## Other Notes to Self

Other Notes to Self, hope you don't mind if I put these here to help my future self!

### To Access Postgres shell from within Docker container:
1. `docker-compose up -d` (if not already done above)
2. `docker exec -it project_postgres_1 psql -U postgres`
or `docker exec -it 9c270f7860ac psql -U postgres` where the alphanumeric string should be replaced by the container ID from running `docker ps -a`
(If developing locally, need to `docker stop project_zesty_app_1` the dockerized zesty_app service and serve it at local host and keep the dockerized postgres db running)

##### Basic Postgres commands:
`\l`  -- Show databases <br>
`\c databaseName`  -- Connect to database of choice<br>
`\dt` -- show tables<br>
`\x` -- expanded view on to show rows in the case the column view wrapping poorly<br>
`SELECT * FROM properties;`<br>
Can run the ST_ commands in psql - make sure to use single quotes, not double quotes<br>
`SELECT ST_AsEWKT('0101000020E6100000A79608AFB80454C08CEABEAD05633A40');`<br>

### Docker
To build image:<br>
`docker build -t zesty .`<br>
`docker images` Get image id<br>
`docker tag INSERT_IMAGE_ID_HERE suejungshin/zesty-app:latest`<br>
`docker push suejungshin/zesty-app:latest`<br>


### Other notes to self
When developing locally:<br>
`python3 -m venv venv`<br>
`source venv/bin/activate`<br>
to get out `deactivate`<br>

urllib.request certificate error on MacOS (one time fix)<br>
Go to Macintosh HD > Applications > Python3.6 folder (or other version) > double click on "Install Certificates.command" file

To enter docker container once it's running:<br>
docker exec -it af6cad9cf311 bash<br>
ctrl + d to exit