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
	<meta name="description" content="About Joss Razo" />
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
				I got into programming because I believe software, when built with care, can create
				real, positive change in people's lives. That's not just something I say; it's what I
				try to put into every project I work on.
			</p>
			<p use:reveal class="reveal-entry" style="--delay: 350ms">
				In every project I've been part of, I've been the one who leans in: taking ownership,
				asking questions before touching anything, and caring about whether what gets built is
				actually good. That's just how I tend to work.
			</p>
		</div>

		<aside use:reveal class="reveal-entry space-y-8 text-sm text-ink-muted" style="--delay: 400ms">
			<div>
				<h3 class="mb-2 text-xs font-medium tracking-[0.1em] text-ink uppercase">Currently</h3>
				<p>Open to Work</p>
			</div>
			<div>
				<h3 class="mb-3 text-xs font-medium tracking-[0.1em] text-ink uppercase">Experience</h3>
				<ul class="space-y-3">
					<li>
						<p class="text-ink">Web Developer</p>
						<p class="text-ink-muted">Collections by Calista · 2025</p>
					</li>
					<li>
						<p class="text-ink">Software Developer Intern</p>
						<p class="text-ink-muted">Shepherd's Voice Radio and Television · 2025</p>
					</li>
				</ul>
			</div>
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
			<h3 class="text-xs font-medium tracking-[0.1em] uppercase">Craft</h3>
			<p class="mt-4 leading-relaxed text-ink-muted">
				I care about how things are built, not just that they work. I try to understand what's
				happening one level below the tools I use, because that's what lets me make good
				decisions and not just follow patterns blindly. 
				
			</p>
		</div>

		<div use:reveal class="reveal-entry" style="--delay: 350ms">
			<h3 class="text-xs font-medium tracking-[0.1em] uppercase">Ownership</h3>
			<p class="mt-4 leading-relaxed text-ink-muted">
				Give me something to work on and I'll see it through. I'm comfortable working
				independently, pushing through ambiguity, and figuring things out when there's no
				clear answer. I also know when to stop guessing and ask, which I think is just as
				important.
			</p>
		</div>

		<div use:reveal class="reveal-entry" style="--delay: 500ms">
			<h3 class="text-xs font-medium tracking-[0.1em] uppercase">Intentionality</h3>
			<p class="mt-4 leading-relaxed text-ink-muted">
				Before I start anything, I want to understand what I'm actually building and why.
				Getting that right upfront is what keeps me from spending a week on the wrong thing.
				I'd rather ask one uncomfortable question early than discover a wrong assumption late.
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
