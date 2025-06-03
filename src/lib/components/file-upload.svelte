<script lang="ts">
	import { UploadIcon } from "lucide-svelte";
	import { Card, CardContent, CardDescription, CardHeader, CardTitle } from "$lib/components/ui/card";
	import { Input } from "$lib/components/ui/input";
	import { Label } from "$lib/components/ui/label";
	import { Alert, AlertDescription } from "$lib/components/ui/alert";

	type Props = {
		onUploadComplete?: (data: any) => void;
	};

	let { onUploadComplete }: Props = $props();

	// Track error state
	let error: string | null = $state(null);

	/** Handle file input change */
	const handleFileChange = async (event: Event) => {
		const input = event.target as HTMLInputElement;
		const file = input.files?.[0];

		if (!file) return;

		// Check if file is JSON
		if (!file.type.includes("json")) {
			error = "Please upload a JSON file";
			return;
		}

		try {
			const text = await file.text();
			const data = JSON.parse(text);
			error = null;
			if (onUploadComplete) onUploadComplete(data);
		} catch (e) {
			error = "Invalid JSON file";
		}
	};
</script>

<div class="container flex min-h-dvh items-center justify-center p-4">
	<Card class="w-full max-w-md">
		<CardHeader>
			<CardTitle>Upload JSON File</CardTitle>
			<CardDescription>Please upload a JSON file to continue</CardDescription>
		</CardHeader>
		<CardContent>
			<div class="flex flex-col gap-4">
				<div class="grid w-full items-center gap-1.5">
					<Label for="file">JSON File</Label>
					<Input id="file" type="file" accept=".json,application/json" onchange={handleFileChange} />
				</div>

				{#if error}
					<Alert variant="destructive">
						<AlertDescription>{error}</AlertDescription>
					</Alert>
				{/if}

				<div class="flex items-center justify-center rounded-lg border-2 border-dashed p-4">
					<div class="flex flex-col items-center gap-2 text-muted-foreground">
						<UploadIcon size={40} />
						<p>Drag and drop your JSON file here</p>
					</div>
				</div>
			</div>
		</CardContent>
	</Card>
</div>
