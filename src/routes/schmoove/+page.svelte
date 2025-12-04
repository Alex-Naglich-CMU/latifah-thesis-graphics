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
	let backgroundImage: string | null = $state(null);

	// Data Parsing
	let data: any[] = $state(d3.csvParse(rawCsvText));

	// Derived values that update when data changes
	let numberOfObservations = $derived(data.length > 0 ? Object.keys(data[0]).length / 2 : 0);

	let observations: { x: number; y: number }[][] = $derived.by(() => {
		const obs: { x: number; y: number }[][] = [];
		for (let i = 1; i <= numberOfObservations; i++) {
			const xKey = `x${i}`;
			const yKey = `y${i}`;

			obs.push(
				data.map((d) => ({
					x: +d[xKey], // Convert string to number
					y: +d[yKey] // Convert string to number
				}))
			);
		}
		return obs;
	});

	// File upload handler
	function handleFileUpload(event: Event) {
		const input = event.target as HTMLInputElement;
		const file = input.files?.[0];

		if (file && file.type.startsWith('image/')) {
			const reader = new FileReader();
			reader.onload = (e) => {
				backgroundImage = e.target?.result as string;
			};
			reader.readAsDataURL(file);
		} else if (file && (file.type === 'text/csv' || file.name.endsWith('.csv'))) {
			const reader = new FileReader();
			reader.onload = (e) => {
				const text = e.target?.result as string;
				data = d3.csvParse(text);
				// Adjust whichToShow array to match new data
				whichToShow = Array(numberOfObservations).fill(true);
			};
			reader.readAsText(file);
		}
	}
</script>

<div class="my-10 text-center">
	<h1 class="text-4xl font-bold">Crab Schmoovin' Visualization</h1>
	<p class="mt-2 text-lg">Adjust the parameters below to see how they affect the visualization.</p>
	<div>
		<!-- upload new background image -->
		<input
			type="file"
			accept="image/*"
			onchange={handleFileUpload}
			class="hidden"
			id="imageupload"
		/>
		<button
			class="mt-4 rounded bg-blue-500 px-4 py-2 text-white hover:bg-blue-600"
			onclick={() => document.getElementById('imageupload')?.click()}
		>
			Upload New Background Image
		</button>

		<!-- upload new data -->
		<input
			type="file"
			accept=".csv,text/csv"
			onchange={handleFileUpload}
			class="hidden"
			id="dataupload"
		/>
		<button
			class="mt-4 rounded bg-blue-500 px-4 py-2 text-white hover:bg-blue-600"
			onclick={() => document.getElementById('dataupload')?.click()}
		>
			Upload New Csv
		</button>
	</div>
</div>

<div class="sliders m-10 flex flex-row gap-20">
	<div class="gap-2 flex flex-col">
		<div>
			<label for="tailDuration" class="font-bold">Tail Duration: </label>
			<span> {tailDuration}</span>
			<input
				class="h-2 w-full cursor-pointer appearance-none rounded-lg bg-gray-200 dark:bg-gray-700"
				type="range"
				id="tailDuration"
				min="0"
				max="10"
				step="0.1"
				bind:value={tailDuration}
			/>
		</div>
		<div>
			<label for="dpf" class="font-bold">DataPoints Per Frame: </label>
			<span>{datapointsPerFrame} DPF</span>
			<input
				class="h-2 w-full cursor-pointer appearance-none rounded-lg bg-gray-200 dark:bg-gray-700"
				type="range"
				id="dpf"
				min="1"
				max="100"
				bind:value={datapointsPerFrame}
			/>
		</div>
		<div>
			<label for="radius" class="font-bold">Point Radius: </label> <span>{radius}</span>
			<input
				class="h-2 w-full cursor-pointer appearance-none rounded-lg bg-gray-200 dark:bg-gray-700"
				type="range"
				id="radius"
				min="1"
				max="10"
				bind:value={radius}
			/>
		</div>
		<div>
			<label for="borderColor" class="font-bold">Border Color: </label>
			<input type="color" id="borderColor" bind:value={borderColor} />
			<span>{borderColor}</span>
		</div>
		<div>
			<label for="borderWidth" class="font-bold">Border Width: </label><span> {borderwidth}</span>
			<input
				class="h-2 w-full cursor-pointer appearance-none rounded-lg bg-gray-200 dark:bg-gray-700"
				type="range"
				id="borderWidth"
				min="0"
				max="4"
				step="0.1"
				bind:value={borderwidth}
			/>
		</div>
	</div>
	<div>
		<label for="whichToShow">Which Observations to Show (less runs better) </label>
		<div class="grid grid-flow-col grid-rows-4 gap-2" id="whichToShow">
			{#each Array(numberOfObservations)
				.fill(0)
				.map((_, i) => i) as index}
				<div>
					<input type="checkbox" id="obs-{index}" bind:checked={whichToShow[index]} />
					<label for="obs-{index}">Observation {index + 1}</label>
				</div>
			{/each}
		</div>
	</div>
</div>

<div class="grid grid-cols-[repeat(3,20vw)]">
	{#each whichToShow as show, index}
		{#if show}
			<Schmoove
				observation={observations[index]}
				{backgroundImage}
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
