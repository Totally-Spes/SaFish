<script>
    import { onDestroy, setContext} from 'svelte';
    import { mapbox, key } from './mapbox.js';
    import * as mapboxgl from 'mapbox-gl';
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
	export let drawMode;

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
    let level = [[],[],[],[]];
    let levelId = ['level-one','level-two','level-three','level-four'];
    
    let fishAmount;

    


	function load() { 
		map = new mapbox.Map({
			container,
			style: 'mapbox://styles/batlle/ckwhnl75t1g5514n46mzemfnz',
			center: [lon, lat],
			zoom,
		});
        // Add zoom and rotation controls to the map.
        map.addControl(new mapboxgl.NavigationControl());
        // add control scale to map
        map.addControl(new mapboxgl.ScaleControl({
            maxWidth: 100,
            unit: 'metric'
        }));
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
        map.addControl(grid);

        // add choropleth legend to map
        


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

    async function queryOnWater(lat, lon)
    {
        // TODO: uncomment
        // let token = 'KdjuCzo_PW56GE5tsr5e';
        // let resp = await fetch(`https://api.onwater.io/api/v1/results/${lat},${lon}?access_token=${token}`);
        // return await resp.json();

        return {water: true};
    }

	onDestroy(() => {
        map.removeControl(grid);
		if (map) map.remove();
	});


    function selectBox(bbox,amount,oldAmount, isInit = false){
        let i = amount <= 20 ? 0 : amount <= 40 ? 1 :  amount <= 60 ? 2 : 3 ;
        let j = oldAmount <= 20 ? 0 : oldAmount <= 40 ? 1 :  oldAmount <= 60 ? 2 : 3 ;

        console.log("selectBox: i=" + i + "; j=" + j);
        if (oldAmount == 0 || isInit)
        {
            let coordinates = [
                [
                    [bbox[0], bbox[1]],
                    [bbox[2], bbox[1]],
                    [bbox[2], bbox[3]],
                    [bbox[0], bbox[3]],
                    [bbox[0], bbox[1]],
                ]
            ];
            console.log("creating cell");
            let cell = { type: 'Feature', geometry: { type: 'Polygon', bbox, coordinates }};
            level[i].push(cell);
        }
        else if (i == j)
        {
            console.log("same level");
            return;
        }
        else
        {
            let cellIndex = level[j].findIndex(x =>
            {
                console.log(x);
                return x.geometry.bbox.toString() === bbox.toString();
            } );
            console.log("removing cell in level j, adding cell in level i")
            let cell = level[j][cellIndex];
            level[i].push(cell);
            level[j].splice(cellIndex, 1);
            // let cell = level[j].splice(cellIndex, 1);
        }

        let source1 = map.getSource(levelId[j]);
        source1.setData({ type: 'FeatureCollection', features: level[j] });

        let source2 = map.getSource(levelId[i]);
        source2.setData({ type: 'FeatureCollection', features: level[i] });
    }

    function selectBoxOld(bbox,amount,oldAmount){
        let i = amount < 20 ? 0 : amount < 40 ? 1 :  amount < 60 ? 2 : 3 ;
        let j = oldAmount < 20 ? 0 : oldAmount < 40 ? 1 :  oldAmount < 60 ? 2 : 3 ;

        let cellIndex = level[j].findIndex(x => x.geometry.bbox.toString() === bbox.toString());
        if (cellIndex === -1) {
            cellIndex = level[i].findIndex(x => x.geometry.bbox.toString() === bbox.toString());
            if (cellIndex === -1) {
                let coordinates = [
                [
                    [bbox[0], bbox[1]],
                    [bbox[2], bbox[1]],
                    [bbox[2], bbox[3]],
                    [bbox[0], bbox[3]],
                    [bbox[0], bbox[1]],
                ]
                ];
                let cell = { type: 'Feature', geometry: { type: 'Polygon', bbox, coordinates }};
                level[i].push(cell);
            }
        }
        else{
            let cell = level[j].splice(cellIndex, 1);
            level[i].push(cell);
        }
        let source1 = map.getSource(levelId[j]);
        source1.setData({ type: 'FeatureCollection', features: level[j] });
        
        let source2 = map.getSource(levelId[i]);
        source2.setData({ type: 'FeatureCollection', features: level[i] });
    }
    // map.on('mousemove', (e) => {
    //     document.getElementById('info').innerHTML =
    //     // `e.point` is the x, y coordinates of the `mousemove` event
    //     // relative to the top-left corner of the map.
    //     JSON.stringify(e.point) +
    //     '<br />' +
    //     // `e.lngLat` is the longitude, latitude geographical position of the event.
    //     JSON.stringify(e.lngLat.wrap());
    // });

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
<div bind:this={container} style="margin-left: auto; margin-right: 0;z-index:1;position: absolute">
	{#if map}
		<slot />
	{/if}@
</div>

{#if map}
    {map.on('load', () => {
        seaLayer();
        initLine();
        for(let i = 0; i < 4; i++)
        {
            map.addSource(levelId[i], {
                type: 'geojson',
                data: { type: 'FeatureCollection', features: level[i] }
            });
            let colors = ['#00ff00','#448800','#884400','#ff0000'];
            map.addLayer({
                id: levelId[i],
                source: levelId[i],
                type: 'fill',
                paint: {
                    'fill-color': colors[i],
                    'fill-opacity': 0.6,
                    'fill-outline-color': 'transparent'
                }
            });
        }
        fetch("http://127.0.0.1:5001/api/location/getbox").then(x => x.json()).then(x => {
            console.log(x)
            x.forEach(element => {
                var timestamp = element[1];

                var bbox = [element[2],element[3],element[4],element[5]];
                var amount = element[6];
                selectBox(bbox,amount,amount, true);
            });
        }
        );
        console.log(map.getStyle().layers);
        map.moveLayer('Land');
    })}

    {map.on(MaplibreGrid.GRID_CLICK_EVENT, async function(event) {
        var bbox = event.bbox;
        console.log(bbox);

        if(drawMode == 'move'){
            console.log("draw in move mode");
            var [lon1,lat1,lon2,lat2] = bbox;
            var lon = (lon1 + lon2)/2;
            var lat = (lat1 + lat2)/2;
            console.log([lon,lat]);
            path.push([lon,lat]);
            if(path.length > 1)
            {
                map.getSource('path').setData(data);
            }
            console.log(path);
        }
        else{
            console.log("draw in fish mode");
            let onWater = await queryOnWater(lat, lon);
            // console.log(onWater);
            if (!onWater.water)
            {
                console.log("not on water");
                return;
            }
            fishAmount = 0;
            let resp = await fetch(`http://127.0.0.1:5001/api/location/getbox1/${bbox[0]}/${bbox[1]}/${bbox[2]}/${bbox[3]}`);
            let json = await resp.json();
            
            json.forEach(element => {
                    fishAmount += element[0];
            });

            //update fishAmount
            var oldAmount = fishAmount;
            console.log("oldAmount = " + oldAmount);
            fishAmount += 20;

            selectBox(bbox,fishAmount,oldAmount);
            fetch(`http://127.0.0.1:5001/api/location/setbox/${bbox[0]}/${bbox[1]}/${bbox[2]}/${bbox[3]}/${fishAmount}`, {
                mode: 'no-cors',
                headers: {
                    'Access-Control-Allow-Origin':'*'
                }
            })
        }

        /*
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
            // console.log("TRACE PATH START");

            for (let i = 0; i < distance; i++)
            {
                iter_lon += lon_step;
                iter_lat += lat_step;
                // console.log([iter_lon, iter_lat]);

                const bbox = getGridCell([iter_lon, iter_lat], 10, 10, 'kilometers');
                // todo: set amount
                let amount = 99;
                selectBox(bbox,amount);
                fetch(`http://127.0.0.1:5001/api/location/setbox/${bbox[0]}/${bbox[1]}/${bbox[2]}/${bbox[3]}/${amount}`, {
                mode: 'no-cors',
                headers: {
                    'Access-Control-Allow-Origin':'*'
                }
                })

                // console.log(bbox)
            }

            // console.log("TRACE PATH END")
        }*/
    })}
{/if}

<style>
	div {
		width: 100%;
		height: 100%;
	}
    #info {
        display: table;
        position: relative;
        margin: 0px auto;
        word-wrap: anywhere;
        white-space: pre-wrap;
        padding: 10px;
        border: none;
        border-radius: 3px;
        font-size: 12px;
        text-align: center;
        color: #222;
        background: #fff;
    }
    .marker {
        /* background-image: url('mapbox-icon.png'); */
        background-size: cover;
        width: 50px;
        height: 50px;
        border-radius: 50%;
        cursor: pointer;
    }

    .legend {
        background-color: #fff;
        border-radius: 3px;
        bottom: 30px;
        box-shadow: 0 1px 2px rgba(0, 0, 0, 0.1);
        font: 12px/20px 'Helvetica Neue', Arial, Helvetica, sans-serif;
        padding: 10px;
        position: absolute;
        right: 10px;
        z-index: 1;
    }
    
    .legend h4 {
        margin: 0 0 10px;
    }
    
    .legend div span {
        border-radius: 50%;
        display: inline-block;
        height: 10px;
        margin-right: 5px;
        width: 10px;
    }
</style>
