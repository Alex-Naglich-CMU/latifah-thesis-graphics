<script lang="ts">
	import { resolve } from '$app/paths';

	const navRoutes = [
		{ path: '/', name: 'Homepage' },
		{ path: '/schmoove', name: 'Schmoove' },
		{ path: 'https://www.linkedin.com/in/latifah-almaghrabi', name: 'Important Person' }
	];
</script>

<nav>
	<ul class="flex gap-10 list-none m-5 p-0">
		{#each navRoutes as { path, name }}
			<li class="font-bold text-2xl">
				<!-- External Links, necessary for GitHub Pages -->
				{#if path.startsWith('http')}
					<a href={path} target="_blank" rel="noopener noreferrer">
						{name}
					</a>
				<!-- Internal Links, uses <resolve> to append to the base path to avoid 404 errors -->
				{:else}
					<a href={resolve(path as any)}>
						{name}
					</a>
				{/if}
			</li>
		{/each}
	</ul>
</nav>
