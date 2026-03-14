<script lang="ts">
	import { onMount } from 'svelte';

	let canvas: HTMLCanvasElement;

	onMount(() => {
		const ctx = canvas.getContext('2d')!;
		let animId: number;
		let mouse = { x: -1000, y: -1000 };
		let w: number, h: number;

		const COUNT = 80;
		const CONNECT_DIST = 140;
		const MOUSE_DIST = 180;
		const ACCENT = [255, 28, 212];

		type Node = { x: number; y: number; vx: number; vy: number };
		let nodes: Node[] = [];

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
			nodes = [];
			for (let i = 0; i < COUNT; i++) {
				nodes.push({
					x: Math.random() * w,
					y: Math.random() * h,
					vx: (Math.random() - 0.5) * 0.4,
					vy: (Math.random() - 0.5) * 0.4
				});
			}
		}

		function draw() {
			ctx.clearRect(0, 0, w, h);

			for (const n of nodes) {
				n.x += n.vx;
				n.y += n.vy;
				if (n.x < 0 || n.x > w) n.vx *= -1;
				if (n.y < 0 || n.y > h) n.vy *= -1;
			}

			// Lines between nodes
			for (let i = 0; i < nodes.length; i++) {
				for (let j = i + 1; j < nodes.length; j++) {
					const dx = nodes[i].x - nodes[j].x;
					const dy = nodes[i].y - nodes[j].y;
					const dist = Math.sqrt(dx * dx + dy * dy);
					if (dist < CONNECT_DIST) {
						const alpha = (1 - dist / CONNECT_DIST) * 0.15;
						ctx.beginPath();
						ctx.moveTo(nodes[i].x, nodes[i].y);
						ctx.lineTo(nodes[j].x, nodes[j].y);
						ctx.strokeStyle = `rgba(28,28,28,${alpha})`;
						ctx.lineWidth = 0.6;
						ctx.stroke();
					}
				}
			}

			// Lines from mouse to nearby nodes
			for (const n of nodes) {
				const dx = n.x - mouse.x;
				const dy = n.y - mouse.y;
				const dist = Math.sqrt(dx * dx + dy * dy);
				if (dist < MOUSE_DIST) {
					const alpha = (1 - dist / MOUSE_DIST) * 0.35;
					ctx.beginPath();
					ctx.moveTo(mouse.x, mouse.y);
					ctx.lineTo(n.x, n.y);
					ctx.strokeStyle = `rgba(${ACCENT[0]},${ACCENT[1]},${ACCENT[2]},${alpha})`;
					ctx.lineWidth = 0.8;
					ctx.stroke();
				}
			}

			// Draw nodes
			for (const n of nodes) {
				const dx = n.x - mouse.x;
				const dy = n.y - mouse.y;
				const dist = Math.sqrt(dx * dx + dy * dy);
				const nearMouse = dist < MOUSE_DIST;
				const r = nearMouse ? 2 + (1 - dist / MOUSE_DIST) * 2 : 1.5;

				ctx.beginPath();
				ctx.arc(n.x, n.y, r, 0, Math.PI * 2);
				ctx.fillStyle = nearMouse
					? `rgba(${ACCENT[0]},${ACCENT[1]},${ACCENT[2]},0.6)`
					: 'rgba(28,28,28,0.25)';
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
