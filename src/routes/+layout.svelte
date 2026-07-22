<script lang="ts">
	import favicon from '$lib/assets/favicon.svg';

	let { children } = $props();
</script>

<svelte:head>
	<link rel="icon" href={favicon} />
</svelte:head>

{@render children()}

<style>
	:global(:root) {
		--bg: #060911;
		--bg-deep: #030509;
		--ocean-a: #0e2033;
		--ocean-b: #1b3a5c;
		--land: #3e7c6e;
		--land-dark: #2c5a50;
		--gold: #e8b75c;
		--gold-soft: rgba(232, 183, 92, 0.35);
		--gold-glow: rgba(232, 183, 92, 0.15);
		--ink: #f5f3ee;
		--ink-dim: #9ba3ae;
		--display: 'Space Grotesk', sans-serif;
		--body: 'IBM Plex Sans', sans-serif;
	}

	:global(*) {
		box-sizing: border-box;
	}

	:global(html, body) {
		margin: 0;
		padding: 0;
		width: 100%;
		height: 100%;
		background: var(--bg);
		color: var(--ink);
		font-family: var(--body);
		overflow: hidden;
	}

	:global(body.intro-done) {
		overflow-y: auto;
	}
</style>
