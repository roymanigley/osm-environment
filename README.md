# OSM Environment

## build the latest image localy
> open `osrm-backend/src/engine/routing_algorithms/alternative_path_mld.cpp` and adapt the `struct Parameters` to customize the routing

    git clone git@github.com:Project-OSRM/osrm-backend.git
    cd ./osrm-backend
    docker build -t osrm/osrm-backend:LOCAL -f docker/Dockerfile .
    cd ..

## initialisation

    wget https://download.geofabrik.de/europe/switzerland-latest.osm.pbf -O osm-data/switzerland-latest.osm.pbf

    docker compose --profile init up --remove-orphans --build
    docker compose --profile partition up --remove-orphans --build
    docker compose --profile customize up --remove-orphans --build

## run the service
> for the first time it will take much time since will load all the lications in the database for the `nominatim-api`

    docker compose --profile prod up --remove-orphans --build

## Examples

### Get coordiates from address

    def address_to_lon_lat_list(address: str) -> list[str]:
        response = requests.get(
            url=f'http://nominatim-api:8080/search?q={address}&addressdetails=1'
        )
        return [f'{result["lon"]},{result["lat"]}' for result in response.json()]

### map adresses to waypoints

    waypoints = []
    addresses = [
        #'Muri bei Bern',
        # 'Chutzenstrasse 20, 3007 Bern',
        #'Belpstrasse 23, 3007 Bern',
        'Scheibenstrasse, 3014 Bern',
        'Reichenbachstrasse 101, 3004 Bern',
    ]
    for address in addresses:
        results = address_to_lon_lat_list(address)
        waypoints.append({
            'address': address,
            'location': results[0],
        })

### get routes by waypoints

    path = ';'.join([w['location'] for w in waypoints])
    approaches = ';'.join(['curb' for _ in waypoints])
    response = requests.get(
        url=f'http://osrm-api:5000/route/v1/driving/{path}?steps=true&alternatives=3&overview=false&continue_straight=false&continue_straight=true&approaches={approaches}'
    )
    print(response.status_code)
    print('routes found:', len(response.json()['routes']))

### render a map with path and marker

    import folium
    from IPython.display import display
    
    m = folium.Map(location=[waypoints[0]['location'].split(',')[1], waypoints[0]['location'].split(',')[0]], zoom_start=12)
    routes = []
    for route in response.json()['routes']:
        locations = []
        for leg in route['legs']:
            for step in leg['steps']:
                for intersection in step['intersections']:
                    locations.append(intersection['location'])
        routes.append({
            'distance': round(route['distance'] / 1000, 3),
            'duration': round(route['duration'] / 60, 2),
            'path': [(location[1], location[0]) for location in locations],
        })

    colors = ['green', 'blue', 'yellow', 'red']
    for i in range(len(routes)):
        folium.PolyLine(
            routes[i]['path'], 
            color=colors[i], 
            weight=5, 
            opacity=1, 
            popup=f'{routes[i]["duration"]}&nbsp;min<br>{routes[i]["distance"]}&nbsp;km'
        ).add_to(m)

    for i in range(len(waypoints)):
        title = ''
        color = ''
        if i == 0:
            title = '<strong>START</strong><br>'
            color = 'green'
        elif i == len(waypoints) -1:
            title = '<strong>END</strong><br>'
            color = 'blue'
        else:
            title = f'<strong>PICKUP {i}</strong><br>'
            color = 'gray'
        folium.Marker(
            location=[waypoints[i]['location'].split(',')[1], 
            waypoints[i]['location'].split(',')[0]], 
            popup=f'{title}{waypoints[i]["address"]}', 
            icon=folium.Icon(color=color),
            
        ).add_to(m)

    m.save("route_map.html")
    display(m)
