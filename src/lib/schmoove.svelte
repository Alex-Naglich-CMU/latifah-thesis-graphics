<script lang="ts">
	import * as d3 from 'd3';

	// Props, States, Types
	const { observation: observationData, index }: { observation: observationData, index: number } = $props();
	type observationData = { x: number; y: number }[];

	let cleanObservation = $derived(
		observationData.filter((d) => !Number.isNaN(d.x) && !Number.isNaN(d.y))
	);

	// Dimensions and Margins
	let containerWidth = $state(0);
	const width = $derived(containerWidth);
	const height = $derived(width);
	const margin = { top: 30, right: 20, bottom: 20, left: 30 };

	// SVG refs, must exist for D3 to bind to
	let mainSvgRef = $state<SVGSVGElement>();
	let xAxisRef = $state<SVGGElement>();
	let yAxisRef = $state<SVGGElement>();

	// Scales, define the extent of the data for d3 to map to screen coordinates
	let xScale = $derived(
		d3
			.scaleLinear()
			.range([margin.left, width - margin.right])
			.domain([13, 42])
	);

	let yScale = $derived(
		d3
			.scaleLinear()
			.range([height - margin.bottom, margin.top])
			.domain([8, 37])
	);

	// Axes, temporary variables to hold d3 axis. axis_<name> expects a scale as input and generates a set of ticks
	let xAxis = $derived(d3.axisBottom(xScale));
	let yAxis = $derived(d3.axisLeft(yScale));

	// Effects, to render axes when refs are available. D3 needs direct access to DOM elements to render axes
	$effect(() => {
		if (!xAxisRef || !yAxisRef) return;
		d3.select(xAxisRef).call(xAxis);
		d3.select(yAxisRef).call(yAxis);
	});
</script>

<div class="chart-wrapper relative" bind:clientWidth={containerWidth}>
	<!-- <div
		class="absolute z-10 w-full text-center text-3xl"
		style="font-family: Calibri, sans-serif; color: black;"
	>
		Schmoove Chart
	</div> -->
	<svg {width} {height} role="img" bind:this={mainSvgRef}>
		<rect
			x={xScale(15)}
			y={yScale(35)}
			width={xScale(40) - xScale(15)}
			height={yScale(10) - yScale(35)}
			stroke="black"
			stroke-width="2"
			fill="rgba(0,0,0,0.3)"
		></rect>

		{#each cleanObservation as point}
			<circle
				cx={xScale(point.x)}
				cy={yScale(point.y)}
				r="3"
				fill="red"
				stroke="black"
				stroke-width="1.5"
				style="pointer-events: none;"
			/>
		{/each}

		<g class="x-axis" transform="translate(0,{height - margin.bottom})" bind:this={xAxisRef}></g>
		<g class="y-axis" transform="translate({margin.left},0)" bind:this={yAxisRef}></g>

		<text
			text-anchor="middle"
			x={margin.left + (width - margin.left - margin.right) / 2}
			y={15}
			font-family="Arial, sans-serif"
			class="font-bold text-lg"

		>
			Observation {index + 1}
		</text>

		<!-- <text
			text-anchor="middle"
			x={margin.left + (width - margin.left - margin.right) / 2}
			y={height - 15}
			font-family="Arial, sans-serif"
			font-size="14px"
			font-weight="600"
		>
			X
		</text>

		<text
			text-anchor="middle"
			transform="rotate(-90)"
			x={-(margin.top + (height - margin.top - margin.bottom) / 2)}
			y={20}
			font-family="Arial, sans-serif"
			font-size="14px"
			font-weight="600"
		>
			Y
		</text> -->
	</svg>
</div>

<style>
.x-axis,
.y-axis {
	font-family: Arial, sans-serif;
	font-size: 12px;
	color: rgba(0,0,0,0.3);
}

</style>