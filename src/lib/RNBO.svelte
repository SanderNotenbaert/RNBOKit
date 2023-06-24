<script>
	import './RNBO.css';
	import { onMount } from 'svelte';
	import { createDevice } from '@rnbo/js';
	import RNBOParam from './RNBOcomponents/RNBOParam.svelte';
	import RNBOOutport from './RNBOcomponents/RNBOOutport.svelte';
	import RNBOInport from './RNBOcomponents/RNBOInport.svelte';
	import RNBOInlet from './RNBOcomponents/RNBOInlet.svelte';
	import RNBOMidiIn from './RNBOcomponents/RNBOMidiIn.svelte';
	// import RNBOMidiOut from './RNBOcomponents/RNBOMidiOut.svelte';

	/** @type {string} */
	export let path = '/src/RNBO/patch.export.json';
	/** @type {string} */
	export let dependencies = '/src/RNBO/dependencies.json';

	// Create AudioContext
	/** @type {AudioContext|undefined} */
	let context = undefined;

	const loadContext = () => {
		// @ts-ignore
		let WAContext = window.AudioContext || window.webkitAudioContext;
		context = new WAContext();
	};

	// Create device
	/** @type {import ('@rnbo/js').Device|undefined} */
	export let device = undefined;
	$: device;

	/** @type {import ('@rnbo/js').IPatcher|undefined} */
	let patcher = undefined;

	/** @type {import ('@rnbo/js').ExternalDataInfo[]|undefined} */
	let dependencyFile = undefined;

	/** @type {import ('@rnbo/js').Parameter[]} */
	export let parameters = [];
	let numParameters = 0;

	/** @type {import ('@rnbo/js').MessageInfo[]} */
	export let inports = [];
	let numInports = 0;

	/** @typedef {object} signalPort
	 * @property {number} index - the port index
	 * @property {string} tag - the tag
	 * @property {'signal'} type - the type
	 * @property {string} meta - the meta
	 */
	/** @type {signalPort[]} */
	export let inlets = [];
	let numInlets = 0;

	/** @type {Array<Number>} */
	export let MIDIOutports = [];
	let numMIDIOutports = 0;

	/** @type {import ('@rnbo/js').MessageInfo[]} */
	export let outports = [];
	let numOutports = 0;

	/** @type {signalPort[]} */
	export let outlets = [];
	let numOutlets = 0;

	/** @type {Array<Number>} */
	export let MIDIInports = [];
	let numMIDIInports = 0;
	// set up device
	const deviceSetup = async () => {
		//import the patcher json dynamically!
		patcher = await import(/* @vite-ignore */ path);

		//import the dependency json dynamically!
		dependencyFile = await import(/* @vite-ignore */ dependencies);

		if (patcher && context) {
			device = await createDevice({ context, patcher });

			//load dependencies if they exist
			if (dependencyFile && dependencyFile.length) {
				device.loadDataBufferDependencies(dependencyFile);
			}
			if (patcher.presets && patcher.presets[0]) {
				device.setPreset(patcher.presets[0].preset);
			}
			device.node.connect(context.destination);

			parameters = device.parameters;
			numParameters = patcher.desc.numParameters;

			inports = device.inports;
			numInports = patcher.desc.numInputChannels;

			// @ts-expect-error - desc.inlets does indeed exist
			inlets = patcher.desc.inlets.filter((inlet) => inlet.type === 'signal');
			numInlets = inlets.length;

			MIDIInports = Array.from({ length: device.numMIDIInputPorts }, (_, i) => i + 1);
			numMIDIInports = device.numMIDIInputPorts;

			outports = device.outports;
			numOutports = patcher.desc.numOutputChannels;

			// @ts-expect-error - desc.outlets does indeed exist
			outlets = patcher.desc.outlets.filter((outlet) => outlet.type === 'signal');
			numOutlets = outlets.length;

			MIDIOutports = Array.from({ length: device.numMIDIOutputPorts }, (_, i) => i + 1);
			numMIDIOutports = device.numMIDIOutputPorts;
		} else if (!patcher) {
			throw new Error('No patcher found!');
		}
		if (!context) {
			throw new Error('No context found!');
		}
	};

	// load on client side only (no Window server side)
	onMount(async () => {
		loadContext();
		deviceSetup();
	});
</script>

<!-- html -->
{#if patcher && device && context}
	<div on:click={() => context?.resume()} on:keydown={() => context?.resume()}>
		<slot
			{patcher}
			{device}
			{path}
			{dependencies}
			{MIDIInports}
			{inports}
			{inlets}
			{parameters}
			{outports}
			{outlets}
			{MIDIOutports}
			{numParameters}
			{numInports}
			{numInlets}
			{numOutports}
			{numOutlets}
			{numMIDIOutports}
			{numMIDIInports}
		>
			<div class="RNBOsection">
				<!-- use the json file name as header -->
				<h1>{path.split('/').pop().replace('.json', '')}</h1>

				<!-- create input for each MIDI input port -->
				{#if numMIDIInports > 0}
					<div class="RNBOsection">
						<h2>MIDI inputs</h2>
						{#each MIDIInports as port}
							<RNBOMidiIn {port} {device} />
						{/each}
					</div>
				{/if}

				<!-- handle all signal inlets -->
				{#if numInlets > 0}
					<div class="RNBOsection">
						<h2>signal inlets</h2>
						{#each inlets as inlet}
							<RNBOInlet {inlet} {device} />
						{/each}
					</div>
				{/if}

				<!-- handle all event inlets -->
				{#if numInports > 0}
					<div class="RNBOsection">
						<!-- list only the event inlets (skip signal ones) -->
						<h2>message inlets</h2>
						{#each inports as inport}
							<RNBOInport {inport} {device} />
						{/each}
					</div>
				{/if}

				<!-- handle all parameters -->
				{#if numParameters > 0}
					<div class="RNBOsection">
						<h2>parameters</h2>
						{#each parameters as parameter}
							<RNBOParam {parameter} />
						{/each}
					</div>
				{/if}

				<!-- handle all event outlets -->
				{#if numOutports > 0}
					<div class="RNBOsection">
						<h2>outport events</h2>
						{#each outports as outport}
							<RNBOOutport {outport} {device} />
						{/each}
					</div>
				{/if}

				<!-- midi outs not working yet (RNBO bug?????), exclude for now -->
				<!-- <div class="RNBOsection">
					<h2>MIDI outputs</h2>
					{#each Array(device.numMIDIOutputPorts) as _, port}
						<RNBOMidiOut {port} {device} />
					{/each}
				</div> -->
			</div>
		</slot>
	</div>
{:else}
	<!-- show a placeholder skeleton when loading -->
	<div class="RNBOsection">
		<div class="RNBOplaceholder" />
		<div class="RNBOsection">
			<div class="RNBOplaceholder" />
			<div class="RNBOsection">
				<div class="RNBOplaceholder" />
			</div>
		</div>
		<div class="RNBOsection">
			<div class="RNBOplaceholder" />
		</div>
		<div class="RNBOsection">
			<div class="RNBOplaceholder" />
		</div>
	</div>
{/if}
