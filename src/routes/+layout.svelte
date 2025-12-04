<script lang="ts">
	import '../app.css';
	import crab from '$lib/assets/crab.jpg';
	import Nav from '$lib/Nav.svelte';

	let { children } = $props();
</script>

<svelte:head>
	<link rel="icon" href={crab} />
</svelte:head>
	<Nav />
<div class="container">
	<div class="content">
		{@render children?.()}
	</div>
</div>
