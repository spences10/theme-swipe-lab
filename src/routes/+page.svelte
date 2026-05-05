<script lang="ts">
	import { onMount } from 'svelte';

	let theme = $state<'light' | 'dark'>('light');
	let mounted = $state(false);

	onMount(() => {
		const saved = localStorage.getItem('theme-swipe-theme');
		const prefersDark = window.matchMedia('(prefers-color-scheme: dark)').matches;
		theme = saved === 'dark' || (!saved && prefersDark) ? 'dark' : 'light';
		document.documentElement.classList.toggle('dark', theme === 'dark');
		document.documentElement.style.colorScheme = theme;
		mounted = true;
	});

	function applyTheme(nextTheme: 'light' | 'dark') {
		theme = nextTheme;
		localStorage.setItem('theme-swipe-theme', nextTheme);
		document.documentElement.classList.toggle('dark', nextTheme === 'dark');
		document.documentElement.style.colorScheme = nextTheme;
	}

	function toggleTheme() {
		const nextTheme = theme === 'dark' ? 'light' : 'dark';

		if ('startViewTransition' in document) {
			document.startViewTransition(() => applyTheme(nextTheme));
			return;
		}

		applyTheme(nextTheme);
	}
</script>

<svelte:head>
	<title>Theme Swipe Lab</title>
	<meta
		name="description"
		content="A small SvelteKit demo of the diagonal theme swipe animation using the View Transitions API."
	/>
</svelte:head>

<main class="shell">
	<section class="hero" aria-labelledby="page-title">
		<div class="eyebrow">View Transitions API demo</div>
		<h1 id="page-title">Diagonal theme swipe</h1>
		<p>
			Click the toggle. The page switches theme inside <code>document.startViewTransition()</code>,
			then CSS clips the new root snapshot across the old one.
		</p>

		<div class="actions">
			<button class="theme-toggle" onclick={toggleTheme} aria-label={`Switch to ${theme === 'dark' ? 'light' : 'dark'} mode`} disabled={!mounted}>
				<span class="toggle-icon" aria-hidden="true">{theme === 'dark' ? '☾' : '☀'}</span>
				<span>{theme === 'dark' ? 'Switch to light' : 'Switch to dark'}</span>
			</button>
		</div>
	</section>

	<section class="demo-panel" aria-label="How it works">
		<div>
			<span class="step">01</span>
			<h2>Snapshot</h2>
			<p>The browser captures the current rendered page as the old root view.</p>
		</div>
		<div>
			<span class="step">02</span>
			<h2>Flip class</h2>
			<p>The callback toggles <code>html.dark</code>, changing CSS variables instantly.</p>
		</div>
		<div>
			<span class="step">03</span>
			<h2>Clip reveal</h2>
			<p><code>::view-transition-new(root)</code> animates a diagonal polygon wipe.</p>
		</div>
	</section>

	<pre class="code"><code>{`::view-transition-group(root) {
  animation-duration: 700ms;
  animation-timing-function: cubic-bezier(0.16, 1, 0.3, 1);
}

.dark::view-transition-new(root) {
  animation-name: reveal-dark;
}`}</code></pre>
</main>

<style>
	:global(:root) {
		--bg: oklch(96% 0.025 78);
		--surface: oklch(99% 0.018 78);
		--text: oklch(23% 0.04 70);
		--muted: oklch(46% 0.035 70);
		--line: oklch(83% 0.035 78);
		--accent: oklch(61% 0.17 35);
		--accent-text: oklch(99% 0.016 78);
		--shadow: 0 28px 80px oklch(30% 0.08 50 / 16%);
		background: var(--bg);
		color: var(--text);
	}

	:global(html.dark) {
		--bg: oklch(17% 0.035 260);
		--surface: oklch(22% 0.035 260);
		--text: oklch(93% 0.02 78);
		--muted: oklch(72% 0.025 260);
		--line: oklch(35% 0.035 260);
		--accent: oklch(76% 0.14 85);
		--accent-text: oklch(20% 0.04 75);
		--shadow: 0 28px 90px oklch(5% 0.02 260 / 55%);
	}

	:global(body) {
		margin: 0;
		min-width: 320px;
		min-height: 100vh;
		background:
			linear-gradient(135deg, color-mix(in oklch, var(--accent), transparent 82%) 0 1px, transparent 1px 22px),
			radial-gradient(circle at 15% 10%, color-mix(in oklch, var(--accent), transparent 70%), transparent 28rem),
			var(--bg);
		color: var(--text);
		font-family:
			ui-serif, Georgia, Cambria, 'Times New Roman', serif;
	}

	:global(::view-transition-group(root)) {
		animation-duration: 700ms;
		animation-timing-function: cubic-bezier(0.16, 1, 0.3, 1);
	}

	:global(::view-transition-old(root)) {
		z-index: -1;
		animation: none;
	}

	:global(::view-transition-new(root)) {
		animation-name: reveal-light;
	}

	:global(.dark::view-transition-new(root)) {
		animation-name: reveal-dark;
	}

	@keyframes reveal-dark {
		0% {
			clip-path: polygon(50% -71%, -50% 71%, -50% 71%, 50% -71%);
			opacity: 0.55;
		}
		100% {
			clip-path: polygon(50% -71%, -50% 71%, 50% 171%, 171% 50%);
			opacity: 1;
		}
	}

	@keyframes reveal-light {
		0% {
			clip-path: polygon(171% 50%, 50% 171%, 50% 171%, 171% 50%);
			opacity: 0.55;
		}
		100% {
			clip-path: polygon(171% 50%, 50% 171%, -50% 71%, 50% -71%);
			opacity: 1;
		}
	}

	@media (prefers-reduced-motion: reduce) {
		:global(::view-transition-group(root)) {
			animation-duration: 1ms;
		}
	}

	.shell {
		width: min(1120px, calc(100vw - 32px));
		min-height: 100vh;
		margin: 0 auto;
		padding: clamp(32px, 7vw, 96px) 0;
	}

	.hero {
		display: grid;
		gap: 24px;
		max-width: 820px;
	}

	.eyebrow,
	.step {
		width: fit-content;
		border: 1px dashed var(--line);
		border-radius: 999px;
		padding: 7px 11px;
		color: var(--muted);
		font-family: ui-monospace, SFMono-Regular, Menlo, monospace;
		font-size: 12px;
		letter-spacing: 0.08em;
		text-transform: uppercase;
	}

	h1 {
		max-width: 11ch;
		margin: 0;
		font-size: clamp(64px, 12vw, 160px);
		font-weight: 900;
		letter-spacing: -0.08em;
		line-height: 0.82;
	}

	p {
		max-width: 680px;
		margin: 0;
		color: var(--muted);
		font-size: clamp(18px, 2vw, 23px);
		line-height: 1.55;
	}

	code {
		border-radius: 0.35em;
		background: color-mix(in oklch, var(--accent), transparent 86%);
		padding: 0.08em 0.28em;
		font-family: ui-monospace, SFMono-Regular, Menlo, monospace;
		font-size: 0.86em;
		color: var(--text);
	}

	.actions {
		padding-top: 10px;
	}

	.theme-toggle {
		display: inline-flex;
		align-items: center;
		gap: 12px;
		border: 0;
		border-radius: 999px;
		background: var(--accent);
		color: var(--accent-text);
		padding: 14px 18px 14px 14px;
		font: 700 16px/1 ui-sans-serif, system-ui, sans-serif;
		box-shadow: var(--shadow);
		cursor: pointer;
		transition:
			transform 180ms ease,
			filter 180ms ease;
	}

	.theme-toggle:hover {
		transform: translateY(-2px);
		filter: saturate(1.08);
	}

	.theme-toggle:active {
		transform: translateY(0) scale(0.98);
	}

	.theme-toggle:focus-visible {
		outline: 3px solid color-mix(in oklch, var(--accent), white 45%);
		outline-offset: 4px;
	}

	.theme-toggle:disabled {
		cursor: wait;
		opacity: 0.7;
	}

	.toggle-icon {
		display: grid;
		width: 34px;
		height: 34px;
		place-items: center;
		border-radius: 50%;
		background: color-mix(in oklch, var(--accent-text), transparent 82%);
		font-size: 18px;
	}

	.demo-panel {
		display: grid;
		grid-template-columns: repeat(3, 1fr);
		gap: 1px;
		margin-top: clamp(48px, 9vw, 120px);
		overflow: hidden;
		border: 1px solid var(--line);
		border-radius: 32px;
		background: var(--line);
		box-shadow: var(--shadow);
	}

	.demo-panel > div {
		display: grid;
		align-content: start;
		gap: 18px;
		min-height: 240px;
		background: color-mix(in oklch, var(--surface), transparent 5%);
		padding: clamp(22px, 4vw, 36px);
	}

	h2 {
		margin: 0;
		font-size: clamp(28px, 4vw, 46px);
		line-height: 0.95;
		letter-spacing: -0.05em;
	}

	.demo-panel p {
		font-size: 16px;
		line-height: 1.55;
	}

	.code {
		overflow: auto;
		margin: clamp(28px, 5vw, 56px) 0 0;
		border: 1px solid var(--line);
		border-radius: 24px;
		background: color-mix(in oklch, var(--surface), transparent 4%);
		padding: 24px;
		color: var(--text);
		font-size: 14px;
		line-height: 1.6;
		box-shadow: var(--shadow);
	}

	.code code {
		background: transparent;
		padding: 0;
	}

	@media (max-width: 760px) {
		.demo-panel {
			grid-template-columns: 1fr;
		}

		.demo-panel > div {
			min-height: 0;
		}
	}
</style>
