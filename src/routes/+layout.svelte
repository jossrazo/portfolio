<script lang="ts">
	import './layout.css';
	// import favicon from '$lib/assets/favicon.svg';
	import favicon from '$lib/assets/portfoliofavicon.svg';
	import Nav from '$lib/components/Nav.svelte';
	import Footer from '$lib/components/Footer.svelte';

	let { children } = $props();
</script>

<svelte:head>
	<link rel="icon" href={favicon} />
</svelte:head>

<div class="mx-auto max-w-[64rem] px-6 sm:px-10">
	<Nav />
	<main>
		{@render children()}
	</main>
	<Footer />
</div>
