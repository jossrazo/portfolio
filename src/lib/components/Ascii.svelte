<script lang="ts">
	import { onMount } from 'svelte';

	let canvas: HTMLCanvasElement;

	onMount(() => {
		const ctx = canvas.getContext('2d')!;
		let animId: number;
		let mouse = { x: -1000, y: -1000 };
		let w: number, h: number;

		const ACCENT = [255, 28, 212];
		const CHARS = '.·:;+*#@%&$?!=<>~^░▒▓'.split('');
		const CELL = 14;
		let cols: number, rows: number;
		let grid: { char: string; targetChar: string; phase: number; speed: number; glow: number }[] = [];
		let time = 0;

		function createNoise() {
			const perm = new Uint8Array(512);
			for (let i = 0; i < 256; i++) perm[i] = i;
			for (let i = 255; i > 0; i--) {
				const j = Math.floor(Math.random() * (i + 1));
				[perm[i], perm[j]] = [perm[j], perm[i]];
			}
			for (let i = 0; i < 256; i++) perm[i + 256] = perm[i];

			return function (x: number, y: number): number {
				const xi = Math.floor(x) & 255;
				const yi = Math.floor(y) & 255;
				const xf = x - Math.floor(x);
				const yf = y - Math.floor(y);
				const u = xf * xf * (3 - 2 * xf);
				const v = yf * yf * (3 - 2 * yf);
				const a = perm[perm[xi] + yi];
				const b = perm[perm[xi + 1] + yi];
				const c = perm[perm[xi] + yi + 1];
				const d = perm[perm[xi + 1] + yi + 1];
				return (a + u * (b - a) + v * (c - a) + u * v * (a - b - c + d)) / 255;
			};
		}

		const noise = createNoise();

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

			grid = [];
			for (let i = 0; i < cols * rows; i++) {
				const char = CHARS[Math.floor(Math.random() * CHARS.length)];
				grid.push({
					char,
					targetChar: char,
					phase: Math.random() * Math.PI * 2,
					speed: 0.01 + Math.random() * 0.02,
					glow: 0
				});
			}
		}

		function draw() {
			ctx.clearRect(0, 0, w, h);
			time += 0.008;

			ctx.font = `${CELL - 2}px monospace`;
			ctx.textAlign = 'center';
			ctx.textBaseline = 'middle';

			for (let row = 0; row < rows; row++) {
				for (let col = 0; col < cols; col++) {
					const idx = row * cols + col;
					const cell = grid[idx];
					const cx = col * CELL + CELL / 2;
					const cy = row * CELL + CELL / 2;

					const dx = cx - mouse.x;
					const dy = cy - mouse.y;
					const dist = Math.sqrt(dx * dx + dy * dy);

					const n = noise(col * 0.08 + time, row * 0.08 + time * 0.5);
					const wave = Math.sin(cell.phase + time * 3) * 0.5 + 0.5;

					const mouseInfluence = dist < 150 ? (1 - dist / 150) : 0;
					const mEase = mouseInfluence * mouseInfluence;

					// Pick character based on noise + mouse
					const charIdx = Math.floor((n * 0.6 + wave * 0.2 + mEase * 0.8) * (CHARS.length - 1));
					cell.targetChar = CHARS[Math.min(charIdx, CHARS.length - 1)];
					if (Math.random() < 0.03 + mEase * 0.3) {
						cell.char = cell.targetChar;
					}

					// Glow near mouse
					cell.glow += (mEase - cell.glow) * 0.15;

					const baseAlpha = 0.06 + n * 0.12 + wave * 0.04;
					const alpha = baseAlpha + cell.glow * 0.6;

					if (cell.glow > 0.15) {
						const accentAlpha = cell.glow * 0.7;
						ctx.fillStyle = `rgba(${ACCENT[0]},${ACCENT[1]},${ACCENT[2]},${accentAlpha})`;
					} else {
						ctx.fillStyle = `rgba(28,28,28,${alpha})`;
					}

					// Slight displacement from mouse
					const pushX = mEase * (dx / (dist || 1)) * 3;
					const pushY = mEase * (dy / (dist || 1)) * 3;

					ctx.fillText(cell.char, cx + pushX, cy + pushY);
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
