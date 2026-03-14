<script lang="ts">
	import { onMount } from 'svelte';

	let canvas: HTMLCanvasElement;

	onMount(() => {
		const ctx = canvas.getContext('2d')!;
		let animId: number;
		let mouse = { x: -1000, y: -1000 };
		let w: number, h: number;

		const ACCENT = [255, 28, 212];
		const CURVE_COUNT = 6;

		type Curve = {
			a: number;
			b: number;
			delta: number;
			deltaSpeed: number;
			scale: number;
			accent: boolean;
			points: { x: number; y: number }[];
		};

		let curves: Curve[] = [];
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

		function init() {
			resize();
			curves = [];
			const baseScale = Math.min(w, h) * 0.28;
			for (let i = 0; i < CURVE_COUNT; i++) {
				curves.push({
					a: 1 + Math.floor(Math.random() * 5),
					b: 1 + Math.floor(Math.random() * 5),
					delta: Math.random() * Math.PI * 2,
					deltaSpeed: 0.002 + Math.random() * 0.006,
					scale: baseScale * (0.5 + Math.random() * 0.6),
					accent: i === 1 || i === 4,
					points: []
				});
			}
		}

		function draw() {
			ctx.fillStyle = 'rgba(255,255,255,0.015)';
			ctx.fillRect(0, 0, w, h);
			time += 0.01;

			const cx = w / 2;
			const cy = h / 2;

			// Mouse influence on center offset
			let offsetX = 0;
			let offsetY = 0;
			if (mouse.x > 0 && mouse.y > 0) {
				offsetX = (mouse.x - cx) * 0.06;
				offsetY = (mouse.y - cy) * 0.06;
			}

			for (const c of curves) {
				c.delta += c.deltaSpeed;

				// Mouse proximity affects frequency ratio
				let aModifier = 0;
				if (mouse.x > 0) {
					aModifier = Math.sin(time * 2) * 0.1 * (mouse.x / w);
				}

				const t = time * 2;
				const x = cx + offsetX + Math.sin((c.a + aModifier) * t + c.delta) * c.scale;
				const y = cy + offsetY + Math.sin(c.b * t) * c.scale;

				c.points.unshift({ x, y });
				if (c.points.length > 500) c.points.pop();

				// Draw curve
				if (c.points.length > 2) {
					ctx.beginPath();
					ctx.moveTo(c.points[0].x, c.points[0].y);

					for (let i = 1; i < c.points.length; i++) {
						const alpha = (1 - i / c.points.length) * (c.accent ? 0.15 : 0.06);
						ctx.strokeStyle = c.accent
							? `rgba(${ACCENT[0]},${ACCENT[1]},${ACCENT[2]},${alpha})`
							: `rgba(28,28,28,${alpha})`;
						ctx.lineWidth = c.accent ? 1.2 : 0.6;
						ctx.beginPath();
						ctx.moveTo(c.points[i - 1].x, c.points[i - 1].y);
						ctx.lineTo(c.points[i].x, c.points[i].y);
						ctx.stroke();
					}
				}

				// Draw head
				if (c.points.length > 0) {
					ctx.beginPath();
					ctx.arc(c.points[0].x, c.points[0].y, c.accent ? 2.5 : 1.5, 0, Math.PI * 2);
					ctx.fillStyle = c.accent
						? `rgba(${ACCENT[0]},${ACCENT[1]},${ACCENT[2]},0.5)`
						: 'rgba(28,28,28,0.25)';
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
