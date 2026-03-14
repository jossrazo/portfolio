<script lang="ts">
	import { onMount } from 'svelte';

	let canvas: HTMLCanvasElement;

	onMount(() => {
		const ctx = canvas.getContext('2d')!;
		let animId: number;
		let mouse = { x: -1000, y: -1000 };
		let w: number, h: number;

		const ACCENT = [255, 28, 212];
		const CHARS = '01.:;+*#=<>~^|/\\!?@%&$░▒▓'.split('');
		const CELL = 14;
		let cols: number;

		type Drop = {
			col: number;
			y: number;
			speed: number;
			length: number;
			chars: string[];
			phase: number;
		};

		let drops: Drop[] = [];

		function randomChar() {
			return CHARS[Math.floor(Math.random() * CHARS.length)];
		}

		function spawnDrop(col?: number): Drop {
			const c = col ?? Math.floor(Math.random() * cols);
			const len = 4 + Math.floor(Math.random() * 14);
			return {
				col: c,
				y: -len * CELL,
				speed: 1 + Math.random() * 2.5,
				length: len,
				chars: Array.from({ length: len }, randomChar),
				phase: Math.random() * Math.PI * 2
			};
		}

		function resize() {
			const rect = canvas.getBoundingClientRect();
			const dpr = Math.min(window.devicePixelRatio, 2);
			w = rect.width;
			h = rect.height;
			canvas.width = w * dpr;
			canvas.height = h * dpr;
			ctx.setTransform(dpr, 0, 0, dpr, 0, 0);

			cols = Math.ceil(w / CELL);
			drops = [];
			for (let i = 0; i < Math.floor(cols * 0.4); i++) {
				const d = spawnDrop();
				d.y = Math.random() * h;
				drops.push(d);
			}
		}

		let time = 0;

		function draw() {
			ctx.clearRect(0, 0, w, h);
			time += 0.02;

			ctx.font = `${CELL - 2}px monospace`;
			ctx.textAlign = 'center';
			ctx.textBaseline = 'middle';

			for (const drop of drops) {
				drop.y += drop.speed;

				if (Math.random() < 0.04) {
					const idx = Math.floor(Math.random() * drop.chars.length);
					drop.chars[idx] = randomChar();
				}

				const cx = drop.col * CELL + CELL / 2;

				for (let i = 0; i < drop.length; i++) {
					const cy = drop.y + i * CELL;
					if (cy < -CELL || cy > h + CELL) continue;

					const dx = cx - mouse.x;
					const dy = cy - mouse.y;
					const dist = Math.sqrt(dx * dx + dy * dy);
					const mouseNear = dist < 120;
					const mEase = mouseNear ? (1 - dist / 120) * (1 - dist / 120) : 0;

					const headFade = i / drop.length;
					const tailFade = 1 - (drop.length - 1 - i) / drop.length;
					const fade = Math.min(headFade * 3, tailFade * 2, 1);
					const baseAlpha = fade * 0.18;

					const pushX = mEase * (dx / (dist || 1)) * 6;
					const pushY = mEase * (dy / (dist || 1)) * 3;

					if (mEase > 0.1) {
						ctx.fillStyle = `rgba(${ACCENT[0]},${ACCENT[1]},${ACCENT[2]},${baseAlpha + mEase * 0.5})`;
					} else if (i === drop.length - 1) {
						ctx.fillStyle = `rgba(28,28,28,${fade * 0.35})`;
					} else {
						ctx.fillStyle = `rgba(28,28,28,${baseAlpha})`;
					}

					ctx.fillText(drop.chars[i], cx + pushX, cy + pushY);
				}
			}

			// Respawn off-screen drops
			for (let i = drops.length - 1; i >= 0; i--) {
				if (drops[i].y > h + drops[i].length * CELL) {
					drops[i] = spawnDrop();
				}
			}

			// Occasionally add new drops
			if (drops.length < cols * 0.5 && Math.random() < 0.05) {
				drops.push(spawnDrop());
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
