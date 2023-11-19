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
	import path from 'path-browserify';

	/** @type {string} */
	export let dir = '/src/RNBO';
	/** @type {string} */
	export let patchName = 'patch.export';
	const dependencies = path.format({ dir, base: 'dependencies' });

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
	let dependencyFileCorrected = undefined;

	/** @type {import ('@rnbo/js').Parameter[]} */
	export let parameters = [];

	/** @type {import ('@rnbo/js').MessageInfo[]} */
	export let inports = [];

	/** @typedef {object} signalPort
	 * @property {number} index - the port index
	 * @property {string} tag - the tag
	 * @property {'signal'} type - the type
	 * @property {string} meta - the meta
	 */
	/** @type {signalPort[]} */
	export let inlets = [];

	/** @type {Array<Number>} */
	export let midiOutports = [];

	/** @type {import ('@rnbo/js').MessageInfo[]} */
	export let outports = [];

	/** @type {signalPort[]} */
	export let outlets = [];

	/** @type {Array<Number>} */
	export let midiInports = [];

	// set up device
	const deviceSetup = async () => {
		//import the patcher json dynamically!
		const patchPath = path.format({ dir, base: patchName });
		patcher = await import(/* @vite-ignore */ patchPath);

		if (patcher && context) {
			//import the dependency json dynamically!
			const dependencyFile = (await import(/* @vite-ignore */ dependencies)).default;

			dependencyFileCorrected = dependencyFile.map((dependency) => {
				if ('file' in dependency) {
					const newFile = path.join(dir, dependency.file);
					return Object.assign({}, dependency, { file: newFile });
				}
				return dependency;
			});

			device = await createDevice({ context, patcher });

			//load dependencies if they exist
			if (dependencyFileCorrected && dependencyFileCorrected.length) {
				device.loadDataBufferDependencies(dependencyFileCorrected);
			}
			if (patcher.presets && patcher.presets[0]) {
				device.setPreset(patcher.presets[0].preset);
			}
			device.node.connect(context.destination);

			parameters = device.parameters;

			inports = device.inports;

			// @ts-expect-error - desc.inlets does indeed exist
			inlets = patcher.desc.inlets.filter((inlet) => inlet.type === 'signal');

			midiInports = Array.from({ length: device.numMIDIInputPorts }, (_, i) => i + 1);

			outports = device.outports;

			// @ts-expect-error - desc.outlets does indeed exist
			outlets = patcher.desc.outlets.filter((outlet) => outlet.type === 'signal');

			midiOutports = Array.from({ length: device.numMIDIOutputPorts }, (_, i) => i + 1);
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
	<div
		on:click={() => context?.resume()}
		on:keydown={() => context?.resume()}
		role="button"
		tabindex="-2"
	>
		<slot
			{patcher}
			{device}
			{dir}
			{patchName}
			{dependencyFileCorrected}
			{parameters}
			{context}
			{inports}
			{inlets}
			{outports}
			{outlets}
			{midiInports}
			{midiOutports}
		>
			<div class="RNBOsection">
				<!-- use the json file name as header -->
				<h1>{patchName}</h1>

				<!-- create input for each MIDI input port -->
				{#if midiInports.length > 0}
					<div class="RNBOsection">
						<h2>MIDI inputs</h2>
						{#each midiInports as port}
							<RNBOMidiIn {port} {device} />
						{/each}
					</div>
				{/if}

				<!-- handle all signal inlets -->
				{#if inlets.length > 0}
					<div class="RNBOsection">
						<h2>signal inlets</h2>
						{#each inlets as inlet}
							<RNBOInlet {inlet} {device} />
						{/each}
					</div>
				{/if}

				<!-- handle all event inlets -->
				{#if inports.length > 0}
					<div class="RNBOsection">
						<!-- list only the event inlets (skip signal ones) -->
						<h2>message inlets</h2>
						{#each inports as inport}
							<RNBOInport {inport} {device} />
						{/each}
					</div>
				{/if}

				<!-- handle all parameters -->
				{#if parameters.length > 0}
					<div class="RNBOsection">
						<h2>parameters</h2>
						{#each parameters as parameter}
							<RNBOParam {parameter} />
						{/each}
					</div>
				{/if}

				<!-- handle all event outlets -->
				{#if outports.length > 0}
					<div class="RNBOsection">
						<h2>message outlets</h2>
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
