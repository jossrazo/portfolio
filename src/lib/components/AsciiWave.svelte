<script lang="ts">
	import { onMount } from 'svelte';

	let canvas: HTMLCanvasElement;

	onMount(() => {
		const ctx = canvas.getContext('2d')!;
		let animId: number;
		let mouse = { x: -1000, y: -1000 };
		let w: number, h: number;

		const ACCENT = [255, 28, 212];
		const DENSITY = ' .·:;=+*#%@▓'.split('');
		const CELL = 13;
		let cols: number, rows: number;
		let ripples: { x: number; y: number; time: number; strength: number }[] = [];
		let time = 0;

		function resize() {
			const rect = canvas.getBoundingClientRect();
			const dpr = Math.min(window.devicePixelRatio, 2);
			w = rect.width;
			h = rect.height;
			canvas.width = w * dpr;
			canvas.height = h * dpr;
			ctx.setTransform(dpr, 0, 0, dpr, 0, 0);
			cols = Math.ceil(w / CELL);
			rows = Math.ceil(h / CELL);
		}

		function draw() {
			ctx.clearRect(0, 0, w, h);
			time += 0.025;

			ctx.font = `${CELL - 1}px monospace`;
			ctx.textAlign = 'center';
			ctx.textBaseline = 'middle';

			// Decay old ripples
			ripples = ripples.filter((r) => time - r.time < 6);

			for (let row = 0; row < rows; row++) {
				for (let col = 0; col < cols; col++) {
					const cx = col * CELL + CELL / 2;
					const cy = row * CELL + CELL / 2;

					// Base wave — layered sine waves creating a terrain
					let val =
						Math.sin(col * 0.15 + time * 0.8) * 0.3 +
						Math.sin(row * 0.12 - time * 0.6) * 0.25 +
						Math.sin((col + row) * 0.1 + time * 0.4) * 0.2 +
						Math.sin(col * 0.05 - row * 0.08 + time * 0.3) * 0.15;

					// Mouse proximity — creates a radial peak
					const dx = cx - mouse.x;
					const dy = cy - mouse.y;
					const dist = Math.sqrt(dx * dx + dy * dy);
					const mouseInfluence = dist < 160 ? (1 - dist / 160) : 0;
					const mEase = mouseInfluence * mouseInfluence;
					val += mEase * 0.6;

					// Ripple contributions
					for (const r of ripples) {
						const rdx = cx - r.x;
						const rdy = cy - r.y;
						const rDist = Math.sqrt(rdx * rdx + rdy * rdy);
						const age = time - r.time;
						const waveRadius = age * 80;
						const ringDist = Math.abs(rDist - waveRadius);
						if (ringDist < 40) {
							const ringFade = 1 - ringDist / 40;
							const ageFade = Math.max(0, 1 - age / 6);
							val += ringFade * ageFade * r.strength * 0.5;
						}
					}

					// Map value to character density
					const normalized = (val + 1) * 0.5;
					const charIdx = Math.floor(Math.max(0, Math.min(1, normalized)) * (DENSITY.length - 1));
					const char = DENSITY[charIdx];

					if (char === ' ') continue;

					const baseAlpha = 0.05 + normalized * 0.2;

					if (mEase > 0.15) {
						ctx.fillStyle = `rgba(${ACCENT[0]},${ACCENT[1]},${ACCENT[2]},${baseAlpha + mEase * 0.55})`;
					} else {
						ctx.fillStyle = `rgba(28,28,28,${baseAlpha})`;
					}

					ctx.fillText(char, cx, cy);
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

		function handleClick(e: MouseEvent) {
			const rect = canvas.getBoundingClientRect();
			ripples.push({
				x: e.clientX - rect.left,
				y: e.clientY - rect.top,
				time,
				strength: 0.8
			});
		}

		canvas.addEventListener('mousemove', handleMouseMove);
		canvas.addEventListener('mouseleave', handleMouseLeave);
		canvas.addEventListener('click', handleClick);
		window.addEventListener('resize', resize);

		resize();
		draw();

		return () => {
			cancelAnimationFrame(animId);
			canvas.removeEventListener('mousemove', handleMouseMove);
			canvas.removeEventListener('mouseleave', handleMouseLeave);
			canvas.removeEventListener('click', handleClick);
			window.removeEventListener('resize', resize);
		};
	});
</script>

<canvas bind:this={canvas} class="absolute inset-0 h-full w-full"></canvas>
