<script>
	import { MIDIEvent, TimeNow } from '@rnbo/js';
	// import ExtMidiOut from './ExtMidiOut.svelte';
	/** type {number} - the MIDI port index*/
	export let port = 0;
	/** @type {import ('@rnbo/js').Device} */
	export let device;
	/** @type {import('@rnbo/js').MIDIData|undefined} */
	export let midiMessage = new MIDIEvent(TimeNow, 0, [144, 100, 100]);
	// /** @type {number|undefined} */
	// export let midiChannel = 0;

	//TODO: replace block below with a midiMessage receive from a Max Midi out, send to extMidiOut
	device.midiEvent.subscribe((ev) => {
		console.log(ev);
	});
</script>

<div class="RNBOcomponent RNBOsection" {...$$restProps}>
	<slot {port}>
		<div class="RNBOtag">in {port}</div>

		<!-- <ExtMidiOut bind:midiMessage /> -->
	</slot>
</div>
