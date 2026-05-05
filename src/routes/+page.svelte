<script lang="ts">
	import { onMount } from 'svelte';

	const variants = [
		{
			id: 'diagonal',
			label: 'Diagonal sweep',
			description: 'The DeepWiki-style slash moving from the upper-left edge.'
		},
		{
			id: 'opposite',
			label: 'Reverse diagonal',
			description: 'Same geometry, but travelling from the lower-right edge.'
		},
		{
			id: 'circle',
			label: 'Button ripple',
			description: 'A circular reveal expanding from the toggle button.'
		},
		{
			id: 'diamond',
			label: 'Diamond bloom',
			description: 'A hard-edged diamond opening from the page centre.'
		},
		{
			id: 'horizontal',
			label: 'Horizontal wipe',
			description: 'A fast left-to-right plate slide.'
		},
		{
			id: 'vertical',
			label: 'Vertical shutter',
			description: 'A top-to-bottom shutter drop.'
		},
		{
			id: 'split',
			label: 'Centre split',
			description: 'The new theme opens outward from a narrow centre seam.'
		},
		{
			id: 'blinds',
			label: 'Venetian blinds',
			description: 'Alternating horizontal bands scan the new theme in.'
		},
		{
			id: 'bands',
			label: 'Angled bands',
			description: 'A striped diagonal mask sweeps across the viewport.'
		}
	] as const;

	type Variant = (typeof variants)[number]['id'];

	function isVariant(value: string | null): value is Variant {
		return variants.some((item) => item.id === value);
	}

	let theme = $state<'light' | 'dark'>('light');
	let mounted = $state(false);
	let variant = $state<Variant>('diagonal');

	const selectedVariant = $derived(variants.find((item) => item.id === variant) ?? variants[0]);
	const codeSample = $derived(`html[data-theme-transition="${variant}"]::view-transition-new(root) {
  animation-name: reveal-${variant};
}`);

	onMount(() => {
		const savedTheme = localStorage.getItem('theme-swipe-theme');
		const savedVariant = localStorage.getItem('theme-swipe-variant') as Variant | null;
		const prefersDark = window.matchMedia('(prefers-color-scheme: dark)').matches;

		if (isVariant(savedVariant)) variant = savedVariant;
		theme = savedTheme === 'dark' || (!savedTheme && prefersDark) ? 'dark' : 'light';

		document.documentElement.classList.toggle('dark', theme === 'dark');
		document.documentElement.style.colorScheme = theme;
		document.documentElement.dataset.themeTransition = variant;
		mounted = true;
	});

	function applyTheme(nextTheme: 'light' | 'dark') {
		theme = nextTheme;
		localStorage.setItem('theme-swipe-theme', nextTheme);
		document.documentElement.classList.toggle('dark', nextTheme === 'dark');
		document.documentElement.style.colorScheme = nextTheme;
	}

	function saveVariant(nextVariant: Variant) {
		variant = nextVariant;
		localStorage.setItem('theme-swipe-variant', nextVariant);
		document.documentElement.dataset.themeTransition = nextVariant;
	}

	function setTransitionOrigin(event: MouseEvent) {
		const target = event.currentTarget as HTMLElement;
		const rect = target.getBoundingClientRect();
		document.documentElement.style.setProperty('--swipe-x', `${rect.left + rect.width / 2}px`);
		document.documentElement.style.setProperty('--swipe-y', `${rect.top + rect.height / 2}px`);
	}

	function toggleTheme(event: MouseEvent) {
		const nextTheme = theme === 'dark' ? 'light' : 'dark';
		setTransitionOrigin(event);
		document.documentElement.dataset.themeTransition = variant;

		if ('startViewTransition' in document) {
			const transition = document.startViewTransition(() => applyTheme(nextTheme));
			transition.finished.finally(() => {
				document.documentElement.dataset.themeTransition = variant;
			});
			return;
		}

		applyTheme(nextTheme);
	}
</script>

<svelte:head>
	<title>Theme Swipe Lab</title>
	<meta
		name="description"
		content="A SvelteKit lab for testing diagonal and alternate theme swipe animations with the View Transitions API."
	/>
</svelte:head>

<main class="lab-shell">
	<section class="hero" aria-labelledby="page-title">
		<div class="hero-copy">
			<p class="kicker">Theme transition test bench</p>
			<h1 id="page-title">Swipe lab</h1>
			<p class="intro">
				Pick a transition, then flip the theme. Each option uses the same View Transitions API
				snapshot, but swaps the clip or mask animation on the new root view.
			</p>
		</div>

		<div class="control-deck" aria-label="Theme swipe controls">
			<label>
				<span>Swipe variant</span>
				<select value={variant} onchange={(event) => saveVariant(event.currentTarget.value as Variant)}>
					{#each variants as item}
						<option value={item.id}>{item.label}</option>
					{/each}
				</select>
			</label>

			<button
				class="theme-toggle"
				onclick={toggleTheme}
				aria-label={`Switch to ${theme === 'dark' ? 'light' : 'dark'} mode`}
				disabled={!mounted}
			>
				<span class="toggle-glyph" aria-hidden="true">{theme === 'dark' ? '☾' : '☀'}</span>
				<span>{theme === 'dark' ? 'Light mode' : 'Dark mode'}</span>
			</button>
		</div>
	</section>

	<section class="stage" aria-label="Selected transition preview information">
		<div class="stage-card primary-card">
			<span class="card-label">Selected</span>
			<h2>{selectedVariant.label}</h2>
			<p>{selectedVariant.description}</p>
		</div>
		<div class="stage-card sample-card">
			<span class="card-label">CSS hook</span>
			<pre><code>{codeSample}</code></pre>
		</div>
	</section>

	<section class="variant-grid" aria-label="Available swipe variants">
		{#each variants as item, index}
			<button
				class:active={variant === item.id}
				onclick={() => saveVariant(item.id)}
				aria-pressed={variant === item.id}
			>
				<span>{String(index + 1).padStart(2, '0')}</span>
				<strong>{item.label}</strong>
				<small>{item.description}</small>
			</button>
		{/each}
	</section>
</main>

<style>
	:global(:root) {
		--bg: oklch(94% 0.06 98);
		--ink: oklch(20% 0.06 245);
		--muted: oklch(43% 0.05 240);
		--panel: oklch(98% 0.04 98);
		--panel-strong: oklch(89% 0.13 96);
		--line: oklch(26% 0.05 245);
		--accent: oklch(66% 0.22 28);
		--accent-2: oklch(79% 0.17 138);
		--accent-3: oklch(61% 0.16 258);
		--button-text: oklch(98% 0.025 98);
		--shadow: 12px 12px 0 var(--line);
		--swipe-x: 50vw;
		--swipe-y: 50vh;
		background: var(--bg);
		color: var(--ink);
	}

	:global(html.dark) {
		--bg: oklch(16% 0.05 250);
		--ink: oklch(94% 0.04 98);
		--muted: oklch(74% 0.04 250);
		--panel: oklch(22% 0.06 250);
		--panel-strong: oklch(30% 0.08 250);
		--line: oklch(88% 0.035 98);
		--accent: oklch(72% 0.2 33);
		--accent-2: oklch(83% 0.19 138);
		--accent-3: oklch(74% 0.16 258);
		--button-text: oklch(14% 0.05 250);
		--shadow: 12px 12px 0 oklch(4% 0.02 250);
	}

	:global(body) {
		margin: 0;
		min-width: 320px;
		min-height: 100vh;
		background:
			linear-gradient(90deg, color-mix(in oklch, var(--line), transparent 86%) 1px, transparent 1px) 0 0 / 44px 44px,
			linear-gradient(color-mix(in oklch, var(--line), transparent 88%) 1px, transparent 1px) 0 0 / 44px 44px,
			radial-gradient(circle at 85% 15%, color-mix(in oklch, var(--accent-2), transparent 62%), transparent 28rem),
			radial-gradient(circle at 5% 90%, color-mix(in oklch, var(--accent-3), transparent 66%), transparent 26rem),
			var(--bg);
		color: var(--ink);
		font-family: ui-sans-serif, system-ui, -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
	}

	:global(::view-transition-group(root)) {
		animation-duration: 760ms;
		animation-timing-function: cubic-bezier(0.16, 1, 0.3, 1);
	}

	:global(::view-transition-old(root)) {
		z-index: -1;
		animation: none;
	}

	:global(::view-transition-new(root)) {
		animation-name: reveal-diagonal-light;
	}

	:global(.dark::view-transition-new(root)) {
		animation-name: reveal-diagonal-dark;
	}

	:global(html[data-theme-transition='opposite']::view-transition-new(root)) {
		animation-name: reveal-opposite-light;
	}

	:global(html.dark[data-theme-transition='opposite']::view-transition-new(root)) {
		animation-name: reveal-opposite-dark;
	}

	:global(html[data-theme-transition='circle']::view-transition-new(root)) {
		animation-name: reveal-circle;
	}

	:global(html[data-theme-transition='diamond']::view-transition-new(root)) {
		animation-name: reveal-diamond;
	}

	:global(html[data-theme-transition='horizontal']::view-transition-new(root)) {
		animation-name: reveal-horizontal;
	}

	:global(html[data-theme-transition='vertical']::view-transition-new(root)) {
		animation-name: reveal-vertical;
	}

	:global(html[data-theme-transition='split']::view-transition-new(root)) {
		animation-name: reveal-split;
	}

	:global(html[data-theme-transition='blinds']::view-transition-new(root)) {
		animation-name: reveal-blinds;
		mask: linear-gradient(#000 0 0) 0 0 / 100% 8% no-repeat;
	}

	:global(html[data-theme-transition='bands']::view-transition-new(root)) {
		animation-name: reveal-bands;
		mask-image: linear-gradient(115deg, #000 0 42%, transparent 48% 100%);
		mask-size: 240% 240%;
	}

	@keyframes reveal-diagonal-dark {
		0% {
			clip-path: polygon(50% -71%, -50% 71%, -50% 71%, 50% -71%);
			opacity: 0.58;
		}
		100% {
			clip-path: polygon(50% -71%, -50% 71%, 50% 171%, 171% 50%);
			opacity: 1;
		}
	}

	@keyframes reveal-diagonal-light {
		0% {
			clip-path: polygon(171% 50%, 50% 171%, 50% 171%, 171% 50%);
			opacity: 0.58;
		}
		100% {
			clip-path: polygon(171% 50%, 50% 171%, -50% 71%, 50% -71%);
			opacity: 1;
		}
	}

	@keyframes reveal-opposite-dark {
		0% {
			clip-path: polygon(50% 171%, 171% 50%, 171% 50%, 50% 171%);
			opacity: 0.58;
		}
		100% {
			clip-path: polygon(50% 171%, 171% 50%, 50% -71%, -50% 71%);
			opacity: 1;
		}
	}

	@keyframes reveal-opposite-light {
		0% {
			clip-path: polygon(-50% 71%, 50% -71%, 50% -71%, -50% 71%);
			opacity: 0.58;
		}
		100% {
			clip-path: polygon(-50% 71%, 50% -71%, 171% 50%, 50% 171%);
			opacity: 1;
		}
	}

	@keyframes reveal-circle {
		0% {
			clip-path: circle(0 at var(--swipe-x) var(--swipe-y));
			opacity: 0.72;
		}
		100% {
			clip-path: circle(150vmax at var(--swipe-x) var(--swipe-y));
			opacity: 1;
		}
	}

	@keyframes reveal-diamond {
		0% {
			clip-path: polygon(50% 50%, 50% 50%, 50% 50%, 50% 50%);
		}
		100% {
			clip-path: polygon(50% -55%, 155% 50%, 50% 155%, -55% 50%);
		}
	}

	@keyframes reveal-horizontal {
		0% {
			clip-path: inset(0 100% 0 0);
		}
		100% {
			clip-path: inset(0 0 0 0);
		}
	}

	@keyframes reveal-vertical {
		0% {
			clip-path: inset(0 0 100% 0);
		}
		100% {
			clip-path: inset(0 0 0 0);
		}
	}

	@keyframes reveal-split {
		0% {
			clip-path: inset(0 50% 0 50%);
		}
		100% {
			clip-path: inset(0 0 0 0);
		}
	}

	@keyframes reveal-blinds {
		0% {
			mask-size: 100% 0%;
		}
		100% {
			mask-size: 100% 140%;
		}
	}

	@keyframes reveal-bands {
		0% {
			mask-position: 140% 140%;
			opacity: 0.7;
		}
		100% {
			mask-position: 0 0;
			opacity: 1;
		}
	}

	@media (prefers-reduced-motion: reduce) {
		:global(::view-transition-group(root)) {
			animation-duration: 1ms;
		}
	}

	.lab-shell {
		width: min(1180px, calc(100vw - 32px));
		margin: 0 auto;
		padding: clamp(28px, 5vw, 72px) 0;
	}

	.hero {
		display: grid;
		grid-template-columns: minmax(0, 1fr) minmax(280px, 380px);
		gap: clamp(24px, 6vw, 84px);
		align-items: end;
		min-height: min(680px, 78vh);
	}

	.hero-copy {
		position: relative;
	}

	.hero-copy::before {
		position: absolute;
		top: -38px;
		left: -28px;
		z-index: -1;
		width: clamp(110px, 18vw, 220px);
		height: clamp(110px, 18vw, 220px);
		border: 3px solid var(--line);
		background: var(--accent-2);
		content: '';
		transform: rotate(-8deg);
	}

	.kicker,
	.card-label {
		margin: 0 0 18px;
		font-family: ui-monospace, SFMono-Regular, Menlo, monospace;
		font-size: 12px;
		font-weight: 800;
		letter-spacing: 0.14em;
		text-transform: uppercase;
	}

	h1 {
		max-width: 7ch;
		margin: 0;
		font-size: clamp(82px, 17vw, 220px);
		font-weight: 950;
		letter-spacing: -0.1em;
		line-height: 0.72;
		text-transform: uppercase;
	}

	.intro {
		max-width: 700px;
		margin: 32px 0 0;
		color: var(--muted);
		font-size: clamp(18px, 2.3vw, 26px);
		font-weight: 650;
		line-height: 1.25;
	}

	.control-deck {
		display: grid;
		gap: 18px;
		border: 3px solid var(--line);
		background: var(--panel);
		padding: 18px;
		box-shadow: var(--shadow);
		transform: rotate(1.5deg);
	}

	label {
		display: grid;
		gap: 8px;
		font-weight: 850;
	}

	label span {
		font-family: ui-monospace, SFMono-Regular, Menlo, monospace;
		font-size: 12px;
		letter-spacing: 0.12em;
		text-transform: uppercase;
	}

	select,
	.theme-toggle {
		min-height: 56px;
		border: 3px solid var(--line);
		border-radius: 0;
		font: 900 18px/1 ui-sans-serif, system-ui, sans-serif;
	}

	select {
		width: 100%;
		background: var(--bg);
		color: var(--ink);
		padding: 0 14px;
		cursor: pointer;
	}

	.theme-toggle {
		display: inline-flex;
		align-items: center;
		justify-content: center;
		gap: 12px;
		background: var(--accent);
		color: var(--button-text);
		padding: 0 18px;
		box-shadow: 6px 6px 0 var(--line);
		cursor: pointer;
		transition:
			transform 150ms ease,
			box-shadow 150ms ease;
	}

	.theme-toggle:hover {
		box-shadow: 9px 9px 0 var(--line);
		transform: translate(-3px, -3px);
	}

	.theme-toggle:active {
		box-shadow: 3px 3px 0 var(--line);
		transform: translate(3px, 3px);
	}

	.theme-toggle:focus-visible,
	select:focus-visible,
	.variant-grid button:focus-visible {
		outline: 4px solid var(--accent-2);
		outline-offset: 4px;
	}

	.theme-toggle:disabled {
		cursor: wait;
		opacity: 0.65;
	}

	.toggle-glyph {
		font-size: 24px;
		line-height: 1;
	}

	.stage {
		display: grid;
		grid-template-columns: 0.8fr 1.2fr;
		gap: 22px;
		margin-top: clamp(34px, 7vw, 84px);
	}

	.stage-card {
		border: 3px solid var(--line);
		background: var(--panel);
		padding: clamp(22px, 4vw, 36px);
		box-shadow: var(--shadow);
	}

	.primary-card {
		background: var(--panel-strong);
	}

	h2 {
		margin: 0;
		font-size: clamp(42px, 7vw, 84px);
		font-weight: 950;
		letter-spacing: -0.08em;
		line-height: 0.85;
	}

	.stage-card p {
		margin: 22px 0 0;
		color: var(--muted);
		font-size: 19px;
		font-weight: 700;
		line-height: 1.35;
	}

	pre {
		overflow: auto;
		margin: 18px 0 0;
		background: var(--ink);
		color: var(--bg);
		padding: 20px;
		font-size: 14px;
		line-height: 1.6;
	}

	code {
		font-family: ui-monospace, SFMono-Regular, Menlo, monospace;
	}

	.variant-grid {
		display: grid;
		grid-template-columns: repeat(3, minmax(0, 1fr));
		gap: 14px;
		margin-top: 48px;
	}

	.variant-grid button {
		display: grid;
		gap: 14px;
		min-height: 170px;
		border: 3px solid var(--line);
		background: var(--panel);
		color: var(--ink);
		padding: 18px;
		text-align: left;
		cursor: pointer;
		transition:
			transform 150ms ease,
			background 150ms ease;
	}

	.variant-grid button:hover,
	.variant-grid button.active {
		background: var(--accent-2);
		transform: translateY(-4px);
	}

	.variant-grid span {
		font-family: ui-monospace, SFMono-Regular, Menlo, monospace;
		font-size: 12px;
		font-weight: 900;
	}

	.variant-grid strong {
		font-size: 24px;
		font-weight: 950;
		letter-spacing: -0.05em;
		line-height: 0.95;
	}

	.variant-grid small {
		color: var(--muted);
		font-size: 14px;
		font-weight: 700;
		line-height: 1.35;
	}

	@media (max-width: 900px) {
		.hero,
		.stage,
		.variant-grid {
			grid-template-columns: 1fr;
		}

		.hero {
			align-items: start;
			min-height: auto;
		}

		.control-deck {
			transform: none;
		}
	}
</style>
