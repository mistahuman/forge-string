<script lang="ts">
	import { onMount } from 'svelte';
	import { Copy, Check } from 'lucide-svelte';
	import { historyStore } from '$lib/stores/history.svelte';

	const PRESETS = ['OR', 'AND', ',', '|', 'Custom'] as const;
	type Preset = (typeof PRESETS)[number];

	const OP_MAP: Record<Preset, string> = {
		OR: ' OR ',
		AND: ' AND ',
		',': ', ',
		'|': ' | ',
		Custom: ''
	};

	let input = $state('');
	let preset = $state<Preset>('OR');
	let customOp = $state('');
	let copiedMain = $state(false);

	let operator = $derived(preset === 'Custom' ? customOp : OP_MAP[preset]);

	let tokens = $derived(
		input
			.split(/[\s,;]+/)
			.map((t) => t.trim())
			.filter(Boolean)
	);

	let output = $derived(tokens.length > 0 ? tokens.join(operator) : '');

	onMount(() => {
		historyStore.init();
	});

	async function copyOutput() {
		if (!output) return;
		await navigator.clipboard.writeText(output);
		historyStore.save({ id: Date.now(), output, operatorLabel: preset, tokenCount: tokens.length });
		copiedMain = true;
		setTimeout(() => (copiedMain = false), 2000);
	}
</script>

<svelte:head>
	<title>Forge String</title>
</svelte:head>

<div class="container mx-auto max-w-5xl space-y-6 px-6 py-10">

	<!-- Input -->
	<div class="space-y-2">
		<label for="strings-input" class="text-xs font-semibold uppercase tracking-widest text-surface-500-400">
			Input
		</label>
		<textarea
			id="strings-input"
			bind:value={input}
			placeholder="Paste or type tokens — spaces, commas, semicolons, and newlines all work as separators…"
			rows="8"
			spellcheck="false"
			class="w-full resize-y rounded-base border border-surface-200-800 bg-surface-50-950 p-4 font-mono text-sm
				focus-visible:outline-none focus-visible:ring-1 focus-visible:ring-primary-500"
		></textarea>
	</div>

	<!-- Join with -->
	<div class="space-y-3">
		<p class="text-xs font-semibold uppercase tracking-widest text-surface-500-400">Join with</p>
		<div
			class="flex gap-1 rounded-container bg-surface-200-800 p-1"
			role="group"
			aria-label="Join operator"
		>
			{#each PRESETS as p (p)}
				<button
					type="button"
					onclick={() => (preset = p)}
					class="flex-1 rounded-base px-3 py-1.5 text-center text-sm font-mono font-medium transition-colors
						{preset === p
							? 'bg-surface-50-950 text-surface-900-50 shadow-sm'
							: 'text-surface-500-400 hover:text-surface-700-300'}"
					aria-pressed={preset === p}
				>
					{p}
				</button>
			{/each}
		</div>
		<div class="h-10">
			{#if preset === 'Custom'}
				<input
					type="text"
					bind:value={customOp}
					placeholder="Type a custom separator, e.g.  &&  or  UNION"
					class="h-full w-full rounded-base border border-surface-200-800 bg-surface-50-950 px-3 font-mono text-sm
						focus-visible:outline-none focus-visible:ring-1 focus-visible:ring-primary-500"
				/>
			{/if}
		</div>
	</div>

	<!-- Output -->
	<div class="card preset-filled-surface-100-900 overflow-hidden">
		<div class="flex items-center justify-between border-b border-surface-200-800 px-4 py-2.5">
			<div class="flex items-center gap-2">
				<span class="text-xs font-semibold uppercase tracking-widest text-surface-500-400">Output</span>
				{#if tokens.length > 0}
					<span class="badge preset-tonal-primary font-mono text-xs">
						{tokens.length} token{tokens.length !== 1 ? 's' : ''}
					</span>
				{/if}
			</div>
			<button
				onclick={copyOutput}
				disabled={!output}
				class="btn preset-filled-primary-500 gap-1.5 transition-opacity disabled:opacity-40"
			>
				{#if copiedMain}
					<Check size={14} />
					Copied!
				{:else}
					<Copy size={14} />
					Copy
				{/if}
			</button>
		</div>
		<textarea
			readonly
			rows="5"
			value={output}
			placeholder="Output will appear here as you type…"
			spellcheck="false"
			class="w-full resize-none bg-transparent p-4 font-mono text-sm
				text-surface-900-50 placeholder:text-surface-400-600 focus-visible:outline-none"
		></textarea>
	</div>

</div>
