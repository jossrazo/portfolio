<script lang="ts">
	import { fly } from 'svelte/transition';
	import FlowField from '$lib/components/FlowField.svelte';
	import DotGrid from '$lib/components/DotGrid.svelte';
	import Constellation from '$lib/components/Constellation.svelte';
	import Orbits from '$lib/components/Orbits.svelte';
	import Magnetic from '$lib/components/Magnetic.svelte';
	import Pulse from '$lib/components/Pulse.svelte';
	import Traces from '$lib/components/Traces.svelte';
	import Lissajous from '$lib/components/Lissajous.svelte';
	import TouchFlow from '$lib/components/TouchFlow.svelte';

	const options = [
		'flow',
		'dots',
		'constellation',
		'orbits',
		'magnetic',
		'pulse',
		'traces',
		'lissajous'
	] as const;
	type Option = (typeof options)[number];
	let current: Option = $state('flow');
	let isMobile = $state(false);

	function cycle() {
		const idx = options.indexOf(current);
		current = options[(idx + 1) % options.length];
	}

	function checkMobile() {
		isMobile = window.innerWidth < 640;
	}

	import { onMount } from 'svelte';
	onMount(() => {
		checkMobile();
		window.addEventListener('resize', checkMobile);
		return () => window.removeEventListener('resize', checkMobile);
	});
</script>

<svelte:head>
	<title>J. Razo — Developer</title>
	<meta name="description" content="Developer portfolio of J. Razo" />
</svelte:head>

<section class="relative -mx-6 -mt-8 sm:-mx-10 sm:-mt-10">
	<div class="relative h-[82vh] min-h-[32rem] overflow-hidden">
		{#if isMobile}
			<TouchFlow />
		{:else}
			{#key current}
				{#if current === 'flow'}
					<FlowField />
				{:else if current === 'dots'}
					<DotGrid />
				{:else if current === 'constellation'}
					<Constellation />
				{:else if current === 'orbits'}
					<Orbits />
				{:else if current === 'magnetic'}
					<Magnetic />
				{:else if current === 'pulse'}
					<Pulse />
				{:else if current === 'traces'}
					<Traces />
				{:else}
					<Lissajous />
				{/if}
			{/key}
		{/if}

		<div
			class="pointer-events-none absolute inset-0 flex flex-col justify-end p-6 pb-16 sm:p-10 sm:pb-20"
		>
			<h1
				class="font-serif text-[clamp(2.5rem,8vw,6rem)] leading-[1.02] tracking-tight"
				in:fly={{ y: 24, duration: 600, delay: 200 }}
			>
				Joss Razo
			</h1>
			<p
				class="mt-3 text-lg text-ink-muted sm:text-xl"
				in:fly={{ y: 20, duration: 500, delay: 450 }}
			>
				Developer | Manila
			</p>
		</div>

		{#if !isMobile}
			<button
				class="absolute right-6 bottom-6 cursor-pointer rounded-full border border-rule bg-white/80 px-4 py-2 text-xs uppercase tracking-widest text-ink-muted backdrop-blur-sm transition-colors hover:text-ink sm:right-10 sm:bottom-10"
				onclick={cycle}
			>
				{current}
			</button>
		{/if}
	</div>
</section>
