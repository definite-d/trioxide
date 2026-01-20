<script lang="ts">
	import { onMount, type Snippet } from 'svelte';
	const { children }: { children?: Snippet } = $props();

	let w = $state(100);
	let h = $state(100);
	let r = $state(0);
	let x = $state(50);
	let y = $state(50);

	let scale = $state(1);
	let el: HTMLElement;
	let scaleEl: HTMLElement;
	let cursorEl: SVGSVGElement = $state(null!);
	let action: 'wl' | 'wr' | 'ht' | 'hb' | 'tl' | 'br' | 'bl' | 'tr' | 'rot' | undefined = undefined;
	let mp = [0, 0];
	let pp: [number, number] = $state([0, 0]);

	const startAction = (dir: typeof action) => (e: MouseEvent) => {
		e.preventDefault();
		e.stopPropagation();
		mp[0] = e.clientX;
		mp[1] = e.clientY;
		action = dir;
	};

	const endAction = () => {
		action = undefined;
	};

	const actionMove = (e: MouseEvent) => {
		if (!action) return;

		const dx = (e.clientX - mp[0]) / scale;
		const dy = (e.clientY - mp[1]) / scale;

		const rad = (r * Math.PI) / 180;
		const sin = Math.sin(rad);
		const cos = Math.cos(rad);

		const dw = dx * cos + dy * sin;
		const dh = dy * cos - dx * sin;

		let dxp = 0;
		let dyp = 0;

		if (action === 'hb' || action === 'br' || action === 'bl') {
			h += dh;
			dxp += (-sin * dh) / 2;
			dyp += (dh * (cos - 1)) / 2;
		}

		if (action === 'ht' || action === 'tr' || action === 'tl') {
			h -= dh;
			dxp += (-sin * dh) / 2;
			dyp += (dh * (1 + cos)) / 2;
		}

		if (action === 'wr' || action === 'br' || action === 'tr') {
			w += dw;
			dxp += (dw * (cos - 1)) / 2;
			dyp += (dw * sin) / 2;
		}

		if (action === 'wl' || action === 'bl' || action === 'tl') {
			w -= dw;
			dxp += (dw * (1 + cos)) / 2;
			dyp += (dw * sin) / 2;
		}

		if (action === 'rot') {
			const rect = el.getBoundingClientRect();
			const cx = rect.x + rect.width / 2;
			const cy = rect.y + rect.height / 2;

			const v1x = mp[0] - cx;
			const v1y = mp[1] - cy;
			const v2x = e.clientX - cx;
			const v2y = e.clientY - cy;

			r = (r + (Math.atan2(v1x * v2y - v1y * v2x, v1x * v2x + v1y * v2y) * 180) / Math.PI) % 360;
			return;
		}

		if (w < 0) {
			w = -w;

			if (action === 'wl') action = 'wr';
			else if (action === 'wr') action = 'wl';
			else if (action === 'tl') action = 'tr';
			else if (action === 'tr') action = 'tl';
			else if (action === 'bl') action = 'br';
			else if (action === 'br') action = 'bl';
		}

		// vertical flip
		if (h < 0) {
			h = -h;

			// swap vertical part of corner
			if (action === 'ht') action = 'hb';
			else if (action === 'hb') action = 'ht';
			else if (action === 'tl') action = 'bl';
			else if (action === 'bl') action = 'tl';
			else if (action === 'tr') action = 'br';
			else if (action === 'br') action = 'tr';
		}

		x += dxp;
		y += dyp;
	};

	const mouseMove = (e: MouseEvent) => {
		actionMove(e);
		movePointer(e);
		mp[0] = e.clientX;
		mp[1] = e.clientY;
	};

	const movePointer = (e: MouseEvent) => {
		// const rc = scaleEl.getBoundingClientRect();
		// const c = cursorEl.getBoundingClientRect();
		// const rad = (r * Math.PI) / 180;
		// pp[0] = e.clientX - rc.x - c.width / 2;
		// pp[1] = e.clientY - rc.y - c.height / 2;
	};

	onMount(() => {
		const updateZoom = () => {
			const rect = scaleEl.getBoundingClientRect();
			scale = rect.width / scaleEl.offsetWidth || 1;
			animationFrame = requestAnimationFrame(updateZoom);
		};
		let animationFrame = requestAnimationFrame(updateZoom);
		return () => {
			cancelAnimationFrame(animationFrame);
		};
	});
</script>

<svelte:window onmousemove={mouseMove} onmouseup={endAction} />

<div
	style:position="relative"
	style:transform-origin="center"
	style:width="{w}px"
	style:height="{h}px"
	style:--r="{r}deg"
	style:--x="{x}px"
	style:--y="{y}px"
	style:transform="translate(var(--x), var(--y)) rotate(var(--r))"
	style:--s={scale}
	bind:this={el}
	{...{}}
>
	<button
		style:--s={scale}
		class="resizer-w absolute top-0 left-[-0.5px]"
		onmousedown={startAction('wl')}
		aria-label="l"
	></button>
	<button
		style:--s={scale}
		class=" resizer-w absolute top-0 right-[-0.5px]"
		onmousedown={startAction('wr')}
		aria-label="l"
	></button>
	<button
		style:--s={scale}
		class=" resizer-h absolute top-[-0.5px] left-0"
		onmousedown={startAction('ht')}
		aria-label="l"
	></button>
	<button
		style:--s={scale}
		class=" resizer-h absolute bottom-[-0.5px] left-0"
		onmousedown={startAction('hb')}
		aria-label="l"
	></button>

	<button
		style:--s={scale}
		class=" absolute -top-4 -left-4 h-4 w-4 scale-[calc(1/var(--s))] cursor-nwse-resize border bg-white"
		aria-label="l"
		onmousedown={startAction('rot')}
	></button>

	<button
		style:--s={scale}
		class=" absolute -top-1 -left-1 h-2 w-2 scale-[calc(1/var(--s))] cursor-nwse-resize border border-highlight-10 bg-white"
		aria-label="l"
		onmousedown={startAction('tl')}
	></button>

	<button
		style:--s={scale}
		class=" absolute -top-1 -right-1 h-2 w-2 scale-[calc(1/var(--s))] cursor-nesw-resize border border-highlight-10 bg-white"
		aria-label="l"
		onmousedown={startAction('tr')}
	></button>
	<button
		style:--s={scale}
		onmousedown={startAction('bl')}
		class=" absolute -bottom-1 -left-1 h-2 w-2 scale-[calc(1/var(--s))] cursor-nesw-resize border border-highlight-10 bg-white"
		aria-label="l"
	></button>
	<button
		style:--s={scale}
		class=" absolute -right-1 -bottom-1 h-2 w-2 scale-[calc(1/var(--s))] cursor-nwse-resize border border-highlight-10 bg-white"
		onmousedown={startAction('br')}
		aria-label="l"
	></button>
	<div
		style="width:1px;height:1px;position:absolute;pointer-events:none;"
		style:rotate="{-r}deg"
		bind:this={scaleEl}
	></div>
	<!-- {@render Cursor(pp[0], pp[1])} -->
	<div>
		{h}
		{@render children?.()}
	</div>
</div>

{#snippet Cursor(x: number, y: number, a = 0)}
	<svg
		width="10"
		height="21"
		style:scale={1 / scale}
		style:position="absolute"
		style:left="0"
		style:top="0"
		style:transform="translate(var(--x), var(--y))"
		style:transform-origin="center center"
		style:pointer-events="none"
		style:--x="{x}px"
		style:--y="{y}px"
		viewBox="0 0 10 21"
		fill="none"
		xmlns="http://www.w3.org/2000/svg"
		bind:this={cursorEl}
	>
		<path
			fill-rule="evenodd"
			clip-rule="evenodd"
			d="M5.99404 14.9399C5.93881 14.9399 5.89404 14.8952 5.89404 14.8399V6.03994C5.89404 5.98471 5.93881 5.93994 5.99404 5.93994H8.68598C8.76983 5.93994 8.81645 5.84295 8.76407 5.77747L4.97213 1.03755C4.9321 0.987509 4.85599 0.987509 4.81596 1.03755L1.02402 5.77747C0.971638 5.84295 1.01825 5.93994 1.10211 5.93994H3.79404C3.84927 5.93994 3.89404 5.98471 3.89404 6.03994V14.8399C3.89404 14.8952 3.84927 14.9399 3.79404 14.9399H1.10211C1.01825 14.9399 0.971638 15.0369 1.02402 15.1024L4.81596 19.8423C4.85599 19.8924 4.9321 19.8924 4.97213 19.8423L8.76407 15.1024C8.81645 15.0369 8.76983 14.9399 8.68598 14.9399H5.99404Z"
			fill="black"
		/>
		<path
			d="M4.47314 0.672363C4.70548 0.442634 5.0826 0.442633 5.31494 0.672363L5.36279 0.725098L9.15479 5.46533C9.46872 5.85817 9.189 6.4399 8.68604 6.43994H6.39404V14.4399H8.68604C9.189 14.44 9.46872 15.0217 9.15479 15.4146L5.36279 20.1548C5.1226 20.455 4.66549 20.455 4.42529 20.1548L0.633301 15.4146C0.319365 15.0217 0.599091 14.44 1.10205 14.4399H3.39404V6.43994H1.10205C0.599091 6.4399 0.319366 5.85817 0.633301 5.46533L4.42529 0.725098L4.47314 0.672363Z"
			stroke="white"
			stroke-opacity="0.8"
		/>
	</svg>
{/snippet}

<style lang="postcss">
	.resizer-w {
		position: absolute;
		background-color: var(--highlight-10);
		height: 100%;
		width: 1px;
		transform: scaleX(calc(1 / var(--s)));
		/* cursor: none; */
		&::after {
			content: '';
			top: 0;
			left: -2.5px;
			position: absolute;
			display: block;
			width: calc(100% + 5px);
			height: 100%;
		}
	}

	.resizer-h {
		position: absolute;
		background-color: var(--highlight-10);
		width: 100%;
		height: 1px;
		transform: scaleY(calc(1 / var(--s)));
		/* cursor: none; */
		&::after {
			content: '';
			left: 0;
			top: -2.5px;
			position: absolute;
			display: block;
			height: calc(100% + 5px);
			width: 100%;
		}
	}
</style>
