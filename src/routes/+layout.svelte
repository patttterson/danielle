<script lang="ts">
	import "$lib/css/fonts.css";
	import { dev } from '$app/environment';

	let { children } = $props();
	const title = (dev ? ' [dev] ' : '') + 'TETR.IO Amateur Weeklies';
</script>

<svelte:head>
	<title>{title}</title>
	<link rel="preconnect" href="https://fonts.googleapis.com">
	<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin="anonymous">
	<link rel="stylesheet" href="https://fonts.googleapis.com/css2?family=Plus+Jakarta+Sans:ital,wght@0,500;0,600;0,700;0,800;1,500&family=Inter:wght@300;400;500;600&display=swap">
	<link rel="apple-touch-icon" sizes="180x180" href="/apple-touch-icon.png">
	<link rel="icon" type="image/png" sizes="32x32" href="/favicon-32x32.png">
	<link rel="icon" type="image/png" sizes="16x16" href="/favicon-16x16.png">
	<link rel="manifest" href="/site.webmanifest">
</svelte:head>

<style>
  :global(body) {
		--bg-overlay-rgb: 0, 0, 0;
		--bg-dim: 0.6;
		--font-heading: 'Plus Jakarta Sans', system-ui, sans-serif;
		--font-body: 'Inter', system-ui, sans-serif;
		font-family: var(--font-body);
		background: url('$lib/assets/backgrounds/gf.png');
		background-color: black;
		background-size: cover;
		background-position: center;
		background-repeat: no-repeat;
		color: white;
		display: flex;
		justify-content: center;
		align-items: flex-start;
		min-height: 100vh;
		margin: 0;
  }

	:global(body)::before {
		content: '';
		position: fixed;
		inset: 0;
		/* background: rgba(var(--bg-overlay-rgb), var(--bg-dim)); */
		pointer-events: none;
		z-index: -1;
	}

	:global(h1, h2, h3, h4, h5, h6) {
		font-family: var(--font-heading);
		letter-spacing: 0.02em;
	}

	:global(::selection), :global(::-moz-selection) {
		background: oklch(0.7411 0.2001 345.28);
	}

	#root {
		display: flex;
		flex-direction: column;
		align-items: center;
		width: 100%;
	}
</style>

<div id="root">
	{@render children()}
</div>