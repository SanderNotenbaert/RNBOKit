<script>
	/** @type {number} - the MIDI channel (0-15)*/
	export let midiChannel = 0;
	/** @type {Uint8Array} */
	let currentNote = new Uint8Array([0, 0]);
	/** @type {Uint8Array} */
	export let midiMessage;
	let isOn = false;

	/** @param {MouseEvent} e */
	function NoteOn(e) {
		// @ts-ignore - ts not knowing getBoundingClientRect()
		const rect = e.currentTarget.getBoundingClientRect();
		currentNote[0] = Math.round(((e.clientX - rect.left) / rect.width) * 127);
		currentNote[1] = Math.round((1 - (e.clientY - rect.top) / rect.height) * 127);
		midiMessage = new Uint8Array([midiChannel + 144, ...currentNote]);
		isOn = true;
		// console.log('from XYMidiIn', midiMessage);
	}
	function NoteOff() {
		if (!isOn) return;
		// currentNote[1] = 0;
		midiMessage = new Uint8Array([midiChannel + 128, ...currentNote]);
		isOn = false;
	}
	//preview the note and velocity value on hover
	/** @param {MouseEvent} e */
	function NoteUpdate(e) {
		if (isOn) return;
		// @ts-ignore - ts not knowing getBoundingClientRect()
		const rect = e.currentTarget.getBoundingClientRect();
		currentNote[0] = Math.round(((e.clientX - rect.left) / rect.width) * 127);
		currentNote[1] = Math.round((1 - (e.clientY - rect.top) / rect.height) * 127);
	}
</script>

<svelte:window on:mouseup={NoteOff} />
<div class="RNBOsubcomponent" {...$$restProps}>
	<p class="RNBOtext">click the pad to generate MIDI notes</p>
	<!-- TODO: fix accessibility roles, quick hack for now -->
	<div
		class="xy"
		id="group"
		on:mousedown={NoteOn}
		on:mousemove={NoteUpdate}
		on:keydown={NoteOn}
		role="none"
	>
		<p class="xyval" class:text-gray-500={!isOn}>
			note: {currentNote[0]} | velocity: {currentNote[1]}
		</p>
	</div>
</div>

<style>
	.xy {
		/* @apply group card flex align-center justify-center p-4 m-4 max-w-md min-h-max select-none hover:bg-primary-hover-token active:bg-primary-active-token */
		display: flex;
		justify-content: center;
		align-items: center;
		padding: 5rem;
		margin-top: 1rem;
		max-width: 28rem /* 448px */;
		min-height: max-content;
		user-select: none;
		background-color: rgba(var(--local-accent-color), 0.1);
		border-radius: var(--local-rounded-container);
		transition: var(--local-animation) var(--local-animation-duration);
		transition-timing-function: var(--local-animation-function);
	}
	.xy:hover {
		background-color: rgba(var(--local-accent-color), 0.3);
		transition: var(--local-animation) var(--local-animation-duration);
		transition-timing-function: var(--local-animation-function);
	}
	.xy:active {
		background-color: rgb(var(--local-accent-color)) !important;
		/* color: black; */
		fill: rgb(var(--local-accent-color));
		transition: var(--local-animation) calc(var(--local-animation-duration) / 3);
		transition-timing-function: var(--local-animation-function);
	}
	.xy .xyval {
		/* visibility: hidden; */
		color: rgba(var(--local-font-color), 0);
	}
	.xy:hover .xyval {
		/* color: blue; */
		color: rgba(var(--local-font-color), 0.5);
	}
	.xy:active .xyval {
		/* color: black; */
		color: rgb(var(--local-font-color));
	}
</style>
