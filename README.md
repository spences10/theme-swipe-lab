# Theme Swipe Lab

A tiny SvelteKit example of DeepWiki's diagonal light/dark theme wipe.

## How it works

The toggle calls `document.startViewTransition()` and flips `html.dark` inside the callback. CSS then animates `::view-transition-new(root)` with a diagonal `clip-path` polygon while the old root snapshot sits behind it.

```ts
document.startViewTransition(() => {
	document.documentElement.classList.toggle('dark', nextTheme === 'dark');
});
```

## Run

```sh
pnpm dev
```

Open http://localhost:5173.
