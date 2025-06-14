<script lang="ts">
	import { fade } from "svelte/transition";
	import { ChevronDownIcon, ArrowDownIcon, CirclePlayIcon } from "lucide-svelte";
	import { Button } from "$lib/components/ui/button";
	import { DropdownMenu, DropdownMenuContent, DropdownMenuItem, DropdownMenuTrigger } from "$lib/components/ui/dropdown-menu";
	import { Badge } from "$lib/components/ui/badge";
	import FileUpload from "$lib/components/file-upload.svelte";
	import SplitView from "$lib/components/split-view.svelte";

	// Track if file is uploaded
	let isFileUploaded = $state(false);
	// Store uploaded file data
	let fileData: any = $state(null);
	// Track if user wants to go to upload section
	let showUpload = $state(false);

	/** Handle file upload completion */
	const handleUploadComplete = (data: any) => {
		fileData = data;
		isFileUploaded = true;
	};

	/** Navigate to LinkedIn upload page */
	const goToLinkedIn = () => {
		window.location.href = '/linkedin';
	};

	/** Navigate to upload section */
	const goToUpload = () => {
		const uploadSection = document.getElementById('upload');
		if (uploadSection) {
			uploadSection.scrollIntoView({ behavior: 'smooth' });
		}
	};

	/** Start from scratch with sample resume */
	const startFromScratch = async () => {
		try {
			const response = await fetch('/Alexander_Weimer.resume.json');
			const sampleData = await response.json();
			fileData = sampleData;
			isFileUploaded = true;
		} catch (error) {
			console.error('Failed to load sample resume:', error);
		}
	};
</script>

<div class="min-h-dvh bg-gray-50 text-foreground">
	<!-- Floating Navigation -->
	<div class="fixed left-0 right-0 top-6 z-50 px-4">
		<nav class="mx-auto flex h-14 max-w-2xl items-center justify-between rounded-full border bg-white/80 px-6 shadow-lg backdrop-blur-sm">
			<!-- Logo -->
			<a href="/" class="flex items-center gap-0 text-xl font-semibold text-sky-600	">
				<img src="/favicon.png" alt="Logo" class="h-8 w-8 mr-2" />
				<span class="inline italic font-bold">effortless</span>
				<span class="inline font-bold">.cv</span>
			</a>

			<!-- Nav Links -->
			<div class="hidden items-center gap-6 md:flex">
				<a href="#features" class="text-sm text-sky-900 transition-colors hover:text-sky-500">Features</a>
				<a href="#templates" class="text-sm text-sky-900 transition-colors hover:text-sky-500">Templates</a>
				<Button variant="default" class="bg-sky-500 text-white transition-colors hover:bg-sky-600" onclick={goToUpload}>
					Get Started
				</Button>
			</div>

			<!-- Mobile Menu Button -->
			<Button variant="ghost" class="text-gray-600 md:hidden">
				<span class="sr-only">Open menu</span>
				<ChevronDownIcon class="h-6 w-6" />
			</Button>
		</nav>
	</div>

	<!-- Hero Section -->
	<main class="flex min-h-[calc(100vh-4rem)] flex-col items-center justify-center px-4 pt-16">
		<div class="text-center">
			<!-- Beta Badge -->
			<Badge variant="outline" class="mb-8 border-sky-200 bg-sky-50 text-sky-700">
				<span class="mr-1 flex h-2 w-2 rounded-full bg-sky-500"></span>
				Now in Beta
			</Badge>

			<h1 class="mb-6 text-4xl font-bold tracking-tight text-gray-900 sm:text-6xl md:text-7xl">
				your dream cv,
				<span class="text-sky-600 italic">made effortless</span>
	
			</h1>

			<p class="mx-auto mb-8 max-w-2xl text-lg text-gray-600">
				Create a standout CV in minutes, not hours&mdash;entirely free.
			</p>

			<!-- CTA Buttons -->
			<div class="flex flex-col items-center gap-4 sm:flex-row sm:justify-center">
				<Button class="bg-sky-500 text-white transition-colors hover:bg-sky-600" onclick={goToLinkedIn}>
					Start with <img src="/img/linkedin-wordmark.svg" alt="LinkedIn" class="h-14 transform translate-y-[2px] invert -ml-[3px]" />
				</Button>
				<Button variant="outline" class="border-gray-200 text-gray-700 hover:bg-gray-50" onclick={startFromScratch}>
					Start from scratch
					<ArrowDownIcon class="ml-2 h-4 w-4" />
				</Button>
				<!--<Button variant="outline" class="border-gray-200 text-gray-700 hover:bg-gray-50">
					Watch demo
					<CirclePlayIcon class="ml-2 h-4 w-4" />
				</Button>-->
			</div>
		</div>
	</main>

	<!-- Upload Section -->
	<section id="upload" class="bg-background text-foreground">
		{#if !isFileUploaded}
			<div transition:fade>
				<FileUpload onUploadComplete={handleUploadComplete} />
			</div>
		{:else}
			<div transition:fade>
				<SplitView data={fileData} />
			</div>
		{/if}
	</section>
</div>
