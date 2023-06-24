<script>
	import RadioGroup from '$lib/UIcomponents/Radio/RadioGroup.svelte';
	import RadioItem from '$lib/UIcomponents/Radio/RadioItem.svelte';
	/** @type {MIDIAccess|null|false} */
	let midi = null; // global MIDIAccess object
	/**@type {MIDIInputMap|null} */
	let outports = null; // available MIDI outports

	//TODO: choose port by name rather than index

	/** @type {number|null} */
	export let port = null;
	/** @type {MIDIInput|null} */
	let outport = null;
	/** @type {Uint8Array} */
	export let midiMessage;

	/**
	 * set the midi access point and get the midi output port
	 * @param {MIDIAccess} midiAccess
	 */
	function onMIDISuccess(midiAccess) {
		midi = midiAccess;
		outports = midi.outputs; // store in the global (in real usage, would probably keep in an object instance)
		if (port) {
			setPort();
		}
	}

	/**
	 * handle if external MIDI not available
	 * @param {string} msg
	 */
	function onMIDIFailure(msg) {
		console.error(`Failed to get MIDI access - ${msg}`);
		midi = false;
	}

	/**
	 * set the output port when selected
	 */
	function setPort() {
		if (!outports || !port) return;

		// @ts-ignore - ts getting their types wrong...
		outport = [...outports][port];
		console.log(
			// @ts-ignore - because outports will always exist, since we use the #if block
			'connecting to MIDI output port: %c' + outports.get(outport[0]).name,
			'font-style:italic'
		);
		// @ts-ignore - see previous comment
		outports.get(outport[0]).onmidimessage = onMIDIMessage;
	}

	/** @param {MIDIMessageEvent} event */
	function onMIDIMessage(event) {
		midiMessage = event.data;
		console.log(
			'MIDI message from %c' + event.target.name + ': ' + midiMessage,
			'font-style:italic'
		);
	}

	navigator.requestMIDIAccess().then(onMIDISuccess, onMIDIFailure);
</script>

<div class="RNBOsubcomponent">
	<p class="RNBOtext">External MIDI outputs</p>

	{#if midi && outports}
		<!-- <slot {midi} {outports} {port}> -->
		<RadioGroup>
			{#each [...outports.entries()] as output, i}
				<!-- first index of the output array is the id, second the object -->
				<RadioItem bind:group={port} name="outports" value={i} on:change={setPort}
					>{output[1].name}
				</RadioItem>
			{/each}
		</RadioGroup>
		<!-- </slot> -->
	{:else if midi === false}
		<p>MIDI access failed, please try in a different browser</p>
	{:else}
		<p>loading MIDI ports</p>
	{/if}
</div>
