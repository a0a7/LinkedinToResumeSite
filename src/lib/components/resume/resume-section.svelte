<script lang="ts">
	import type { Snippet } from "svelte";

	let {
		title,
		children,
	}: {
		title: string;
		children: Snippet;
	} = $props();
</script>

<section>
	<h2 class="mb-4 border-b pb-1 text-lg font-semibold">{title}</h2>
	{@render children()}
</section>
