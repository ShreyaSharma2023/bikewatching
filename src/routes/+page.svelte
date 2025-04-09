<script>
import * as d3 from "d3";
import mapboxgl from "mapbox-gl";
import "../../node_modules/mapbox-gl/dist/mapbox-gl.css";
mapboxgl.accessToken = "pk.eyJ1Ijoic2hyZXlhc2hhcm1hMjAyNSIsImEiOiJjbTkxcGZraTQwM2owMmpwcXBwZ3U1Z20wIn0.GxhB5GSOaHAUUZ9LK8soOw";
import { onMount } from "svelte";
import { scaleSqrt } from "d3-scale";
let map;
let mapViewChanged = 0;
let radiusScale;
let stationFlow = d3.scaleQuantize()
	.domain([0, 1])
	.range([0, 0.5, 1]);

let selectedTime = -1; // Default to "any time"
// Define a radius scale that changes based on time filtering
function getRadius(value) {
    return scaleSqrt()
        .domain([0, 100]) // Adjust domain based on your data range
        .range(selectedTime === -1 ? [0, 25] : [3, 30])(value);
}

function formatTime(minutes) {
    if (minutes === -1) return "";
    let hours = Math.floor(minutes / 60);
    let mins = minutes % 60;
    return `${hours}:${mins.toString().padStart(2, '0')}`;
}

let timeFilter = -1;
$: timeFilterLabel = new Date(0, 0, 0, 0, timeFilter)
                     .toLocaleString("en", {timeStyle: "short"});

function getCoords (station) {
	let point = new mapboxgl.LngLat(+station.Long, +station.Lat);
	let {x, y} = map.project(point);
	return {cx: x, cy: y};
}
function minutesSinceMidnight (date) {
	return date.getHours() * 60 + date.getMinutes();
}
$: filteredTrips = timeFilter === -1? trips : trips.filter(trip => {
	let startedMinutes = minutesSinceMidnight(trip.started_at);
	let endedMinutes = minutesSinceMidnight(trip.ended_at);
	return Math.abs(startedMinutes - timeFilter) <= 60
	       || Math.abs(endedMinutes - timeFilter) <= 60;
});

$: filteredDepartures = d3.rollup(filteredTrips, v => v.length, d => d.start_station_id);
$: filteredArrivals = d3.rollup(filteredTrips, v => v.length, d => d.end_station_id);

$: filteredStations = stations.map(station => {
	const id = station.Number;
	const arr = filteredArrivals.get(id) ?? 0;
	const dep = filteredDepartures.get(id) ?? 0;
	return {
		...station,
		arrivals: arr,
		departures: dep,
		totalTraffic: arr + dep
	};
});

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
$: radiusScale = d3.scaleSqrt()
	.domain([0, d3.max(stations, d => d.totalTraffic) || 0])
	.range([0, 25]);
onMount(() => {
	    initMap();
    });
let stations = [];
let trips = [];
let departures;
let arrivals;

onMount(async () => {
    const data = await d3.csv("https://vis-society.github.io/labs/8/data/bluebikes-stations.csv");
    stations = [...data]; // Spread to trigger reactivity
    const data2 = await d3.csv("https://vis-society.github.io/labs/8/data/bluebikes-traffic-2024-03.csv");
    trips = await d3.csv("https://vis-society.github.io/labs/8/data/bluebikes-traffic-2024-03.csv").then(trips => {
	for (let trip of trips) {
		trip.started_at = new Date(trip.started_at)
		trip.ended_at = new Date(trip.ended_at)
	}
	return trips;
});

    departures = d3.rollup(trips, v => v.length, d => d.start_station_id);
    arrivals = d3.rollup(trips, v => v.length, d => d.end_station_id);
    stations = stations.map(station => {
	let id = station.Number;
	station.arrivals = arrivals.get(id) ?? 0;
	station.departures = departures.get(id) ?? 0;
	station.totalTraffic = station.arrivals + station.departures;
	return station;
});

});


</script>

<h1>Bikewatching</h1>
    <label>
        Filter by time:
        <input type="range" min="-1" max="1440" bind:value={timeFilter} />
        {#if timeFilter !== -1}
            <time style="display: block">
                {timeFilterLabel}
            </time>
        {:else}
            <em style="display: block">(any time)</em>
        {/if}
    </label>
<div id="map">
	<svg>
    {#key mapViewChanged}
        {#each filteredStations as station}
	        <circle { ...getCoords(station) } 
            r={radiusScale(station.totalTraffic)} 
            style="--departure-ratio: { stationFlow(station.departures / station.totalTraffic) }"
/>
        {/each}
    {/key}
    </svg>
</div>
<div class="legend">
	<div style="--departure-ratio: 1">More departures</div>
	<div style="--departure-ratio: 0.5">Balanced</div>
	<div style="--departure-ratio: 0">More arrivals</div>
</div>


<style>
@import url("$lib/global.css");
</style>
