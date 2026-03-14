<script lang="ts">
	import { onMount } from 'svelte';

	let canvas: HTMLCanvasElement;

	onMount(() => {
		const ctx = canvas.getContext('2d')!;
		let animId: number;
		let mouse = { x: -1000, y: -1000 };
		let w: number, h: number;
		let dpr: number;

		const GAP = 28;
		const BASE_RADIUS = 1.5;
		const INFLUENCE = 120;
		const ACCENT = [255, 28, 212];
		const INK = [28, 28, 28];

		function resize() {
			const rect = canvas.getBoundingClientRect();
			dpr = Math.min(window.devicePixelRatio, 2);
			w = rect.width;
			h = rect.height;
			canvas.width = w * dpr;
			canvas.height = h * dpr;
			ctx.setTransform(dpr, 0, 0, dpr, 0, 0);
		}

		let time = 0;

		function draw() {
			ctx.clearRect(0, 0, w, h);
			time += 0.008;

			const cols = Math.ceil(w / GAP) + 1;
			const rows = Math.ceil(h / GAP) + 1;
			const offsetX = (w - (cols - 1) * GAP) / 2;
			const offsetY = (h - (rows - 1) * GAP) / 2;

			for (let row = 0; row < rows; row++) {
				for (let col = 0; col < cols; col++) {
					const baseX = offsetX + col * GAP;
					const baseY = offsetY + row * GAP;

					const dx = baseX - mouse.x;
					const dy = baseY - mouse.y;
					const dist = Math.sqrt(dx * dx + dy * dy);

					let displaceX = 0;
					let displaceY = 0;
					let scale = 1;
					let blend = 0;

					if (dist < INFLUENCE) {
						const t = 1 - dist / INFLUENCE;
						const ease = t * t;
						displaceX = (dx / dist) * ease * 14;
						displaceY = (dy / dist) * ease * 14;
						scale = 1 + ease * 2.5;
						blend = ease;
					}

					// Subtle ambient breathing
					const breathe = Math.sin(time + col * 0.3 + row * 0.2) * 0.15;
					scale += breathe;

					const x = baseX + displaceX;
					const y = baseY + displaceY;
					const r = BASE_RADIUS * scale;

					const cr = Math.round(INK[0] + (ACCENT[0] - INK[0]) * blend);
					const cg = Math.round(INK[1] + (ACCENT[1] - INK[1]) * blend);
					const cb = Math.round(INK[2] + (ACCENT[2] - INK[2]) * blend);
					const alpha = 0.25 + blend * 0.55;

					ctx.beginPath();
					ctx.arc(x, y, r, 0, Math.PI * 2);
					ctx.fillStyle = `rgba(${cr},${cg},${cb},${alpha})`;
					ctx.fill();
				}
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
		window.addEventListener('resize', resize);

		resize();
		draw();

		return () => {
			cancelAnimationFrame(animId);
			canvas.removeEventListener('mousemove', handleMouseMove);
			canvas.removeEventListener('mouseleave', handleMouseLeave);
			window.removeEventListener('resize', resize);
		};
	});
</script>

<canvas bind:this={canvas} class="absolute inset-0 h-full w-full"></canvas>
