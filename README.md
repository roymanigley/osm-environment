# OSM Environment

## build the latest image localy

    docker build -t osrm/osrm-backend:LOCAL -f docker/Dockerfile .

## initialisation

    wget https://download.geofabrik.de/europe/switzerland-latest.osm.pbf -O osm-data/switzerland-latest.osm.pbf

    docker compose --profile init up --remove-orphans --build
    docker compose --profile partition up --remove-orphans --build
    docker compose --profile customize up --remove-orphans --build

## run the service
> for the first time it will take much time since will load all the lications in the database for the `nominatim-api`

    docker compose --profile prod up --remove-orphans --build
