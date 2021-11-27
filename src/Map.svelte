<script>
    import { onDestroy, setContext} from 'svelte';
    import { mapbox, key } from './mapbox.js';
    import * as MaplibreGrid from 'maplibre-grid';
    import { getGrid, getGridCell } from 'maplibre-grid/src/calc';
    import MapMarker from './MapMarker.svelte';
    import { onMount } from "svelte";

	setContext(key, {
		getMap: () => map,
	});

	export let lat;
	export let lon;
	export let zoom;

    // const express = require("express");
	// const bodyParser = require("body-parser");
	// const cors = require('cors')

	// const app = express();
	// app.use(bodyParser.json());
	// app.use(cors())

	let container;
	let map;
    let grid;
    let path = [];
    let data;
    let selectedCells = [];
    let selectedCellsId = 'selected-cells';

	function load() { 
		map = new mapbox.Map({
			container,
			style: 'mapbox://styles/batlle/ckwhnl75t1g5514n46mzemfnz',
			center: [lon, lat],
			zoom,
		});
        // Add zoom and rotation controls to the map.
        // map.addControl(new mapboxgl.NavigationControl());
        grid = new MaplibreGrid.Grid({
            gridWidth: 10,
            gridHeight: 10,
            units: 'kilometers',
            minZoom: 7,
            maxZoom: 12,
            paint: {    
                'line-opacity': .5,
                "line-color": "#FFF"
                }
        });
        // map.moveLayer('')
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
        })
    }

    function initLine(){
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
    }

	onDestroy(() => {
        map.removeControl(grid);
		if (map) map.remove();
	});

    function selectBox(bbox){
        const cellIndex = selectedCells.findIndex(x => x.geometry.bbox.toString() === bbox.toString());
        if (cellIndex === -1) {
            const coordinates = [
            [
                [bbox[0], bbox[1]],
                [bbox[2], bbox[1]],
                [bbox[2], bbox[3]],
                [bbox[0], bbox[3]],
                [bbox[0], bbox[1]],
            ]
            ];
            const cell = { type: 'Feature', geometry: { type: 'Polygon', bbox, coordinates }};
            selectedCells.push(cell);
        }
        const source = map.getSource(selectedCellsId);
        source.setData({ type: 'FeatureCollection', features: selectedCells });
    }


</script>


<svelte:head>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/5.12.0-2/css/all.min.css">
	<link
		rel="stylesheet"
		href="https://unpkg.com/mapbox-gl/dist/mapbox-gl.css"
		on:load={load}
	/>

</svelte:head>

<!-- MAP CONTAINER -->
<div bind:this={container} style="width:auto; margin-left: auto; margin-right: 0;">
	{#if map}
		<slot />
	{/if}
</div>

{#if map}
    {map.on('load', () => {
        seaLayer();
        initLine();
        map.addSource(selectedCellsId, {
            type: 'geojson',
            data: { type: 'FeatureCollection', features: selectedCells }
        });
        map.addLayer({
            id: selectedCellsId,
            source: selectedCellsId,
            type: 'fill',
            paint: {
                'fill-color': '#00f',
                'fill-opacity': 0.2,
                'fill-outline-color': 'transparent'
            }
        });
        // fetch("http://127.0.0.1:5001/api/location/getbox", 
        //         {
        //         mode: 'no-cors',
        //         headers: {
        //             'Access-Control-Allow-Origin':'*'
        //         }
        //         })
        //         .then(x => {
        //             console.log("response:");
        //             console.log(x);
        //             // selectBox([res[0][2],res[0][3],res[0][4],res[0][5]]);
        //             // return x.json();
        //         });
    })}

    {map.on(MaplibreGrid.GRID_CLICK_EVENT, event =>{
        console.log(event.bbox);
        var [lon1,lat1,lon2,lat2] = event.bbox;
        var lon = (lon1 + lon2)/2;
        var lat = (lat1 + lat2)/2;
        console.log([lon,lat]);
        path.push([lon,lat]);

        selectBox(event.bbox);


        if (path.length > 1)
        {
            let [prev_lon, prev_lat] = path[path.length-2];

            let [iter_lon, iter_lat] = [prev_lon, prev_lat];
            let lon_step = prev_lon < lon ? 0.01 : -0.01;
            let lat_step = prev_lat < lat ? 0.01 : -0.01;

            let latlondist = (lat1, lon1, lat2, lon2) =>
            {
                let diff_lat = lat1 - lat2
                let diff_lon = lon1 - lon2
                return Math.sqrt(diff_lat * diff_lat + diff_lon * diff_lon)
            };

            let distance = latlondist(lat, lon, prev_lat, prev_lon) * 20;
            lon_step = (lon - prev_lon) / distance;
            lat_step = (lat - prev_lat) / distance;

            // console.log("source:");
            // console.log([prev_lon, prev_lat]);
            // console.log("destination:");
            // console.log([lon, lat]);
            // console.log("distance:");
            // console.log(distance);
            // console.log("step:");
            // console.log([lon_step, lat_step]);
            console.log("TRACE PATH START");

            for (let i = 0; i < distance; i++)
            {
                iter_lon += lon_step;
                iter_lat += lat_step;
                console.log([iter_lon, iter_lat]);

                const bbox = getGridCell([iter_lon, iter_lat], 10, 10, 'kilometers');
                // todo: set amount
                let amount = 99;
                const res = fetch(`http://127.0.0.1:5001/api/location/setbox/${bbox[0]}/${bbox[1]}/${bbox[2]}/${bbox[3]}/${amount}`, {
                mode: 'no-cors',
                headers: {
                    'Access-Control-Allow-Origin':'*'
                }
                })
                selectBox(bbox);

                console.log(bbox)
            }

            console.log("TRACE PATH END")
        }

        if(path.length > 1)
        {
            map.getSource('path').setData(data);
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
