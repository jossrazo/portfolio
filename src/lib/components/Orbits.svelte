<script lang="ts">
	import { onMount } from 'svelte';

	let canvas: HTMLCanvasElement;

	onMount(() => {
		const ctx = canvas.getContext('2d')!;
		let animId: number;
		let mouse = { x: -1000, y: -1000 };
		let w: number, h: number;

		const RING_COUNT = 12;
		const PARTICLES_PER_RING = 24;
		const ACCENT = [255, 28, 212];

		type Particle = {
			ring: number;
			angle: number;
			speed: number;
			radius: number;
			size: number;
			accent: boolean;
		};

		let particles: Particle[] = [];
		let centerX: number, centerY: number;

		function resize() {
			const rect = canvas.getBoundingClientRect();
			const dpr = Math.min(window.devicePixelRatio, 2);
			w = rect.width;
			h = rect.height;
			canvas.width = w * dpr;
			canvas.height = h * dpr;
			ctx.setTransform(dpr, 0, 0, dpr, 0, 0);
			centerX = w / 2;
			centerY = h / 2;
		}

		function init() {
			resize();
			particles = [];
			const maxRadius = Math.min(w, h) * 0.42;

			for (let ring = 0; ring < RING_COUNT; ring++) {
				const ringRadius = ((ring + 1) / RING_COUNT) * maxRadius;
				const count = PARTICLES_PER_RING + ring * 2;
				const direction = ring % 2 === 0 ? 1 : -1;
				const baseSpeed = (0.0008 + Math.random() * 0.001) * direction;

				for (let j = 0; j < count; j++) {
					particles.push({
						ring,
						angle: (j / count) * Math.PI * 2 + Math.random() * 0.3,
						speed: baseSpeed * (0.8 + Math.random() * 0.4),
						radius: ringRadius + (Math.random() - 0.5) * 8,
						size: 0.8 + Math.random() * 1.2,
						accent: Math.random() < 0.06
					});
				}
			}
		}

		function draw() {
			ctx.fillStyle = 'rgba(255,255,255,0.06)';
			ctx.fillRect(0, 0, w, h);

			// Draw faint ring paths
			const maxRadius = Math.min(w, h) * 0.42;
			for (let ring = 0; ring < RING_COUNT; ring++) {
				const r = ((ring + 1) / RING_COUNT) * maxRadius;
				ctx.beginPath();
				ctx.arc(centerX, centerY, r, 0, Math.PI * 2);
				ctx.strokeStyle = 'rgba(28,28,28,0.03)';
				ctx.lineWidth = 0.5;
				ctx.stroke();
			}

			for (const p of particles) {
				p.angle += p.speed;

				// Mouse gravity — gently attract particles
				const px = centerX + Math.cos(p.angle) * p.radius;
				const py = centerY + Math.sin(p.angle) * p.radius;
				const dx = px - mouse.x;
				const dy = py - mouse.y;
				const dist = Math.sqrt(dx * dx + dy * dy);

				let offsetX = 0;
				let offsetY = 0;
				if (dist < 150 && dist > 0) {
					const t = (1 - dist / 150) * 0.4;
					offsetX = -dx * t * 0.15;
					offsetY = -dy * t * 0.15;
				}

				const x = px + offsetX;
				const y = py + offsetY;

				ctx.beginPath();
				ctx.arc(x, y, p.size, 0, Math.PI * 2);
				if (p.accent) {
					ctx.fillStyle = `rgba(${ACCENT[0]},${ACCENT[1]},${ACCENT[2]},0.4)`;
				} else {
					ctx.fillStyle = 'rgba(28,28,28,0.12)';
				}
				ctx.fill();
			}

			animId = requestAnimationFrame(draw);
		}

		function handleMouseMove(e: MouseEvent) {
			const rect = canvas.getBoundingClientRect();
			mouse.x = e.clientX - rect.left;
			mouse.y = e.clientY - rect.top;
		}

		function handleMouseLeave() {
			mouse.x = -1000;
			mouse.y = -1000;
		}

		canvas.addEventListener('mousemove', handleMouseMove);
		canvas.addEventListener('mouseleave', handleMouseLeave);
		window.addEventListener('resize', init);

		init();
		draw();

		return () => {
			cancelAnimationFrame(animId);
			canvas.removeEventListener('mousemove', handleMouseMove);
			canvas.removeEventListener('mouseleave', handleMouseLeave);
			window.removeEventListener('resize', init);
		};
	});
</script>

<canvas bind:this={canvas} class="absolute inset-0 h-full w-full"></canvas>
