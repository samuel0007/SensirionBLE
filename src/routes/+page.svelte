<script lang="ts">
	import RequestDeviceButton from './RequestDeviceButton.svelte';
	import SensorData from './SensorData.svelte';
	import { UUIDS } from '$lib/UUIDS';

	let BLEDevice: BluetoothDevice;
    let BLEDeviceServer: BluetoothRemoteGATTServer;


	async function handlePair() {
		try {
			const device = await navigator.bluetooth.requestDevice({
				filters: [{ namePrefix: 'My' }],
				optionalServices: [UUIDS.SensirionDataLoggerServiceUUID]
			});
            if (!device.gatt) {
                throw new Error('No GATT server found');
            };
            BLEDeviceServer = await device.gatt.connect();
            BLEDevice = BLEDeviceServer.device;
		} catch (error) {
			console.log(error);
		}
	}

    

</script>

<div class="w-full h-screen flex flex-col items-center">
	<h1 class="text-3xl font-bold">WebMyAmbience</h1>
	<p>This is a little web app demo to test the MyCO2 sensor with the Web Bluetooth API.</p>
	<div class="bottomBox h-full w-full flex flex-col justify-center items-center">
		{#if BLEDevice}
			<SensorData {BLEDevice} {BLEDeviceServer}/>
		{:else}
			<RequestDeviceButton {handlePair} />
		{/if}
	</div>
</div>
