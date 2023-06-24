<script>
	import { MIDIEvent, TimeNow } from '@rnbo/js';
	import XyMidiIn from './XYMidiIn.svelte';
	import ExtMidiIn from './ExtMidiIn.svelte';
	import RadioGroup from '$lib/UIcomponents/Radio/RadioGroup.svelte';
	import RadioItem from '$lib/UIcomponents/Radio/RadioItem.svelte';
	/** type {number} - the MIDI port index*/
	export let port = 0;
	/** @type {import ('@rnbo/js').Device} */
	export let device;
	/** @type {import('@rnbo/js').MIDIData|undefined} */
	export let midiMessage = undefined;
	/** @type {number|undefined} */
	export let midiChannel = 0;

	/** @type {{default: 'default', xy: 'xy', external: 'external'}} */
	let modes = { default: 'default', xy: 'xy', external: 'external' };
	/** @type {'default'|'xy'|'external'} */
	export let mode = modes.default;

	//set a defaultMode so the radio group only shows if no mode has been specified
	/** @type {'default'|'xy'|'external'} */
	let defaultMode = mode;
	//make sure the default mode is mic in the radio group
	if (mode == modes.default) mode = modes.xy;

	$: {
		if (midiMessage && midiMessage.length > 0) {
			console.log('from MidiIn', midiMessage);
			device.scheduleEvent(new MIDIEvent(TimeNow, port, midiMessage));
		}
	}
</script>

<div class="RNBOcomponent RNBOsection" {...$$restProps}>
	<slot {midiChannel}>
		<div class="RNBOtag">in {port}</div>
		{#if defaultMode === modes.default}
			<RadioGroup>
				<RadioItem bind:group={mode} name="xy" value={modes.xy}>XY-pad</RadioItem>
				<RadioItem bind:group={mode} name="external" value={modes.external}>ext. MIDI</RadioItem>
			</RadioGroup>
		{/if}

		{#if mode === modes.external}
			<ExtMidiIn bind:midiMessage {midiChannel} />
		{:else}
			<XyMidiIn bind:midiMessage {midiChannel} />
		{/if}
	</slot>
</div>
