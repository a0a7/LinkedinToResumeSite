<script lang="ts">
  import * as Resizable from "$lib/components/ui/resizable";

    type Props = {
        data: any;
    };

    let { data }: Props = $props();

    let isMobile = $state(false);

    const handleResize = () => {
        isMobile = window.innerWidth < 768;
    };
</script>

<svelte:window onresize={handleResize} />

<div class="h-dvh w-full">
    {#if isMobile}
        <div class="h-full overflow-auto p-4">
            <pre class="whitespace-pre-wrap">{JSON.stringify(data, null, 2)}</pre>
        </div>
    {:else}
        <Resizable.PaneGroup direction="horizontal">
            <Resizable.Pane>
                <div class="h-full overflow-auto p-4">
                    <pre class="whitespace-pre-wrap">{JSON.stringify(data, null, 2)}</pre>
                </div>
            </Resizable.Pane>
            <Resizable.Handle withHandle />
            <Resizable.Pane>
            </Resizable.Pane>
        </Resizable.PaneGroup>
    {/if}
</div>