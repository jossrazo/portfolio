<script lang="ts">
	import { onMount } from 'svelte';

	let canvas: HTMLCanvasElement;

	onMount(() => {
		const ctx = canvas.getContext('2d')!;
		let animId: number;
		let mouse = { x: -1000, y: -1000 };
		let w: number, h: number;

		const COUNT = 300;
		const ACCENT = [255, 28, 212];

		type Particle = {
			x: number;
			y: number;
			homeX: number;
			homeY: number;
			size: number;
			accent: boolean;
		};

		let particles: Particle[] = [];

		function resize() {
			const rect = canvas.getBoundingClientRect();
			const dpr = Math.min(window.devicePixelRatio, 2);
			w = rect.width;
			h = rect.height;
			canvas.width = w * dpr;
			canvas.height = h * dpr;
			ctx.setTransform(dpr, 0, 0, dpr, 0, 0);
		}

		function init() {
			resize();
			particles = [];
			for (let i = 0; i < COUNT; i++) {
				const x = Math.random() * w;
				const y = Math.random() * h;
				particles.push({
					x,
					y,
					homeX: x,
					homeY: y,
					size: 1 + Math.random() * 2,
					accent: Math.random() < 0.07
				});
			}
		}

		function draw() {
			ctx.clearRect(0, 0, w, h);

			for (const p of particles) {
				const dx = mouse.x - p.homeX;
				const dy = mouse.y - p.homeY;
				const dist = Math.sqrt(dx * dx + dy * dy);

				let targetX = p.homeX;
				let targetY = p.homeY;

				if (dist < 200) {
					const t = 1 - dist / 200;
					const pull = t * t * 60;
					targetX = p.homeX + (dx / dist) * pull;
					targetY = p.homeY + (dy / dist) * pull;
				}

				// Smooth easing toward target
				p.x += (targetX - p.x) * 0.08;
				p.y += (targetY - p.y) * 0.08;

				// Draw connecting lines to nearby particles
				for (const q of particles) {
					if (p === q) continue;
					const px = p.x - q.x;
					const py = p.y - q.y;
					const pDist = Math.sqrt(px * px + py * py);
					if (pDist < 50) {
						const alpha = (1 - pDist / 50) * 0.08;
						ctx.beginPath();
						ctx.moveTo(p.x, p.y);
						ctx.lineTo(q.x, q.y);
						ctx.strokeStyle = `rgba(28,28,28,${alpha})`;
						ctx.lineWidth = 0.4;
						ctx.stroke();
					}
				}

				const nearMouse = dist < 200 ? 1 - dist / 200 : 0;
				const alpha = 0.2 + nearMouse * 0.5;

				ctx.beginPath();
				ctx.arc(p.x, p.y, p.size + nearMouse * 1.5, 0, Math.PI * 2);
				if (p.accent || nearMouse > 0.6) {
					ctx.fillStyle = `rgba(${ACCENT[0]},${ACCENT[1]},${ACCENT[2]},${alpha})`;
				} else {
					ctx.fillStyle = `rgba(28,28,28,${alpha})`;
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
