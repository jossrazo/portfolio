<script lang="ts">
	import { onMount } from 'svelte';

	let canvas: HTMLCanvasElement;

	onMount(() => {
		const ctx = canvas.getContext('2d')!;
		let animId: number;
		let touch = { x: -1000, y: -1000, active: false };
		let w: number, h: number;

		const COUNT = 250;
		const ACCENT = [255, 28, 212];

		type Particle = {
			x: number;
			y: number;
			baseX: number;
			baseY: number;
			angle: number;
			speed: number;
			radius: number;
			phase: number;
			accent: boolean;
		};

		let particles: Particle[] = [];
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
			particles = [];
			for (let i = 0; i < COUNT; i++) {
				const x = Math.random() * w;
				const y = Math.random() * h;
				particles.push({
					x,
					y,
					baseX: x,
					baseY: y,
					angle: Math.random() * Math.PI * 2,
					speed: 0.2 + Math.random() * 0.4,
					radius: 15 + Math.random() * 25,
					phase: Math.random() * Math.PI * 2,
					accent: Math.random() < 0.1
				});
			}
		}

		function draw() {
			ctx.fillStyle = 'rgba(255,255,255,0.03)';
			ctx.fillRect(0, 0, w, h);
			time += 0.008;

			for (const p of particles) {
				// Gentle orbital drift around base position
				const driftX = Math.cos(time * p.speed + p.phase) * p.radius;
				const driftY = Math.sin(time * p.speed * 0.7 + p.phase) * p.radius;

				let targetX = p.baseX + driftX;
				let targetY = p.baseY + driftY;

				// Touch interaction — particles flee and swirl
				if (touch.active) {
					const dx = targetX - touch.x;
					const dy = targetY - touch.y;
					const dist = Math.sqrt(dx * dx + dy * dy);

					if (dist < 160 && dist > 0) {
						const t = 1 - dist / 160;
						const ease = t * t;

						// Repulse
						targetX += (dx / dist) * ease * 40;
						targetY += (dy / dist) * ease * 40;

						// Swirl
						targetX += (-dy / dist) * ease * 20;
						targetY += (dx / dist) * ease * 20;
					}
				}

				p.x += (targetX - p.x) * 0.06;
				p.y += (targetY - p.y) * 0.06;

				// Proximity to touch for color
				let proximity = 0;
				if (touch.active) {
					const dx = p.x - touch.x;
					const dy = p.y - touch.y;
					const dist = Math.sqrt(dx * dx + dy * dy);
					if (dist < 160) proximity = 1 - dist / 160;
				}

				// Draw connecting lines to nearby particles
				for (const q of particles) {
					if (p === q) continue;
					const dx = p.x - q.x;
					const dy = p.y - q.y;
					const dist = Math.sqrt(dx * dx + dy * dy);
					if (dist < 40) {
						const alpha = (1 - dist / 40) * 0.06;
						ctx.beginPath();
						ctx.moveTo(p.x, p.y);
						ctx.lineTo(q.x, q.y);
					ctx.strokeStyle =
						proximity > 0.3
							? `rgba(${ACCENT[0]},${ACCENT[1]},${ACCENT[2]},${alpha * 3})`
							: `rgba(28,28,28,${alpha})`;
						ctx.lineWidth = 0.4;
						ctx.stroke();
					}
				}

				const alpha = 0.15 + proximity * 0.45;
				const size = 1.2 + proximity * 2;

				ctx.beginPath();
				ctx.arc(p.x, p.y, size, 0, Math.PI * 2);
				if (p.accent || proximity > 0.4) {
					ctx.fillStyle = `rgba(${ACCENT[0]},${ACCENT[1]},${ACCENT[2]},${alpha})`;
				} else {
					ctx.fillStyle = `rgba(28,28,28,${alpha})`;
				}
				ctx.fill();
			}

			animId = requestAnimationFrame(draw);
		}

		function handleTouchStart(e: TouchEvent) {
			const rect = canvas.getBoundingClientRect();
			const t = e.touches[0];
			touch.x = t.clientX - rect.left;
			touch.y = t.clientY - rect.top;
			touch.active = true;
		}

		function handleTouchMove(e: TouchEvent) {
			e.preventDefault();
			const rect = canvas.getBoundingClientRect();
			const t = e.touches[0];
			touch.x = t.clientX - rect.left;
			touch.y = t.clientY - rect.top;
		}

		function handleTouchEnd() {
			touch.active = false;
		}

		canvas.addEventListener('touchstart', handleTouchStart, { passive: true });
		canvas.addEventListener('touchmove', handleTouchMove, { passive: false });
		canvas.addEventListener('touchend', handleTouchEnd);
		window.addEventListener('resize', init);

		init();
		draw();

		return () => {
			cancelAnimationFrame(animId);
			canvas.removeEventListener('touchstart', handleTouchStart);
			canvas.removeEventListener('touchmove', handleTouchMove);
			canvas.removeEventListener('touchend', handleTouchEnd);
			window.removeEventListener('resize', init);
		};
	});
</script>

<canvas bind:this={canvas} class="absolute inset-0 h-full w-full"></canvas>
