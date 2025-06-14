<script lang="ts">
	import { fade } from "svelte/transition";
	import { ArrowLeftIcon } from "lucide-svelte";
	import { Button } from "$lib/components/ui/button";
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

<svelte:head>
	<title>LinkedIn Upload - effortless.cv</title>
</svelte:head>

<div class="min-h-dvh bg-gray-50">
	{#if !isFileUploaded}
		<div transition:fade>
			<!-- Header -->
			<header class="border-b bg-white">
				<div class="mx-auto flex max-w-6xl items-center justify-between px-4 py-4">
					<a href="/" class="flex items-center  text-xl font-semibold text-sky-600">
						<img src="/favicon.png" alt="Logo" class="h-8 w-8 mr-2" />
						<span class="italic font-bold">effortless</span><span class="font-bold">.cv</span>
					</a>
					<Button variant="ghost" onclick={() => window.history.back()}>
						<ArrowLeftIcon class="mr-2 h-4 w-4" />
						Back
					</Button>
				</div>
			</header>

			<!-- LinkedIn Instructions Section -->
			<section class="bg-background text-foreground">
				<!-- LinkedIn Instructions Header -->
				<div class="bg-sky-50 px-4 pt-12 pb-px">
					<div class="mx-auto max-w-4xl text-center">
						<h1 class="mb-6 text-3xl font-bold text-sky-950">
							Import from LinkedIn
						</h1>
						<div class="mx-auto max-w-2xl text-sky-900">
							<p class="mb-6 text-lg">
								Follow these simple steps to import your LinkedIn profile and create your CV:
							</p>
							
							<div class="mb-8 space-y-4 text-left">
								<div class="flex items-start gap-3">
									<div class="flex h-6 w-6 items-center justify-center rounded-full bg-sky-600 text-sm font-medium text-white mt-[1px]">1</div>
									<div>
                                        <a 
                                            href="https://chromewebstore.google.com/detail/json-resume-exporter/caobgmmcpklomkcckaenhjlokpmfbdec" 
                                            target="_blank" 
                                            rel="noopener noreferrer"
                                            class="inline-flex items-center gap-2 rounded-lg font-bold text-sky-900 transition-opacity hover:opacity-70"
                                        >
                                            Install JSON Resume Exporter Chrome Extension
                                            <svg class="h-4 w-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                                                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M10 6H6a2 2 0 00-2 2v10a2 2 0 002 2h10a2 2 0 002-2v-4M14 4h6m0 0v6m0-6L10 14" />
                                            </svg>
                                        </a>

									</div>
								</div>
								<div class="flex items-start gap-3">
									<div class="flex h-6 w-6 items-center justify-center rounded-full bg-sky-600 text-sm font-medium text-white mt-[1px]">2</div>
									<div>
                                        <a 
                                            href="https://www.linkedin.com/login" 
                                            target="_blank" 
                                            rel="noopener noreferrer"
                                            class="inline-flex items-center gap-2 rounded-lg font-bold text-sky-900 transition-opacity hover:opacity-70"
                                        >
                                            Sign in to LinkedIn
                                            <svg class="h-4 w-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                                                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M10 6H6a2 2 0 00-2 2v10a2 2 0 002 2h10a2 2 0 002-2v-4M14 4h6m0 0v6m0-6L10 14" />
                                            </svg>
                                        </a>

									</div>
								</div>
								<div class="flex items-start gap-3">
									<div class="flex h-6 w-6 items-center justify-center rounded-full bg-sky-600 text-sm font-medium text-white mt-[6px]">3</div>
									<div>
										<p class="font-medium">Export your data</p>
										<p class="text-sm text-gray-600">Click the extension icon and download the JSON file</p>
									</div>
								</div>
								<div class="flex items-start gap-3">
									<div class="flex h-6 w-6 items-center justify-center rounded-full bg-sky-600 text-sm font-medium text-white mt-[6px]">4</div>
									<div>
										<p class="font-medium">Upload the file</p>
										<p class="text-sm text-gray-600">Upload your JSON file below to create your CV</p>
									</div>
								</div>
							</div>
						</div>
					</div>
				</div>

				<!-- Upload Section -->
				<div class="bg-background">
					<FileUpload onUploadComplete={handleUploadComplete} />
				</div>

				<!-- Temporary measure notice -->
				<div class="bg-gray-100 px-4 py-6">
					<div class="mx-auto max-w-4xl text-center">
						<p class="text-sm text-gray-600">
							<em>This is a temporary measure while we restore LinkedIn API access.</em>
						</p>
					</div>
				</div>
			</section>
		</div>
	{:else}
		<div transition:fade>
			<SplitView data={fileData} />
		</div>
	{/if}
</div>