<script lang="ts">
	import { onMount } from 'svelte';

	let canvas: HTMLCanvasElement;

	onMount(() => {
		const ctx = canvas.getContext('2d')!;
		let animId: number;
		let mouse = { x: -1000, y: -1000 };
		let w: number, h: number;

		const COLS = 20;
		const ACCENT = [255, 28, 212];

		let time = 0;

		function resize() {
			const rect = canvas.getBoundingClientRect();
			const dpr = Math.min(window.devicePixelRatio, 2);
			w = rect.width;
			h = rect.height;
			canvas.width = w * dpr;
			canvas.height = h * dpr;
			ctx.setTransform(dpr, 0, 0, dpr, 0, 0);
		}

		function draw() {
			ctx.clearRect(0, 0, w, h);
			time += 0.02;

			const cellW = w / COLS;
			const rows = Math.ceil(h / cellW);

			for (let row = 0; row < rows; row++) {
				for (let col = 0; col < COLS; col++) {
					const cx = (col + 0.5) * cellW;
					const cy = (row + 0.5) * cellW;

					const dx = cx - mouse.x;
					const dy = cy - mouse.y;
					const dist = Math.sqrt(dx * dx + dy * dy);

					// Pulsing base size
					const wave = Math.sin(time + col * 0.4 + row * 0.3) * 0.5 + 0.5;
					const wave2 = Math.sin(time * 0.7 + col * 0.2 - row * 0.5) * 0.5 + 0.5;
					let size = (wave * 0.6 + wave2 * 0.4) * cellW * 0.35;

					let proximity = 0;
					if (dist < 180) {
						proximity = 1 - dist / 180;
						size += proximity * proximity * cellW * 0.25;
					}

					const alpha = 0.06 + wave * 0.06 + proximity * 0.2;
					const isAccent = proximity > 0.5;

					ctx.beginPath();
					ctx.arc(cx, cy, Math.max(size, 1), 0, Math.PI * 2);

					if (isAccent) {
						ctx.fillStyle = `rgba(${ACCENT[0]},${ACCENT[1]},${ACCENT[2]},${alpha})`;
					} else {
						ctx.fillStyle = `rgba(28,28,28,${alpha})`;
					}
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
