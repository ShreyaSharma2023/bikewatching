<script>
import * as d3 from "d3";
import mapboxgl from "mapbox-gl";
import "../../node_modules/mapbox-gl/dist/mapbox-gl.css";
mapboxgl.accessToken = "pk.eyJ1Ijoic2hyZXlhc2hhcm1hMjAyNSIsImEiOiJjbTkxcGZraTQwM2owMmpwcXBwZ3U1Z20wIn0.GxhB5GSOaHAUUZ9LK8soOw";
import { onMount } from "svelte";
let map;
let mapViewChanged = 0;

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
    	map.addSource("cambridge_route", {
		type: "geojson",
		data: "https://raw.githubusercontent.com/cambridgegis/cambridgegis_data/main/Recreation/Bike_Facilities/RECREATION_BikeFacilities.geojson",
	});
    map.addLayer({
	id: "Boston_ID", // A name for our layer (up to you)
	type: "line", // one of the supported layer types, e.g. line, circle, etc.
	source: "boston_route", // The id we specified in `addSource()`
    paint: {
        "line-color": "#008000", // Green color (Hex code format)
        "line-width": 3, // Width of the line
        "line-opacity": 0.4 // Opacity (0 = invisible, 1 = fully visible)
    }
});
    map.addLayer({
	id: "Cambridge_ID", // A name for our layer (up to you)
	type: "line", // one of the supported layer types, e.g. line, circle, etc.
	source: "cambridge_route", // The id we specified in `addSource()`
    paint: {
        "line-color": "#008000", // Green color (Hex code format)
        "line-width": 3, // Width of the line
        "line-opacity": 0.4 // Opacity (0 = invisible, 1 = fully visible)
    }
});
}
$: map?.on("move", evt => mapViewChanged++);

onMount(() => {
	    initMap();
    });
let stations = [];
onMount(async () => {
    const data = await d3.csv("https://vis-society.github.io/labs/8/data/bluebikes-stations.csv");
    stations = [...data]; // Spread to trigger reactivity
});
</script>

<h1>Bikewatching</h1>
<p>This will be an interactive page about bikes</p>
<div id="map">
	<svg>
    {#key mapViewChanged}
        {#each stations as station}
	        <circle { ...getCoords(station) } r="5" fill="steelblue" />
        {/each}
    {/key}
    </svg>
</div>


<style>
@import url("$lib/global.css");
</style>
