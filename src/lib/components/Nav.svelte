<script lang="ts">
	import { page } from '$app/state';

	const links = [
		{ href: '/work', label: 'Work' },
		{ href: '/about', label: 'About' }
	];
</script>

<nav class="flex items-center justify-between py-8 sm:py-10">
	<a href="/" class="font-serif text-[1.375rem] leading-none tracking-tight">J. Razo</a>

	<ul class="flex gap-8 text-[0.8125rem] tracking-[0.08em] uppercase">
		{#each links as { href, label }}
			<li>
				<a
					{href}
					class="text-ink-muted hover:text-ink transition-colors duration-150"
					aria-current={page.url.pathname.startsWith(href) ? 'page' : undefined}
				>
					{label}
				</a>
			</li>
		{/each}
	</ul>
</nav>

<style>
	[aria-current='page'] {
		color: var(--color-ink);
	}
</style>
