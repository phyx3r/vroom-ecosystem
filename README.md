## Open Source Routing Machine Ecosystem

High performance routing engine written in C++14 designed to run on OpenStreetMap data.

The following services are available via HTTP API, C++ library interface and NodeJs wrapper:
- Nearest - Snaps coordinates to the street network and returns the nearest matches
- Route - Finds the fastest route between coordinates
- Table - Computes the duration or distances of the fastest route between all pairs of supplied coordinates
- Match - Snaps noisy GPS traces to the road network in the most plausible way
- Trip - Solves the Traveling Salesman Problem using a greedy heuristic
- Tile - Generates Mapbox Vector Tiles with internal routing metadata

For a quick introduction about how the road network is represented in OpenStreetMap and how to map specific road network features have a look at [this guide about mapping for navigation](https://www.mapbox.com/mapping/mapping-for-navigation/).

Related [Project-OSRM](https://github.com/Project-OSRM) repositories:
- [osrm-frontend](https://github.com/Project-OSRM/osrm-frontend) - User-facing frontend with map. The demo server runs this on top of the backend
- [osrm-text-instructions](https://github.com/Project-OSRM/osrm-text-instructions) - Text instructions from OSRM route response
- [osrm-backend-docker](https://hub.docker.com/r/osrm/osrm-backend/) - Ready to use Docker images

## Documentation

## Quick Start

The easiest and quickest way to setup your own routing engine is to use Docker images we provide.

There are two pre-processing pipelines available:
- Contraction Hierarchies (CH)
- Multi-Level Dijkstra (MLD)

we recommend using MLD by default except for special use-cases such as very large distance matrices where CH is still a better fit for the time being.
In the following we explain the MLD pipeline.
If you want to use the CH pipeline instead replace `osrm-partition` and `osrm-customize` with a single `osrm-contract` and change the algorithm option for `osrm-routed` to `--algorithm ch`.

### Using Docker

We base our Docker images ([backend](https://hub.docker.com/r/osrm/osrm-backend/), [frontend](https://hub.docker.com/r/osrm/osrm-frontend/)) on Debian and make sure they are as lightweight as possible.

Download OpenStreetMap extracts for example from [Geofabrik](http://download.geofabrik.de/)

    wget http://download.geofabrik.de/europe/germany/berlin-latest.osm.pbf

Pre-process the extract with the car profile and start a routing engine HTTP server on port 5000

    docker run -t -v "${PWD}:/data" osrm/osrm-backend osrm-extract -p /opt/car.lua /data/berlin-latest.osm.pbf

The flag `-v "${PWD}:/data"` creates the directory `/data` inside the docker container and makes the current working directory `"${PWD}"` available there. The file `/data/berlin-latest.osm.pbf` inside the container is referring to `"${PWD}/berlin-latest.osm.pbf"` on the host.

    docker run -t -v "${PWD}:/data" osrm/osrm-backend osrm-partition /data/berlin-latest.osrm
    docker run -t -v "${PWD}:/data" osrm/osrm-backend osrm-customize /data/berlin-latest.osrm

Note that `berlin-latest.osrm` has a different file extension. 

    docker run -t -i -p 5000:5000 -v "${PWD}:/data" osrm/osrm-backend osrm-routed --algorithm mld /data/berlin-latest.osrm

## Running Ecosystem

    docker-compose up