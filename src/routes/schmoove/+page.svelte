<script lang="ts">
	import Schmoove from '$lib/schmoove.svelte';
	import rawCsvText from '$lib/T22.csv?raw';
	import * as d3 from 'd3';

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

<div class="grid grid-cols-[repeat(3,25vw)]">
	{#each observations as observation, index}
		<Schmoove {observation} {index} />
	{/each}
</div>

<!-- Display parsed data in a readable way -->
<!-- {#each observations as observation, index}
	<h2>Observation {index + 1}</h2>
	<pre>{JSON.stringify(observation, null, 2)}</pre>
{/each} -->
