<script lang="ts">
	import { onMount } from 'svelte';

	let canvas: HTMLCanvasElement;

	function createNoise() {
		const perm = new Uint8Array(512);
		const grad = new Float32Array(512);
		for (let i = 0; i < 256; i++) {
			perm[i] = i;
		}
		for (let i = 255; i > 0; i--) {
			const j = Math.floor(Math.random() * (i + 1));
			[perm[i], perm[j]] = [perm[j], perm[i]];
		}
		for (let i = 0; i < 256; i++) {
			perm[i + 256] = perm[i];
			grad[i] = (Math.random() - 0.5) * 2;
			grad[i + 256] = grad[i];
		}

		function fade(t: number) {
			return t * t * t * (t * (t * 6 - 15) + 10);
		}
		function lerp(a: number, b: number, t: number) {
			return a + t * (b - a);
		}

		return function noise2d(x: number, y: number): number {
			const xi = Math.floor(x) & 255;
			const yi = Math.floor(y) & 255;
			const xf = x - Math.floor(x);
			const yf = y - Math.floor(y);
			const u = fade(xf);
			const v = fade(yf);

			const aa = perm[perm[xi] + yi];
			const ab = perm[perm[xi] + yi + 1];
			const ba = perm[perm[xi + 1] + yi];
			const bb = perm[perm[xi + 1] + yi + 1];

			return lerp(
				lerp(grad[aa] * xf + grad[aa] * yf, grad[ba] * (xf - 1) + grad[ba] * yf, u),
				lerp(
					grad[ab] * xf + grad[ab] * (yf - 1),
					grad[bb] * (xf - 1) + grad[bb] * (yf - 1),
					u
				),
				v
			);
		};
	}

	onMount(() => {
		const ctx = canvas.getContext('2d')!;
		const noise = createNoise();
		let animId: number;
		let mouse = { x: -1000, y: -1000, vx: 0, vy: 0, prevX: -1000, prevY: -1000 };

		const PARTICLE_COUNT = 400;
		const NOISE_SCALE = 0.003;
		const SPEED = 0.8;
		const MOUSE_RADIUS = 180;

		let w: number, h: number;
		let particles: {
			x: number;
			y: number;
			age: number;
			maxAge: number;
			radius: number;
			accent: boolean;
		}[] = [];
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

		function spawnParticle() {
			return {
				x: Math.random() * w,
				y: Math.random() * h,
				age: 0,
				maxAge: 80 + Math.random() * 140,
				radius: 1 + Math.random() * 2.5,
				accent: Math.random() < 0.08
			};
		}

		function init() {
			resize();
			particles = [];
			for (let i = 0; i < PARTICLE_COUNT; i++) {
				particles.push(spawnParticle());
			}
			ctx.fillStyle = '#ffffff';
			ctx.fillRect(0, 0, w, h);
		}

		function draw() {
			ctx.fillStyle = 'rgba(255, 255, 255, 0.1)';
			ctx.fillRect(0, 0, w, h);

			time += 0.002;

			if (mouse.prevX > -500) {
				mouse.vx = mouse.vx * 0.7 + (mouse.x - mouse.prevX) * 0.3;
				mouse.vy = mouse.vy * 0.7 + (mouse.y - mouse.prevY) * 0.3;
			}
			mouse.prevX = mouse.x;
			mouse.prevY = mouse.y;

			const mouseSpeed = Math.sqrt(mouse.vx * mouse.vx + mouse.vy * mouse.vy);

			for (let i = 0; i < particles.length; i++) {
				const p = particles[i];

				const angle = noise(p.x * NOISE_SCALE, p.y * NOISE_SCALE + time) * Math.PI * 4;

				const dx = p.x - mouse.x;
				const dy = p.y - mouse.y;
				const dist = Math.sqrt(dx * dx + dy * dy);

				let mx = 0;
				let my = 0;
				let nearMouse = false;

				if (dist < MOUSE_RADIUS && dist > 0) {
					const t = 1 - dist / MOUSE_RADIUS;
					const ease = t * t;
					nearMouse = t > 0.3;

					const repulse = ease * 2.5;
					mx += (dx / dist) * repulse;
					my += (dy / dist) * repulse;

					const swirlStrength = ease * 1.5;
					mx += (-dy / dist) * swirlStrength;
					my += (dx / dist) * swirlStrength;

					const turbulence = ease * Math.min(mouseSpeed * 0.12, 2);
					mx += mouse.vx * turbulence * 0.06;
					my += mouse.vy * turbulence * 0.06;
				}

				const vx = Math.cos(angle) * SPEED + mx;
				const vy = Math.sin(angle) * SPEED + my;

				p.x += vx;
				p.y += vy;
				p.age++;

				const lifeFraction = p.age / p.maxAge;
				const alpha = Math.sin(lifeFraction * Math.PI);

				const r = p.radius * (0.6 + alpha * 0.4);

				ctx.beginPath();
				ctx.arc(p.x, p.y, r, 0, Math.PI * 2);

				if (nearMouse || p.accent) {
					const a = (nearMouse ? alpha * 0.3 : alpha * 0.18);
					ctx.fillStyle = `rgba(255, 28, 212, ${a})`;
				} else {
					ctx.fillStyle = `rgba(28, 28, 28, ${alpha * 0.1})`;
				}
				ctx.fill();

				if (p.age > p.maxAge || p.x < -10 || p.x > w + 10 || p.y < -10 || p.y > h + 10) {
					particles[i] = spawnParticle();
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
			mouse.prevX = -1000;
			mouse.prevY = -1000;
			mouse.vx = 0;
			mouse.vy = 0;
		}

		canvas.addEventListener('mousemove', handleMouseMove);
		canvas.addEventListener('mouseleave', handleMouseLeave);
		window.addEventListener('resize', () => {
			init();
		});

		init();
		draw();

		return () => {
			cancelAnimationFrame(animId);
			canvas.removeEventListener('mousemove', handleMouseMove);
			canvas.removeEventListener('mouseleave', handleMouseLeave);
		};
	});
</script>

<canvas bind:this={canvas} class="absolute inset-0 h-full w-full"></canvas>
