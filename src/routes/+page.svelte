<script>
import mapboxgl from "mapbox-gl";
import "../../node_modules/mapbox-gl/dist/mapbox-gl.css";
mapboxgl.accessToken = "pk.eyJ1Ijoic2hyZXlhc2hhcm1hMjAyNSIsImEiOiJjbTkxcGZraTQwM2owMmpwcXBwZ3U1Z20wIn0.GxhB5GSOaHAUUZ9LK8soOw";
import { onMount } from "svelte";

onMount(() => {
        let map = new mapboxgl.Map({
            container: "map", // ID of the div element for the map
            style: "mapbox://styles/mapbox/streets-v12", // Map style
            center: [-71.0589, 42.3601], // Coordinates for Cambridge/Boston (longitude, latitude)
            zoom: 12, // Initial zoom level
        });

        // Optional: Add zoom & rotation controls
        map.addControl(new mapboxgl.NavigationControl());
    });
</script>

<h1>Bikewatching</h1>
<p>This will be an interactive page about bikes</p>
<div id="map" />


<style>
@import url("$lib/global.css");
</style>
