<script lang="ts">
	import { fade } from "svelte/transition";
	import FileUpload from "$lib/components/file-upload.svelte";
	import SplitView from "$lib/components/split-view.svelte";

	// Track if file is uploaded
	let isFileUploaded = $state(false);
	// Store uploaded file data
	let fileData: any = $state(null);

	/** Handle file upload completion */
	const handleUploadComplete = (data: any) => {
		fileData = data;
		isFileUploaded = true;
	};
</script>

<div class="flex min-h-dvh flex-col bg-background text-foreground">
	{#if !isFileUploaded}
		<div transition:fade>
			<FileUpload onUploadComplete={handleUploadComplete} />
		</div>
	{:else}
		<div transition:fade>
			<SplitView data={fileData} />
		</div>
	{/if}
</div>
