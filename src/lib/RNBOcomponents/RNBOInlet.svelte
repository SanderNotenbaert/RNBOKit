<script>
	import RadioGroup from '$lib/UIcomponents/RadioGroup.svelte';
	import RadioItem from '$lib/UIcomponents/RadioItem.svelte';
	import AudioDropIn from './AudioDropIn.svelte';
	import MicIn from './MicIn.svelte';

	/** @type {import ('@rnbo/js').Device} */
	export let device;
	/** @type {AudioContext} */
	let context = device.context;
	/** @typedef {object} inlet
	 * @property {number} index - the port index
	 * @property {string} tag - the tag
	 * @property {'signal'} type - the type
	 * @property {string} meta - the meta
	 */
	/** @type {inlet} */
	export let inlet = { index: 1, tag: 'in1', type: 'signal', meta: 'something' };

	/** @type {AudioNode} */
	let audio;

	/** @type {{default: 'default', mic: 'mic', dropIn: 'dropIn'}} */
	let modes = { default: 'default', mic: 'mic', dropIn: 'dropIn' };
	/** @type {'default'|'mic'|'dropIn'} */
	export let mode = modes.default;
	//set a defaultMode so the radio group only shows if no mode has been specified
	/** @type {'default'|'mic'|'dropIn'} */
	let defaultMode = mode;
	//make sure the default mode is mic in the radio group
	if (mode == modes.default) mode = modes.mic;

	$: {
		if (audio) {
			audio.connect(device.node, inlet.index - 1);
		}
	}
</script>

<div class="RNBOcomponent RNBOsection" {...$$restProps}>
	<slot {context} {audio}>
		<div class="RNBOtag">{inlet.tag}</div>
		{#if defaultMode === modes.default}
			<RadioGroup>
				<RadioItem bind:group={mode} name="Mic" value={modes.mic}>Mic</RadioItem>
				<RadioItem bind:group={mode} name="DropIn" value={modes.dropIn}>Drop-in</RadioItem>
			</RadioGroup>
		{/if}
		{#if mode === modes.dropIn}
			<AudioDropIn class="RNBOsubcomponent" bind:audio {context} />
		{:else}
			<MicIn bind:audio {context} />
		{/if}
	</slot>
</div>
