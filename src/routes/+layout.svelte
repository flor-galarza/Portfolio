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
		background: var(--bg, #060911);
		color: var(--ink);
		font-family: var(--body);
		overflow-y: auto;
		transition:
			background-color 0.4s ease-out,
			color 0.4s ease-out;
	}
</style>
