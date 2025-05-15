<h1>Mapa mapa mapa</h1>
<p>alguma coisa mapa</p>

<div id="map">
    <svg>
        {#each stations as station}
            <circle {...getCoords(station)} r="5" fill="steelblue" />
        {/each}
    </svg>
</div>

<style>
    @import url("$lib/global.css");
</style>

<script>
    import mapboxgl from "mapbox-gl";
    import "../../node_modules/mapbox-gl/dist/mapbox-gl.css";
    import * as d3 from "d3";
    import { onMount } from "svelte";
    
    mapboxgl.accessToken = "pk.eyJ1IjoiZmFsY2FvYW5kZXJzb24iLCJhIjoiY21hcGhlMGl0MGo0MjJqb25hN2dzMjJxeSJ9.5IfozuoL3_WePp7audzpkw";


    let map;
    let stations = [];

    function getCoords (station) {
        let point = new mapboxgl.LngLat(+station.Long, +station.Lat);
        let {x, y} = map.project(point);
        return {cx: x, cy: y};
    }

    async function initMap() {
        map = new mapboxgl.Map({
            container: 'map',
            center: [-71.09415, 42.36027],
            zoom: 12,
            style: "mapbox://styles/mapbox/streets-v12",
        });
        
        await new Promise(resolve => map.on("load", resolve));

        map.addSource("boston_route", {
            type: "geojson",
            data: "https://bostonopendata-boston.opendata.arcgis.com/datasets/boston::existing-bike-network-2022.geojson?outSR=%7B%22latestWkid%22%3A3857%2C%22wkid%22%3A102100%7D",
        });
        
        map.addLayer({
            id: "boston_route_layer", 
            type: "line",
            source: "boston_route", // The id we specified in `addSource()`
            paint: {
                // paint params, e.g. colors, thickness, etc.
                "line-color": "#888",
                "line-width": 2,
                "line-opacity": 0.8,
            },
        });
        
        map.on('move', () => {
            stations = [...stations];
        });
        map.on('resize', () => {
            stations = [...stations];
        });
    }

    async function loadStationData() {
        try {
            const csvUrl = 'https://vis-society.github.io/labs/8/data/bluebikes-stations.csv';
            const data = await d3.csv(csvUrl);
            
            stations = data.map(station => ({
                id: station.Number,
                name: station.NAME,
                Lat: +station.Lat,
                Long: +station.Long,
            }));
            // console.log('Stations loaded:', stations.length);
        } catch (error) {
            console.error('Error loading station data:', error);
        }
    }

    onMount(() => {
        initMap();
        loadStationData();
    });    
</script>