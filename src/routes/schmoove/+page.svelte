<script lang="ts">
	import Schmoove from '$lib/schmoove.svelte';
	import rawCsvText from '$lib/T22.csv?raw';
	import * as d3 from 'd3';

	//States
	let datapointsPerFrame: number = $state(5);
	let tailDuration: number = $state(2);
	let radius: number = $state(7);
	let borderColor: string = $state('#000000');
	let borderwidth: number = $state(0.5);
	let whichToShow: boolean[] = $state([true, true, true, true, true, true]);

	// Data Parsing
	let data: any[] = d3.csvParse(rawCsvText);
	let observations: { x: number; y: number }[][] = [];

	for (let i = 1; i <= 6; i++) {
		const xKey = `x${i}`;
		const yKey = `y${i}`;

		observations.push(
			data.map((d) => ({
				x: +d[xKey], // Convert string to number
				y: +d[yKey] // Convert string to number
			}))
		);
	}
</script>

<div class="sliders m-10 flex flex-row gap-20">
	<div>
		<div>
			<label for="tailDuration">Tail Duration: </label>
			<input type="range" id="tailDuration" min="0" max="10" step="0.1" bind:value={tailDuration} />
			<span>{tailDuration}</span>
		</div>
		<div>
			<label for="dpf">DataPoints Per Frame: </label>
			<input type="range" id="dpf" min="1" max="60" bind:value={datapointsPerFrame} />
			<span>{datapointsPerFrame} DPF</span>
		</div>
		<div>
			<label for="radius">Point Radius: </label>
			<input type="range" id="radius" min="1" max="10" bind:value={radius} />
			<span>{radius}</span>
		</div>
		<div>
			<label for="borderColor">Border Color: </label>
			<input type="color" id="borderColor" bind:value={borderColor} />
			<span>{borderColor}</span>
		</div>
		<div>
			<label for="borderWidth">Border Width: </label>
			<input type="range" id="borderWidth" min="0" max="4" step="0.1" bind:value={borderwidth} />
			<span>{borderwidth}</span>
		</div>
	</div>
	<div>
		<label for="whichToShow">Which Observations to Show (less runs better) </label>
		{#each [0, 1, 2, 3, 4, 5] as index}
			<div>
				<input type="checkbox" id="obs-{index}" bind:checked={whichToShow[index]} />
				<label for="obs-{index}">Observation {index + 1}</label>
			</div>
		{/each}
	</div>
</div>

<div class="grid grid-cols-[repeat(3,20vw)]">
	{#each whichToShow as show, index}
		{#if show}
			<Schmoove
				observation={observations[index]}
				{index}
				{datapointsPerFrame}
				{tailDuration}
				{radius}
				{borderColor}
				{borderwidth}
			/>
		{/if}
	{/each}
</div>

<!-- {#each observations as observation, index}
	<h2>Observation {index + 1}</h2>
	<pre>{JSON.stringify(observation, null, 2)}</pre>
{/each} -->
