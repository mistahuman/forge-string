<script lang="ts">
	import { onMount } from 'svelte';
	import { Collapsible } from '@skeletonlabs/skeleton-svelte';
	import { Copy, Check, ChevronDown } from 'lucide-svelte';

	const PRESETS = ['OR', 'AND', ',', '|', 'Custom'] as const;
	type Preset = (typeof PRESETS)[number];

	const OP_MAP: Record<Preset, string> = {
		OR: ' OR ',
		AND: ' AND ',
		',': ', ',
		'|': ' | ',
		Custom: ''
	};

	const LS_KEY = 'forge-string-history';
	const MAX_HISTORY = 5;

	type HistoryEntry = {
		id: number;
		output: string;
		operatorLabel: string;
		tokenCount: number;
	};

	let input = $state('');
	let preset = $state<Preset>('OR');
	let customOp = $state('');
	let history = $state<HistoryEntry[]>([]);
	let historyOpen = $state(false);
	let copiedMain = $state(false);
	let copiedId = $state<number | null>(null);

	let operator = $derived(preset === 'Custom' ? customOp : OP_MAP[preset]);

	let tokens = $derived(
		input
			.split(/[\s,;]+/)
			.map((t) => t.trim())
			.filter(Boolean)
	);

	let output = $derived(tokens.length > 0 ? tokens.join(operator) : '');

	onMount(() => {
		try {
			const stored = localStorage.getItem(LS_KEY);
			if (stored) history = JSON.parse(stored);
		} catch {
			history = [];
		}
	});

	function saveHistory(entry: HistoryEntry) {
		const deduped = history.filter((h) => h.output !== entry.output);
		history = [entry, ...deduped].slice(0, MAX_HISTORY);
		localStorage.setItem(LS_KEY, JSON.stringify(history));
		historyOpen = true;
	}

	async function copyOutput() {
		if (!output) return;
		await navigator.clipboard.writeText(output);
		saveHistory({ id: Date.now(), output, operatorLabel: preset, tokenCount: tokens.length });
		copiedMain = true;
		setTimeout(() => (copiedMain = false), 2000);
	}

	async function copyHistoryEntry(entry: HistoryEntry) {
		await navigator.clipboard.writeText(entry.output);
		copiedId = entry.id;
		setTimeout(() => (copiedId = null), 2000);
	}

	function clearHistory() {
		history = [];
		localStorage.removeItem(LS_KEY);
	}
</script>

<svelte:head>
	<title>Forge String</title>
</svelte:head>

<div class="container mx-auto max-w-3xl space-y-6 px-4 py-10">
	<!-- Title -->
	<div class="space-y-1">
		<h1 class="h2 font-bold">Forge String</h1>
		<p class="text-sm text-surface-500-400">
			Turn a messy token list into a clean query string.
		</p>
	</div>

	<!-- Input -->
	<div class="space-y-2">
		<label for="strings-input" class="text-sm font-medium">Input</label>
		<textarea
			id="strings-input"
			bind:value={input}
			placeholder="Paste or type tokens — spaces, commas, semicolons, and newlines all work as separators…"
			rows="6"
			spellcheck="false"
			class="w-full resize-y rounded border border-surface-200-800 bg-surface-50-950 p-3 font-mono text-sm
				focus-visible:outline-none focus-visible:ring-1 focus-visible:ring-primary-500"
		></textarea>
	</div>

	<!-- Join with -->
	<div class="space-y-3">
		<p class="text-sm font-medium">Join with</p>
		<div
			class="flex gap-1 rounded-xl bg-surface-200-800 p-1"
			role="group"
			aria-label="Join operator"
		>
			{#each PRESETS as p (p)}
				<button
					type="button"
					onclick={() => (preset = p)}
					class="flex-1 rounded-lg px-3 py-1.5 text-center text-sm font-mono font-medium transition-colors
						{preset === p
							? 'bg-surface-50-950 text-surface-900-50 shadow-sm'
							: 'text-surface-500-400 hover:text-surface-700-300'}"
					aria-pressed={preset === p}
				>
					{p}
				</button>
			{/each}
		</div>
		<!-- Always-rendered placeholder to avoid layout jump -->
		<div class="h-10">
			{#if preset === 'Custom'}
				<input
					type="text"
					bind:value={customOp}
					placeholder="Type a custom separator, e.g.  &&  or  UNION"
					class="h-full w-full rounded border border-surface-200-800 bg-surface-50-950 px-3 font-mono text-sm
						focus-visible:outline-none focus-visible:ring-1 focus-visible:ring-primary-500"
				/>
			{/if}
		</div>
	</div>

	<!-- Output -->
	<div class="space-y-2">
		<div class="flex items-center justify-between gap-2">
			<div class="flex items-center gap-2">
				<span class="text-sm font-medium">Output</span>
				{#if tokens.length > 0}
					<span class="badge preset-outlined font-mono text-xs">
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
			rows="4"
			value={output}
			placeholder="Output will appear here as you type…"
			spellcheck="false"
			class="w-full resize-none rounded border border-surface-200-800 bg-surface-100-900 p-3 font-mono text-sm
				text-surface-900-50 placeholder:text-surface-400-600 focus-visible:outline-none"
		></textarea>
	</div>

	<!-- History -->
	{#if history.length > 0}
		<Collapsible open={historyOpen} onOpenChange={(e) => (historyOpen = e.open)}>
			<div class="card preset-filled-surface-100-900 overflow-hidden border border-surface-200-800">
				<div class="flex items-center gap-2 px-3 py-2.5">
					<Collapsible.Trigger class="flex flex-1 cursor-pointer items-center gap-2 text-left">
						<span class="text-sm font-medium">History</span>
						<span class="badge preset-tonal-surface text-xs">{history.length}</span>
						<Collapsible.Indicator
							class="ml-auto text-surface-500-400 transition-transform data-[state=open]:rotate-180"
						>
							<ChevronDown size={15} />
						</Collapsible.Indicator>
					</Collapsible.Trigger>
					<button onclick={clearHistory} class="btn btn-sm preset-tonal shrink-0 text-xs">
						Clear
					</button>
				</div>
				<Collapsible.Content>
					<div class="divide-y divide-surface-200-800 border-t border-surface-200-800">
						{#each history as entry (entry.id)}
							<div class="flex items-center gap-3 px-3 py-2.5">
								<div class="min-w-0 flex-1 space-y-1">
									<p class="truncate font-mono text-sm">{entry.output}</p>
									<div class="flex items-center gap-1.5">
										<span class="badge preset-tonal-surface text-xs">{entry.operatorLabel}</span>
										<span class="badge preset-outlined text-xs">
											{entry.tokenCount} token{entry.tokenCount !== 1 ? 's' : ''}
										</span>
									</div>
								</div>
								<button
									onclick={() => copyHistoryEntry(entry)}
									class="btn-icon btn-sm preset-tonal shrink-0"
									aria-label="Copy"
								>
									{#if copiedId === entry.id}
										<Check size={14} />
									{:else}
										<Copy size={14} />
									{/if}
								</button>
							</div>
						{/each}
					</div>
				</Collapsible.Content>
			</div>
		</Collapsible>
	{/if}
</div>
