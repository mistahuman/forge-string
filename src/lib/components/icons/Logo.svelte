<script lang="ts">
	let { width = '24', height = '24' }: { width?: string; height?: string } = $props();
</script>

<svg
	{width}
	{height}
	viewBox="0 0 24 24"
	fill="currentColor"
	class="text-primary"
	xmlns="http://www.w3.org/2000/svg"
>
	<rect x="0.5" y="8" width="5" height="8" rx="1.5" />
	<rect x="6.75" y="10.5" width="2" height="3" rx="0.75" />
	<rect x="9.5" y="8" width="5" height="8" rx="1.5" />
	<rect x="15.75" y="10.5" width="2" height="3" rx="0.75" />
	<rect x="18.5" y="8" width="5" height="8" rx="1.5" />
</svg>
