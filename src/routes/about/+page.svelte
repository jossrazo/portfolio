<script lang="ts">
	import { fly } from 'svelte/transition';

	function reveal(node: HTMLElement) {
		const observer = new IntersectionObserver(
			([entry]) => {
				if (entry.isIntersecting) {
					node.classList.add('is-visible');
					observer.unobserve(node);
				}
			},
			{ threshold: 0.15 }
		);
		observer.observe(node);
		return { destroy: () => observer.disconnect() };
	}
</script>

<svelte:head>
	<title>About — J. Razo</title>
	<meta name="description" content="About J. Razo — developer and creative technologist" />
</svelte:head>

<section class="pt-20 pb-16 sm:pt-32 sm:pb-20">
	<h1
		class="font-serif text-5xl tracking-tight sm:text-6xl"
		in:fly={{ y: 20, duration: 500, delay: 100 }}
	>
		About
	</h1>
</section>

<section class="border-t border-rule py-16 sm:py-20">
	<div class="grid grid-cols-1 gap-12 sm:grid-cols-[1fr_14rem] sm:gap-20">
		<div class="space-y-6 text-lg leading-relaxed text-ink-muted">
			<p use:reveal class="reveal-entry" style="--delay: 200ms">
				I value building software that's purposeful, solving real problems without unnecessary
				complexity. I'm drawn to technologies that share that ethos: tools that respect both the
				developer and the end user. When I build something, I want it to work well, look
				intentional, and be something I'd actually use myself.
			</p>
			<p use:reveal class="reveal-entry" style="--delay: 350ms">
				What draws me to this work is the belief that well-built software can be a vehicle for real, positive change. That's what I want to spend my time on.
			</p>
		</div>

		<aside use:reveal class="reveal-entry space-y-8 text-sm text-ink-muted" style="--delay: 400ms">
			<div>
				<h3 class="mb-2 text-xs font-medium tracking-[0.1em] text-ink uppercase">Currently</h3>
				<p>Open to Work</p>
			</div>
			<!-- <div>
				<h3 class="mb-2 text-xs font-medium tracking-[0.1em] text-ink uppercase">Technologies</h3>
				<p>TypeScript, Svelte, PHP, Node.js, Python</p>
			</div> -->
			<div>
				<h3 class="mb-2 text-xs font-medium tracking-[0.1em] text-ink uppercase">Elsewhere</h3>
				<ul class="mt-2 space-y-1">
					<li>
						<a
							href="https://github.com/jossrazo"
							target="_blank"
							rel="noopener noreferrer"
							class="transition-colors hover:text-ink">GitHub</a
						>
					</li>
					<li>
						<a
							href="https://linkedin.com/in/joss-razo"
							target="_blank"
							rel="noopener noreferrer"
							class="transition-colors hover:text-ink">LinkedIn</a
						>
					</li>
					<li>
						<a href="mailto:razojossmichael@gmail.com" class="transition-colors hover:text-ink"
							>razojossmichael@gmail.com</a
						>
					</li>
				</ul>
			</div>
		</aside>
	</div>
</section>

<section class="border-t border-rule py-16 sm:py-20">
	<h2
		use:reveal
		class="reveal-entry font-serif text-3xl tracking-tight sm:text-4xl"
		style="--delay: 100ms"
	>
		Approach to Work
	</h2>

	<div class="mt-12 grid grid-cols-1 gap-14 sm:grid-cols-3 sm:gap-10">
		<div use:reveal class="reveal-entry" style="--delay: 200ms">
			<h3 class="text-xs font-medium tracking-[0.1em] uppercase">Technical</h3>
			<p class="mt-4 leading-relaxed text-ink-muted">
				I care about the craft of writing software. I use AI-assisted development to speed up
				my workflow and get more done, but I am a firm believer that good engineering comes from
				understanding how things work beneath the surface. I make a point to learn one layer deeper
				than the abstractions I use so I can build with confidence and adapt as technologies evolve.
			</p>
		</div>

		<div use:reveal class="reveal-entry" style="--delay: 350ms">
			<h3 class="text-xs font-medium tracking-[0.1em] uppercase">Autonomy</h3>
			<p class="mt-4 leading-relaxed text-ink-muted">
				I can pick up a task and run with it. I'm comfortable figuring things out on my own, but
				I've learned that knowing when to ask for help is just as important as knowing how to push
				through alone.
			</p>
		</div>

		<div use:reveal class="reveal-entry" style="--delay: 500ms">
			<h3 class="text-xs font-medium tracking-[0.1em] uppercase">Problem-Solving</h3>
			<p class="mt-4 leading-relaxed text-ink-muted">
				Before I touch anything, I spend time understanding what I'm actually solving for — the
				context, the constraints, the edge cases. From there, I break it into digestible chunks and
				tackle them one at a time. It's slower at the start, but it saves me from building the wrong
				thing.
			</p>
		</div>
	</div>
</section>

<style>
	.reveal-entry {
		opacity: 0;
		transform: translateY(16px);
		transition:
			opacity 0.5s ease var(--delay),
			transform 0.5s ease var(--delay);
	}

	:global(.reveal-entry.is-visible) {
		opacity: 1;
		transform: translateY(0);
	}
</style>
