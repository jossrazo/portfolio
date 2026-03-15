<script lang="ts">
	import { fly } from 'svelte/transition';
	import { onMount } from 'svelte';
	import AsciiWave from '$lib/components/AsciiWave.svelte';
	import TouchFlow from '$lib/components/TouchFlow.svelte';

	let isMobile = $state(false);

	function checkMobile() {
		isMobile = window.innerWidth < 640;
	}

	onMount(() => {
		checkMobile();
		window.addEventListener('resize', checkMobile);
		return () => window.removeEventListener('resize', checkMobile);
	});
</script>

<svelte:head>
	<title>Joss Razo</title>
	<meta name="description" content="Developer Portfolio" />
</svelte:head>

<section class="relative -mx-6 -mt-8 sm:-mx-10 sm:-mt-10">
	<div class="relative h-[82vh] min-h-[32rem] overflow-hidden">
		{#if isMobile}
			<TouchFlow />
		{:else}
			<AsciiWave />
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
				developer | metro manila
				<span class="pointer-events-auto"> | </span>
				<a
					href="https://drive.google.com/file/d/1nkYoQnmYo_BGeKu27WyryQ2fceITqssW/view?usp=sharing"
					target="_blank"
					rel="noopener noreferrer"
					class="pointer-events-auto text-ink-muted transition-colors hover:text-ink"
				>
					resume
				</a>
			</p>
		</div>

	</div>
</section>
