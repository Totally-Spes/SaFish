<script>
	import { onDestroy, setContext} from 'svelte';
	import { mapbox, key } from './mapbox.js';
    import * as MaplibreGrid from 'maplibre-grid';
    import MapMarker from './MapMarker.svelte';

	setContext(key, {
		getMap: () => map,
	});

	export let lat;
	export let lon;
	export let zoom;

	let container;
	let map;
    let grid;
    let path = [];
    let data;

	function load() {
		map = new mapbox.Map({
			container,
			style: 'mapbox://styles/mapbox/streets-v9',
			center: [lon, lat],
			zoom,
		});
        grid = new MaplibreGrid.Grid({
            gridWidth: 10,
            gridHeight: 10,
            units: 'kilometers',
            minZoom: 7,
            maxZoom: 12,
            paint: {
                'line-opacity': .5
                }
        });
        map.addControl(grid);
	}

    function seaLayer(){
            map.addLayer({
            'id': 'openseamap',
            'type': 'raster',
            'source': {
            'type': "raster",
            'tiles': ['https://tiles.openseamap.org/seamark/{z}/{x}/{y}.png'],
            'tileSize': 256
            },
            "minzoom": 1,
            "maxzoom": 22,
            "paint": {
            "raster-opacity": 1
            }
        }, 'waterway-label')
    }

    function drawLine(){
        map.addSource('path', { type: 'geojson', data: data });
        map.addLayer({
            "id": "road",
            "type": "line",
            "source": 'path',
            "layout": {
                "line-join": "round",
                "line-cap": "round"
            },
            "paint": {
                "line-color": "#888",
                "line-width": 8
            }
        });
    }

	onDestroy(() => {
        map.removeControl(grid);
		if (map) map.remove();
	});
</script>

<svelte:head>
	<link
		rel="stylesheet"
		href="https://unpkg.com/mapbox-gl/dist/mapbox-gl.css"
		on:load={load}
	/>
</svelte:head>

<div bind:this={container}>
	{#if map}
		<slot />
	{/if}
</div>

{#if map}
    {map.on('load', () => {
        seaLayer();
        data = { 'type': 'Feature',
            'properties': {},
            'geometry': {
            'type': 'LineString',
            'coordinates': path
            }
        }
        map.addSource('path', { type: 'geojson', data: data });
        map.addLayer({
            "id": "road",
            "type": "line",
            "source": 'path',
            "layout": {
                "line-join": "round",
                "line-cap": "round"
            },
            "paint": {
                "line-color": "#F00",
                "line-width": 8
            }
        });
    })}
    {map.on('click', function(e) {
        var coordinates = e.lngLat;
        [lon,lat] = [coordinates.lng,coordinates.lat];
        path.push([lon,lat])
        if(path.length > 1)
        {
            map.getSource('path').setData(data);
        }
        else
        {
            new MapMarker(lon,lat,"Start");
        }
        console.log(path);
      })}
{/if}

<style>
	div {
		width: 100%;
		height: 100%;
	}
</style>
