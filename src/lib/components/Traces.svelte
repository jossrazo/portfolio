<script lang="ts">
	import { onMount } from 'svelte';

	let canvas: HTMLCanvasElement;

	onMount(() => {
		const ctx = canvas.getContext('2d')!;
		let animId: number;
		let mouse = { x: -1000, y: -1000 };
		let w: number, h: number;

		const ACCENT = [255, 28, 212];
		const TRAIL_LENGTH = 60;

		type Tracer = {
			points: { x: number; y: number }[];
			speed: number;
			angle: number;
			turnSpeed: number;
			accent: boolean;
		};

		let tracers: Tracer[] = [];
		const TRACER_COUNT = 40;

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
			tracers = [];
			for (let i = 0; i < TRACER_COUNT; i++) {
				const points = [];
				const x = Math.random() * w;
				const y = Math.random() * h;
				for (let j = 0; j < TRAIL_LENGTH; j++) {
					points.push({ x, y });
				}
				tracers.push({
					points,
					speed: 0.8 + Math.random() * 1.2,
					angle: Math.random() * Math.PI * 2,
					turnSpeed: (Math.random() - 0.5) * 0.03,
					accent: Math.random() < 0.1
				});
			}
		}

		function draw() {
			ctx.fillStyle = 'rgba(255,255,255,0.04)';
			ctx.fillRect(0, 0, w, h);

			for (const t of tracers) {
				const head = t.points[0];

				// Gentle turning + mouse attraction
				t.angle += t.turnSpeed;
				t.angle += Math.sin(t.angle * 2) * 0.005;

				const dx = mouse.x - head.x;
				const dy = mouse.y - head.y;
				const dist = Math.sqrt(dx * dx + dy * dy);
				if (dist < 250 && dist > 0) {
					const targetAngle = Math.atan2(dy, dx);
					let diff = targetAngle - t.angle;
					while (diff > Math.PI) diff -= Math.PI * 2;
					while (diff < -Math.PI) diff += Math.PI * 2;
					const attraction = (1 - dist / 250) * 0.06;
					t.angle += diff * attraction;
				}

				const newX = head.x + Math.cos(t.angle) * t.speed;
				const newY = head.y + Math.sin(t.angle) * t.speed;

				// Wrap around edges
				const wrappedX = ((newX % w) + w) % w;
				const wrappedY = ((newY % h) + h) % h;

				t.points.pop();
				t.points.unshift({ x: wrappedX, y: wrappedY });

				// Draw trail
				ctx.beginPath();
				for (let i = 0; i < t.points.length - 1; i++) {
					const a = t.points[i];
					const b = t.points[i + 1];

					// Skip if wrapping caused a big jump
					if (Math.abs(a.x - b.x) > w * 0.5 || Math.abs(a.y - b.y) > h * 0.5) continue;

					const alpha = (1 - i / t.points.length) * (t.accent ? 0.2 : 0.1);
					const lineW = (1 - i / t.points.length) * (t.accent ? 1.8 : 1.2);

					ctx.beginPath();
					ctx.moveTo(a.x, a.y);
					ctx.lineTo(b.x, b.y);

					if (t.accent) {
						ctx.strokeStyle = `rgba(${ACCENT[0]},${ACCENT[1]},${ACCENT[2]},${alpha})`;
					} else {
						ctx.strokeStyle = `rgba(28,28,28,${alpha})`;
					}
					ctx.lineWidth = lineW;
					ctx.stroke();
				}

				// Head dot
				ctx.beginPath();
				ctx.arc(wrappedX, wrappedY, t.accent ? 2 : 1.2, 0, Math.PI * 2);
				ctx.fillStyle = t.accent
					? `rgba(${ACCENT[0]},${ACCENT[1]},${ACCENT[2]},0.5)`
					: 'rgba(28,28,28,0.3)';
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
