<script lang="ts">
	import { Separator } from "$lib/components/ui/separator";

	type Props = {
		data: any;
	};

	let { data }: Props = $props();

	// Track if we're on mobile
	let isMobile = $state(false);
	// Track the split position
	let splitPosition = $state(50);

	/** Handle window resize */
	const handleResize = () => {
		isMobile = window.innerWidth < 768;
	};

	/** Handle mouse movement for resizing */
	const handleMouseMove = (e: MouseEvent) => {
		if (isMobile) return;

		const container = e.currentTarget as HTMLElement;
		const { left, width } = container.getBoundingClientRect();
		const newPosition = ((e.clientX - left) / width) * 100;

		// Limit the split between 20% and 80%
		splitPosition = Math.min(Math.max(newPosition, 20), 80);
	};
</script>

<svelte:window onresize={handleResize} />

<div class="h-dvh w-full" onmousemove={handleMouseMove} role="separator" aria-label="Resize panels">
	<div class="flex h-full">
		<!-- Left panel -->
		<div class="overflow-auto p-4" style={!isMobile ? `width: ${splitPosition}%` : "width: 100%"}>
			<pre class="whitespace-pre-wrap">{JSON.stringify(data, null, 2)}</pre>
		</div>

		{#if !isMobile}
			<!-- Resizer -->
			<div class="flex cursor-col-resize items-center">
				<Separator orientation="vertical" />
			</div>

			<!-- Right panel -->
			<div class="overflow-auto p-4" style={`width: ${100 - splitPosition}%`}>
				<pre class="whitespace-pre-wrap">{JSON.stringify(data, null, 2)}</pre>
			</div>
		{/if}
	</div>
</div>
