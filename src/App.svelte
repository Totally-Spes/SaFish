<script>

	import Map from './Map.svelte';
	import MapMarker from './MapMarker.svelte';
	import Menu from './Menu.svelte';
	
	let drawMode = 'move';

</script>

<Menu bind:drawMode={drawMode}/>

<div class='my-legend'>
	<div class='legend-scale'>
	  <ul class='legend-labels'>
		<li><span style='background:#;'></span>0%</li>
		<li><span style='background:#00FF00;'></span>0 - 20%</li>
		<li><span style='background:#448800;'></span>20 - 40%</li>
		<li><span style='background:#884400;'></span>40 - 60%</li>
		<li><span style='background:#ff0000;'></span>>60%</li>
	  </ul>
	</div>
</div>

<Map bind:drawMode={drawMode} lat={40} lon={0} zoom={3.5} >
</Map>


<style>
	.my-legend {
		margin: 12px;
	}
	  .my-legend .legend-scale ul {
		margin: 0;
		padding: 0;
		/* float: left; */
		list-style: none;
		z-index: 2;
		position: relative;
		padding: 5px;
		background-color: white;
		}
	  .my-legend .legend-scale ul li {
		display: block;
		float: left;
		width: 50px;
		margin-bottom: 6px;
		text-align: center;
		font-size: 80%;
		list-style: none;
		}
	  .my-legend ul.legend-labels li span {
		display: block;
		float: left;
		height: 15px;
		width: 50px;
		}
</style>