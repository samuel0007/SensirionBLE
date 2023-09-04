<script lang="ts">
	import { UUIDS } from '$lib/UUIDS';
	import { onMount } from 'svelte';

	export let BLEDevice: BluetoothDevice;
	export let BLEDeviceServer: BluetoothRemoteGATTServer;

	let characteristic: BluetoothRemoteGATTCharacteristic;

	let samplesAvailable: number = 0;
	let SensorValues: { temperature?: number; humidity?: number; CO2?: number } = {};

	onMount(() => {
		subscribeToSensorData();
	});

	async function subscribeToSensorData() {
		const service = await BLEDeviceServer.getPrimaryService(UUIDS.SensirionDataLoggerServiceUUID);
		characteristic = await service.getCharacteristic(UUIDS.SensirionDataLoggerDataTransferUUID);
		characteristic.oncharacteristicvaluechanged = onSensorDataChanged;
		await characteristic.startNotifications();
	}

	async function onSensorDataChanged(event: Event) {
		const data: DataView | undefined = (event.target as BluetoothRemoteGATTCharacteristic).value;
		if (!data) {
			console.log('No data received');
			return;
		}
		// console.log(data)
		if (data.getInt16(0, true) == 0) {
			// Header sequence
			samplesAvailable = data.getInt16(14, true);
			console.log('Sampling interval:', data.getInt32(6, true));
			console.log('Sample Type:', data.getInt16(4, true));
		} else {
			// Data sequence
			samplesAvailable -= data.getInt16(2, true) ? 1 : 0;
			samplesAvailable -= data.getInt16(8, true) ? 1 : 0;
			samplesAvailable -= data.getInt16(14, true) ? 1 : 0;

			if (samplesAvailable == 0) {
				SensorValues.temperature = -45 + (175.0 * data.getUint16(2, true)) / (2 << (16 - 1));
				SensorValues.humidity = (100.0 * data.getUint16(4, true)) / (2 << (16 - 1));
				SensorValues.CO2 = data.getUint16(6, true);
			}
			// console.log("CO2 value:", data.getUint16(6, true));
		}
		// console.log("Sequence number:", data.getInt16(0, true))
		if (samplesAvailable == 0) {
			console.log('All samples received');
			await characteristic.stopNotifications();
			console.log('Restarting notifications');
			await characteristic.startNotifications();
		}
	}
</script>

<div class="flex flex-row w-full h-full items-center p-10">
	{#if SensorValues.CO2}
    <!-- Three rectangles, each with their respective values -->
        <div class="w-1/3 h-full px-2 flex flex-col gap-1">
            <div class="header pt-1 pb-2 rounded-lg flex items-center justify-center text-white text-3xl text-bold {SensorValues.CO2 < 1000 ? "bg-green-700" : "bg-orange-600"}">
                CO2
            </div>
            <div class="h-full rounded-lg flex items-center justify-center text-white {SensorValues.CO2 < 1000 ? "bg-green-700" : "bg-orange-600"}" >
                {SensorValues.CO2} ppm
            </div>
        </div>
        <div class="w-1/3 h-full px-2 flex flex-col gap-1">
            <div class="header pt-1 pb-2 rounded-lg flex items-center justify-center text-white text-3xl text-bold bg-green-700">
                Temperature
            </div>
            <div class="h-full rounded-lg flex items-center justify-center text-white bg-green-700">
                {SensorValues.temperature?.toFixed(2)} °C
            </div>
        </div>
        <div class="w-1/3 h-full px-2 flex flex-col gap-1">
            <div class="header pt-1 pb-2 rounded-lg flex items-center justify-center text-white text-3xl text-bold bg-green-700">
                Relative Humidity
            </div>
            <div class="h-full rounded-lg flex items-center justify-center text-white bg-green-700">
                {SensorValues.humidity?.toFixed(2)} %
            </div>
        </div>
	{:else}
		<div class="flex w-full h-full items-center justify-center">
            <p>Waiting for data...</p>
        </div>
	{/if}
</div>
