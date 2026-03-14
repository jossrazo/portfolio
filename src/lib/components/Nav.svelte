<script lang="ts">
	import { page } from '$app/state';
	import { onMount } from 'svelte';

	const links = [
		{ href: '/work', label: 'Work' },
		{ href: '/about', label: 'About' }
	];

	let timeDisplay = $state('');
	let yearDisplay = $state('');

	function update() {
		const now = new Date();

		const h = String(now.getHours()).padStart(2, '0');
		const m = String(now.getMinutes()).padStart(2, '0');
		const s = String(now.getSeconds()).padStart(2, '0');
		timeDisplay = `${h}:${m}:${s}`;

		const start = new Date(now.getFullYear(), 0, 1).getTime();
		const end = new Date(now.getFullYear() + 1, 0, 1).getTime();
		const fraction = (now.getTime() - start) / (end - start);
		yearDisplay = (now.getFullYear() + fraction).toFixed(8);
	}

	onMount(() => {
		update();
		const interval = setInterval(update, 50);
		return () => clearInterval(interval);
	});
</script>

<nav class="mb-6 flex items-center justify-between py-8 sm:py-10">
	<a href="/" class="font-serif text-[1.375rem] leading-none tracking-tight">J. Razo</a>

	<div class="flex items-center gap-6 sm:gap-8">
		<span
			class="hidden font-mono text-[0.6875rem] tracking-wider text-ink-muted/50 tabular-nums sm:block"
		>
			{timeDisplay}&ensp;{yearDisplay}
		</span>

		<ul class="flex gap-8 text-[0.8125rem] tracking-[0.08em] uppercase">
			{#each links as { href, label }}
				<li>
					<a
						{href}
						class="text-ink-muted transition-colors duration-150 hover:text-ink"
						aria-current={page.url.pathname.startsWith(href) ? 'page' : undefined}
					>
						{label}
					</a>
				</li>
			{/each}
		</ul>
	</div>
</nav>

<style>
	[aria-current='page'] {
		color: var(--color-ink);
	}
</style>
