<script lang="ts">
	import * as d3 from 'd3';
	import defaultBackground from '$lib/assets/background.png';

	// Props, States, Types
	const {
		observation: observationData,
		index,
		datapointsPerFrame: datapointsPerFrame,
		tailDuration: tailDuration,
		radius: radius,
		borderColor: borderColor,
		borderwidth: borderwidth,
		backgroundImage: backgroundImage
	}: {
		observation: observationData;
		index: number;
		datapointsPerFrame: number;
		tailDuration: number;
		radius: number;
		borderColor: string;
		borderwidth: number;
		backgroundImage: string | null;
	} = $props();

	// Replace background image if provided
	let background = $derived(
		backgroundImage ? backgroundImage : defaultBackground
	);

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
	let pointsGroupRef = $state<SVGGElement>();

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

	// Color Scale for the cooling effect on the points
	const ColorScale = $derived(d3.scaleSequential(d3.interpolateInferno));

	const customColorScale = $derived(
		d3
			.scaleSequential()
			.domain([0, 1])
			.interpolator(
				d3.interpolateRgbBasis([d3.rgb(0, 0, 255), d3.rgb(255, 255, 255), d3.rgb(255, 0, 0)])
			)
	);

	// Axes, temporary variables to hold d3 axis. axis_<name> expects a scale as input and generates a set of ticks
	let xAxis = $derived(d3.axisBottom(xScale));
	let yAxis = $derived(d3.axisLeft(yScale));

	// Animation
	let visibleCount = $state(0);
	let animationKey = $derived(
		`${datapointsPerFrame}-${tailDuration}-${radius}-${borderColor}-${borderwidth}`
	);

	// Effects, to render axes when refs are available. D3 needs direct access to DOM elements to render axes
	$effect(() => {
		if (!xAxisRef || !yAxisRef) return;
		d3.select(xAxisRef).call(xAxis);
		d3.select(yAxisRef).call(yAxis);
	});

	// Animate points appearing and update colors with D3
	$effect(() => {
		if (!pointsGroupRef) return;
		// Reset when any prop changes
		animationKey;

		// Clear all circles on reset
		d3.select(pointsGroupRef!).selectAll('circle').remove();
		visibleCount = 0;
		let animationFrame: number;

		const animate = () => {
			if (visibleCount < cleanObservation.length) {
				const prevCount = visibleCount;
				visibleCount = Math.min(visibleCount + datapointsPerFrame, cleanObservation.length);

				// Only process NEW points (more efficient than full data join)
				const newPoints = cleanObservation.slice(prevCount, visibleCount);
				const selection = d3.select(pointsGroupRef!);

				// Directly append new circles without data join
				newPoints.forEach((d) => {
					selection
						.append('circle')
						.attr('cx', xScale(d.x))
						.attr('cy', yScale(d.y))
						.attr('r', radius)
						.attr('stroke', borderColor)
						.attr('stroke-width', borderwidth)
						.attr('fill', customColorScale(1))
						.style('pointer-events', 'none')
						.transition()
						.duration(tailDuration * 1000)
						.attr('fill', customColorScale(0))
						.ease(d3.easeCubicInOut);
				});

				animationFrame = requestAnimationFrame(animate);
			}
		};
		animationFrame = requestAnimationFrame(animate);
		return () => cancelAnimationFrame(animationFrame);
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
		<image
			href={background}
			x={xScale(15)}
			y={yScale(35)}
			width={xScale(40) - xScale(15)}
			height={yScale(10) - yScale(35)}
		/>
		<rect
			x={xScale(15)}
			y={yScale(35)}
			width={xScale(40) - xScale(15)}
			height={yScale(10) - yScale(35)}
			stroke="black"
			stroke-width="2"
			fill="rgba(255,255,255,0)"
		></rect>

		<g bind:this={pointsGroupRef}></g>

		<!-- <g class="x-axis" transform="translate(0,{height - margin.bottom})" bind:this={xAxisRef}></g>
		<g class="y-axis" transform="translate({margin.left},0)" bind:this={yAxisRef}></g> -->

		<text
			text-anchor="middle"
			x={margin.left + (width - margin.left - margin.right) / 2}
			y={15}
			font-family="Arial, sans-serif"
			class="text-lg font-bold"
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
/* .x-axis,
	.y-axis {
		font-family: Arial, sans-serif;
		font-size: 12px;
		color: rgba(0, 0, 0, 0.3);
	}
*/
</style>
