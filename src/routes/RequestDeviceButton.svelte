<script lang=ts>
    async function getAvaibility() {
        return await navigator.bluetooth.getAvailability();
    }

    export let handlePair: () => void;
</script>

{#await getAvaibility()}
	<p>Checking for bluetooth support...</p>
{:then isAvailable}
    {#if isAvailable}
        <button class="bg-blue-700 text-white text-4xl text-bold px-5 py-3 flex justify-center items-center" on:pointerup={handlePair}>PAIR</button>
    {:else}
        <p>Bluetooth WebApi not available for this device</p>
    {/if}
{:catch error}
	<p style="color: red">{error.message}</p>
{/await}
