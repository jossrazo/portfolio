<script lang="ts">
	import { fly, fade, slide } from 'svelte/transition';

	let expanded: Record<string, boolean> = $state({});
	let lightboxImages: string[] = $state([]);

	function toggle(id: string) {
		expanded[id] = !expanded[id];
	}

	function openLightbox(images: string[]) {
		lightboxImages = images;
	}

	function closeLightbox() {
		lightboxImages = [];
	}

	function handleKeydown(e: KeyboardEvent) {
		if (e.key === 'Escape') closeLightbox();
	}

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

	const projects = [
		{
			id: 'sensenote',
			year: '2026',
			type: 'Browser Extension',
			title: 'SenseNote',
			href: 'https://github.com/jossrazo/sensenote',
			summary: 'A pet project I built for highlighting and annotating text on any webpage.',
			description:
				'Built with vanilla JavaScript and zero dependencies. Highlights persist across page revisits using a context-based text matching system, and all notes are searchable, filterable, and exportable as Markdown.',
			images: ['/images/projects/sensenote1.png', '/images/projects/sensenote2.png']
		},
		{
			id: 'calista',
			year: '2025',
			type: 'E-commerce Store',
			title: 'Collections by Calista',
			href: 'https://collectionsbycalista.com',
			summary: 'Custom Shopify storefront built for an e-commerce client.',
			description:
				"I replaced multiple paid plugins with custom-built features — color swatches, order bumps, dynamic filtering — cutting the client's annual operating costs by over 50% and achieving a 4% conversion rate against Shopify's 1.3–1.8% average.",
			images: ['/images/projects/calista1.png', '/images/projects/calista2.png']
		},
		{
			id: 'feast',
			year: '2025',
			type: 'Event Management System',
			title: 'Feast Events',
			href: null,
			summary:
				"Event management platform built during my internship at Shepherd's Voice Radio and Television.",
			description:
				'I designed and developed the customer-facing storefront and checkout flow, an admin dashboard with revenue and sales visualizations, and ticket and discount management tools. Built with CodeIgniter 4, MySQL, and Bootstrap 5.',
			images: [
				'/images/projects/svrtv1.png',
				'/images/projects/svrtv2.png',
				'/images/projects/svrtv3.png',
				'/images/projects/svrtv4.png',
				'/images/projects/svrtv5.png'
			]
		},
		{
			id: 'salinsign',
			year: '2025',
			type: 'Capstone',
			title: 'SalinSign',
			href: 'https://github.com/jossrazo/salinsign-repo',
			summary:
				'A real-time Filipino Sign Language recognition system, built as my capstone project.',
			description:
				'A custom ML model trained with MediaPipe and scikit-learn recognizes 62 hand gestures at 91% accuracy, processed through a live pipeline from webcam capture to gesture classification. Designed to support communication between doctors and deaf patients.',
			images: []
		}
	];
</script>

<svelte:window onkeydown={handleKeydown} />

<svelte:head>
	<title>Work — J. Razo</title>
	<meta name="description" content="Selected projects and work by J. Razo" />
</svelte:head>

<section class="pt-20 pb-16 sm:pt-32 sm:pb-20">
	<h1
		class="font-serif text-5xl tracking-tight sm:text-6xl"
		in:fly={{ y: 20, duration: 500, delay: 100 }}
	>
		Work
	</h1>
	<p
		class="mt-6 max-w-md text-lg leading-relaxed text-ink-muted"
		in:fly={{ y: 20, duration: 500, delay: 250 }}
	>
		A selection of projects, experiments, and contributions.
	</p>
</section>

<section class="border-t border-rule">
	<ol class="divide-y divide-rule">
		{#each projects as project, i}
			<li use:reveal class="reveal-entry" style="--delay: {150 + i * 100}ms">
				<article class="grid grid-cols-1 gap-4 py-10 sm:grid-cols-[10rem_1fr] sm:gap-12">
					<div class="text-sm text-ink-muted">
						<p>{project.year}</p>
						<p class="mt-1">{project.type}</p>
					</div>
					<div>
						{#if project.href}
							<a
								href={project.href}
								target="_blank"
								rel="noopener noreferrer"
								class="group inline-flex items-baseline gap-2 font-serif text-[1.625rem] leading-snug tracking-tight transition-colors hover:text-accent"
							>
								<h2 class="font-serif text-[1.625rem] leading-snug tracking-tight">
									{project.title}
								</h2>
								<span
									class="text-[0.9em] opacity-0 transition-opacity group-hover:opacity-100"
									title="View project"
									aria-hidden="true">↗</span
								>
							</a>
						{:else}
							<h2 class="font-serif text-[1.625rem] leading-snug tracking-tight">
								{project.title}
							</h2>
						{/if}

						<p class="mt-3 leading-relaxed text-ink-muted">
							{project.summary}{#if !expanded[project.id]}<button
									class="ml-1 inline cursor-pointer text-ink-muted/70 transition-colors hover:text-ink"
									onclick={() => toggle(project.id)}
								>
									...See more
								</button>{/if}
						</p>

						{#if expanded[project.id]}
							<div transition:slide={{ duration: 300 }}>
								<p class="mt-2 leading-relaxed text-ink-muted">
									{project.description}
								</p>

								{#if project.images.length > 0}
									<div
										class="mt-6 flex gap-4 overflow-x-auto scroll-smooth snap-x snap-mandatory p-2"
									>
										{#each project.images as src, imgIdx}
											<button
												class="shrink-0 cursor-zoom-in snap-start overflow-hidden rounded border border-rule transition-all duration-200 hover:scale-[1.02] hover:shadow-md"
												onclick={() => openLightbox(project.images)}
												in:fly={{ y: 16, duration: 350, delay: imgIdx * 80 }}
											>
												<img
													{src}
													alt="{project.title} screenshot"
													class="h-80 w-auto object-cover sm:h-96"
												/>
											</button>
										{/each}
									</div>
								{/if}

								<button
									class="mt-4 cursor-pointer text-sm text-ink-muted/70 transition-colors hover:text-ink"
									onclick={() => toggle(project.id)}
								>
									See less
								</button>
							</div>
						{/if}
					</div>
				</article>
			</li>
		{/each}
	</ol>
</section>

{#if lightboxImages.length > 0}
	<!-- svelte-ignore a11y_no_noninteractive_element_interactions -->
	<div
		class="fixed inset-0 z-50 cursor-zoom-out overflow-y-auto bg-black/80 backdrop-blur-sm"
		role="dialog"
		aria-modal="true"
		tabindex="-1"
		onclick={closeLightbox}
		onkeydown={(e) => {
			if (e.key === 'Escape') closeLightbox();
		}}
		transition:fade={{ duration: 200 }}
	>
		<div class="flex min-h-full flex-col items-center gap-6 px-6 py-12 sm:py-16">
			{#each lightboxImages as src, i}
				<img
					{src}
					alt="Project screenshot"
					class="max-w-[90vw] cursor-default rounded object-contain shadow-2xl sm:max-w-[72vw]"
					in:fly={{ y: 30, duration: 400, delay: 100 + i * 80 }}
				/>
			{/each}
		</div>
	</div>
{/if}

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
