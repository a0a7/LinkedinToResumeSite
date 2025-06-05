<script lang="ts">
	import { AtSignIcon, GlobeIcon, MailIcon } from "lucide-svelte";

	type Contact = {
		icon: string;
		label: string;
		href?: string;
	};

	let {
		name,
		title,
		contacts,
	}: {
		name: string;
		title: string;
		contacts: Contact[];
	} = $props();

	const getIcon = (type: string) => {
		switch (type) {
			case "@":
				return AtSignIcon;
			case "web":
				return GlobeIcon;
			case "email":
				return MailIcon;
			default:
				return AtSignIcon;
		}
	};
</script>

<header class="text-center">
	<h1 class="text-2xl font-bold">{name}</h1>
	<p class="mt-1 text-muted-foreground">{title}</p>
	<div class="mt-3 flex justify-center gap-4">
		{#each contacts as contact}
			<div class="flex items-center gap-1 text-sm">
				<!-- svelte-ignore svelte_component_deprecated -->
				<svelte:component this={getIcon(contact.icon)} class="h-4 w-4" />
				{#if contact.href}
					<a href={contact.href} class="hover:underline">{contact.label}</a>
				{:else}
					<span>{contact.label}</span>
				{/if}
			</div>
		{/each}
	</div>
</header>
