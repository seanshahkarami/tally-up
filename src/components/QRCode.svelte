<script>
    import { onMount } from "svelte";
    let { text } = $props();
    let qrContainer;

    onMount(() => {
        if (!window.QRCode) {
            console.error(
                "QRCode library not found. Make sure it's loaded in index.html.",
            );
            return;
        }

        new QRCode(qrContainer, {
            text,
            width: 256,
            height: 256,
        });
    });
</script>

<div bind:this={qrContainer}></div>