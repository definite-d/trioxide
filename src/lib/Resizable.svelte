<script lang="ts">
	import { onMount, type Snippet } from 'svelte';
	let {
		w = $bindable(),
		h = $bindable(),
		r = $bindable(),
		x = $bindable(),
		y = $bindable(),
		lockAspect = false,
		fromCenter = false,
		defaultEdgeColor = 'var(--highlight-10)',
		defaultEdgeHitSize = '10px',
		defaultEdgeSize = '1px',
		defaultCornerResizeColor = 'white',
		defaultCornerResizeSize = '8px',
		defaultCornerResizeBorder = '1px solid var(--highlight-10)',
		defaultRotateCornerHitSize = '16px',
		Handle,
		getCursor,
		children
	}: {
		x: number;
		y: number;
		w: number;
		h: number;
		r: number;
		lockAspect?: boolean;
		fromCenter?: boolean;
		children?: Snippet;
		Handle?: Snippet<[Omit<Action, 'mv'>, HandleBindings]>;
		defaultEdgeColor?: string;
		defaultEdgeSize?: string;
		defaultEdgeHitSize?: string;
		defaultCornerResizeColor?: string;
		defaultCornerResizeSize?: string;
		defaultCornerResizeBorder?: string;
		defaultRotateCornerHitSize?: string;
		getCursor?: (action: Action | undefined, r: number, active: boolean) => string | undefined;
	} = $props();

	let scale = $state(1);
	let el: HTMLElement;
	let scaleEl: HTMLElement;
	type Action =
		| 'el'
		| 'er'
		| 'et'
		| 'eb'
		| 'stl'
		| 'sbr'
		| 'sbl'
		| 'str'
		| 'rtl'
		| 'rtr'
		| 'rbr'
		| 'rbl'
		| 'mv';
	let action: Action | undefined = $state(undefined);
	let mp = [0, 0];

	const startAction = (newAction: typeof action) => (e: MouseEvent) => {
		e.preventDefault();
		e.stopPropagation();
		mp[0] = e.clientX;
		mp[1] = e.clientY;
		action = newAction;
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
		const isScale = action.startsWith('e') || action.startsWith('s');
		const ratio = w / h;

		let dw = dx * cos + dy * sin;
		let dh = dy * cos - dx * sin;

		let dxp = 0;
		let dyp = 0;

		if (action === 'eb' || action === 'sbr' || action === 'sbl') {
			if (fromCenter) {
				dyp -= dh;
				h += 2 * dh;
			} else {
				h += dh;
				dxp += (-sin * dh) / 2;
				dyp += (dh * (cos - 1)) / 2;
			}
		}

		if (action === 'et' || action === 'str' || action === 'stl') {
			if (fromCenter) {
				dyp += dh;
				h += -2 * dh;
			} else {
				dxp += (-sin * dh) / 2;
				dyp += (dh * (1 + cos)) / 2;
				h += -dh;
			}
		}

		if (action === 'er' || action === 'sbr' || action === 'str') {
			if (fromCenter) {
				dxp -= dw;
				w += 2 * dw;
			} else {
				w += dw;
				dxp += (dw * (cos - 1)) / 2;
				dyp += (dw * sin) / 2;
			}
		}

		if (action === 'el' || action === 'sbl' || action === 'stl') {
			if (fromCenter) {
				dxp += dw;
				w += -2 * dw;
			} else {
				w += -dw;
				dxp += (dw * (1 + cos)) / 2;
				dyp += (dw * sin) / 2;
			}
		}

		if (w < 0) {
			w = -w;
			dxp -= w;

			action =
				(
					{
						el: 'er',
						er: 'el',
						stl: 'str',
						str: 'stl',
						sbl: 'sbr',
						sbr: 'sbl'
					} as Record<Action, Action>
				)[action] ?? action;
		}

		if (h < 0) {
			h = -h;
			dyp -= h;

			action =
				(
					{
						et: 'eb',
						eb: 'et',
						stl: 'sbl',
						sbl: 'stl',
						str: 'sbr',
						sbr: 'str'
					} as Record<Action, Action>
				)[action] ?? action;
		}

		if (action[0] === 'r') {
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

		if (action == 'mv') {
			dxp += dx;
			dyp += dy;
		}

		x += dxp;
		y += dyp;
	};

	const mouseMove = (e: MouseEvent) => {
		actionMove(e);
		mp[0] = e.clientX;
		mp[1] = e.clientY;
	};

	const getCursorEdges = (v: number) => {
		return `url("data:image/svg+xml;charset=utf-8,${encodeURIComponent(`
		<svg width="24" height="24" viewBox="0 0 192 192" fill="none" xmlns="http://www.w3.org/2000/svg">
			<g transform="rotate(${v}, 96, 96)"> 
			<path fill-rule="evenodd" clip-rule="evenodd" d="M100.509 136.593C100.004 136.593 99.5943 136.184 99.5943 135.678V55.2217C99.5943 54.7167 100.004 54.3074 100.509 54.3074H125.121C125.887 54.3074 126.313 53.4207 125.835 52.822L91.1654 9.48556C90.7994 9.02805 90.1036 9.02805 89.7376 9.48556L55.0684 52.822C54.5895 53.4207 55.0157 54.3074 55.7824 54.3074H80.3943C80.8993 54.3074 81.3086 54.7167 81.3086 55.2217V135.678C81.3086 136.184 80.8993 136.593 80.3943 136.593H55.7824C55.0157 136.593 54.5895 137.48 55.0684 138.078L89.7376 181.415C90.1036 181.873 90.7994 181.873 91.1654 181.415L125.835 138.078C126.313 137.48 125.887 136.593 125.121 136.593H100.509Z" fill="black"/>
			<path d="M86.6032 6.14671C88.7274 4.04633 92.1754 4.04633 94.2996 6.14671L94.7371 6.62886L129.407 49.9681C132.277 53.5598 129.72 58.8785 125.121 58.8788H104.166V132.021H125.121C129.72 132.022 132.277 137.341 129.407 140.933L94.7371 184.272C92.5411 187.017 88.3618 187.017 86.1657 184.272L51.4961 140.933C48.6258 137.341 51.1833 132.022 55.7818 132.021H76.7371V58.8788H55.7818C51.1833 58.8785 48.6258 53.5598 51.4961 49.9681L86.1657 6.62886L86.6032 6.14671Z" stroke="white" stroke-width="8"/>
			</g>
		</svg>
	`)}") 12 12, default`;
	};

	const getCursorRotate = (v: number) => {
		return `url("data:image/svg+xml;charset=utf-8,${encodeURIComponent(`
			<svg width="24" height="24" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
				<g transform="rotate(${v}, 12, 12)"> 
					<path d="M4.8 13.2C4.8 12.0969 5.01727 11.0046 5.43941 9.98546C5.86155 8.96632 6.48029 8.04032 7.2603 7.2603C8.04031 6.48029 8.96632 5.86155 9.98546 5.43941C11.0046 5.01727 12.0969 4.8 13.2 4.8H16.8L16.8 0L24 7.2L16.8 14.4V9.6H13.2C12.7272 9.6 12.2591 9.69312 11.8223 9.87403C11.3856 10.055 10.9887 10.3201 10.6544 10.6544C10.3201 10.9887 10.055 11.3856 9.87403 11.8223C9.69312 12.2591 9.6 12.7272 9.6 13.2V16.8H14.4L7.2 24L0 16.8L4.8 16.8L4.8 13.2Z" fill="white"/>
					<path d="M8.4 13.2V18H11.4L7.2 22.2L3 18L6 18V13.2C6 12.2545 6.18623 11.3182 6.54807 10.4447C6.9099 9.57113 7.44025 8.77741 8.10883 8.10883C8.77741 7.44025 9.57113 6.9099 10.4447 6.54806C11.3182 6.18623 12.2545 5.99999 13.2 5.99999H18L18 2.99999L22.2 7.19999L18 11.4V8.39999H13.2C12.5697 8.39999 11.9455 8.52415 11.3631 8.76537C10.7808 9.0066 10.2516 9.36016 9.80589 9.80588C9.36016 10.2516 9.0066 10.7808 8.76538 11.3631C8.52415 11.9455 8.4 12.5697 8.4 13.2Z" fill="black"/>
				</g>
			</svg>
		`)}") 12 12, default`;
	};

	const cursorOffset: Record<Action, number> = {
		rtl: 0,
		rtr: 90,
		rbr: 180,
		rbl: -90,
		eb: 0,
		et: 0,
		el: 90,
		er: 90,
		stl: -45,
		str: 45,
		sbr: -45,
		sbl: 45,
		mv: 0
	};

	const getCursorDefault = (a?: Action) => {
		const active = a == action;
		if (getCursor) return getCursor(a, r, active);
		if (!a) return undefined;
		const offset = cursorOffset[a];
		if (a[0] == 'r') return getCursorRotate(r + offset);
		if (a == 'mv' && active) return 'grabbing';
		if (a != 'mv') return getCursorEdges(r + offset);
		return 'grab';
	};

	const setCursor = (c: string = '') => {
		document.body.style.cursor = c;
	};

	const handleEnter = (a: Action) => (e: MouseEvent) => {
		if (action != undefined) return;
		e.stopImmediatePropagation();
		setCursor(getCursorDefault(a));
	};

	const handleLeave = () => {
		if (action) return;
		setCursor(undefined);
	};

	const handleUp = (a: Action) => (e: MouseEvent) => {
		e.stopPropagation();
		action = undefined;
		skipEffect = true;
		setCursor(getCursorDefault(a));
	};

	let skipEffect = false;

	$effect(() => {
		r;
		action;
		if (skipEffect) {
			skipEffect = false;
		} else {
			setCursor(getCursorDefault(action));
		}
	});

	const getHandleBindings = (action: Action) => ({
		onmousedown: startAction(action),
		onmouseover: handleEnter(action),
		onmouseleave: handleLeave,
		onmouseup: handleUp(action)
	});

	type HandleBindings = ReturnType<typeof getHandleBindings>;

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

{#snippet EdgeResizeDefault(action: Action, binding: HandleBindings)}
	{@const side = action![1]}
	{@const { transform, width, height, top, left, bottom, right, edgeWidth, edgeHeight } =
		side == 't' || side == 'b'
			? {
					transform: 'scaleY(calc(1 / var(--s)))',
					width: '100%',
					height: defaultEdgeHitSize,
					edgeWidth: '100%',
					edgeHeight: defaultEdgeSize,
					...(side == 't'
						? {
								top: `calc(-${defaultEdgeHitSize}/2)`,
								left: '0',
								bottom: '',
								right: ''
							}
						: {
								top: '',
								bottom: `calc(-${defaultEdgeHitSize}/2)`,
								left: '0',
								right: ''
							})
				}
			: {
					transform: 'scaleX(calc(1 / var(--s)))',
					width: defaultEdgeHitSize,
					height: '100%',
					edgeWidth: defaultEdgeSize,
					edgeHeight: '100%',
					...(side == 'l'
						? {
								top: '0',
								left: `calc(-${defaultEdgeHitSize}/2)`,
								bottom: '',
								right: ''
							}
						: {
								top: '0',
								bottom: '',
								left: '',
								right: `calc(-${defaultEdgeHitSize}/2)`
							})
				}}
	<button
		style:--s={scale}
		style:cursor="inherit"
		style:position="absolute"
		style:top
		style:left
		style:bottom
		style:right
		style:height
		style:width
		style:transform
		style:display="flex"
		style:justify-content="center"
		style:align-items="center"
		style:background-color="red"
		{...binding}
	>
		<div
			style:width={edgeWidth}
			style:height={edgeHeight}
			style:background-color={defaultEdgeColor}
		></div>
	</button>
{/snippet}

{#snippet CornerRotateDefault(corner: Action, binding: HandleBindings)}
	{@const [_, y, x] = corner!}
	{@const size = `calc(-${defaultRotateCornerHitSize}/2 - ${defaultRotateCornerHitSize}/(2 * var(--s)))`}
	{@const { top, left, bottom, right } = {
		top: y == 't' ? size : '',
		left: x == 'l' ? size : '',
		bottom: y == 'b' ? size : '',
		right: x == 'r' ? size : ''
	}}
	<button
		style:position="absolute"
		style:cursor="inherit"
		style:scale="calc(1/var(--s))"
		style:width={defaultRotateCornerHitSize}
		style:height={defaultRotateCornerHitSize}
		style:top
		style:left
		style:bottom
		style:right
		{...binding}
	></button>
{/snippet}

{#snippet CornerResizeDefault(corner: Action, binding: HandleBindings)}
	{@const [_, y, x] = corner!}
	{@const size = `calc(-${defaultCornerResizeSize}/2)`}
	{@const { top, left, bottom, right } = {
		top: y == 't' ? size : '',
		left: x == 'l' ? size : '',
		bottom: y == 'b' ? size : '',
		right: x == 'r' ? size : ''
	}}
	<button
		style:--s={scale}
		style:position="absolute"
		style:cursor="inherit"
		style:top
		style:left
		style:bottom
		style:right
		style:width={defaultCornerResizeSize}
		style:height={defaultCornerResizeSize}
		style:scale={1 / scale}
		style:border={defaultCornerResizeBorder}
		style:background-color={defaultCornerResizeColor}
		{...binding}
	></button>
{/snippet}

{#snippet HandleDefault(action: Action)}
	{@const type = action[0]}
	{@const binding = getHandleBindings(action)}
	{#if Handle}
		{@render Handle(action, binding)}
	{:else if type == 'r'}
		{@render CornerRotateDefault(action, binding)}
	{:else if type == 'e'}
		{@render EdgeResizeDefault(action, binding)}
	{:else if type == 's'}
		{@render CornerResizeDefault(action, binding)}
	{/if}
{/snippet}

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
	onmousedown={startAction('mv')}
	onmouseover={handleEnter('mv')}
	onmouseout={handleLeave}
	onmouseup={handleUp('mv')}
	style:background-color="green"
	{...{}}
>
	<div
		style:position="absolute"
		style:left="0"
		style:top="0"
		style:width="100%"
		style:height="100%"
	>
		{@render children?.()}
	</div>

	{#each ['et', 'er', 'eb', 'el', 'rtl', 'rtr', 'rbr', 'rbl', 'stl', 'str', 'sbr', 'sbl'] as action}
		{@render HandleDefault(action as Action)}
	{/each}

	<div
		style="width:1px;height:1px;position:absolute;pointer-events:none;"
		style:rotate="{-r}deg"
		bind:this={scaleEl}
	></div>
</div>
