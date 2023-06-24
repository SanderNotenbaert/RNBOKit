<script>
	import '$lib/RNBO.css';
	// import RangeSlider from '../UIcomponents/RangeSlider/RangeSlider.svelte';
	import { TimeNow, MessageEvent } from '@rnbo/js';

	/** @type {import ('@rnbo/js').MessageInfo} */
	export let inport;
	/** @type {import ('@rnbo/js').Device} */
	export let device;
	/** @type {number[]} */
	export let value = [];
	/** @type {number} */
	export let min = 0;
	/** @type {number} */
	export let max = 1;

	$: device.scheduleEvent(new MessageEvent(TimeNow, inport.tag, value));
</script>

<!-- only show slider if value is controlled from within the component, otherwise only use the component to change the value -->
{#if value.length < 2}
	<div class="RNBOcomponent RNBOsection" {...$$restProps}>
		<div class="RNBOalign">
			<div class="RNBOtag">{inport.tag}</div>
			<div class="RNBOval">{value} / {max}</div>
		</div>
		<p>{inport.meta.slice(1, -1)}</p>
		<div>
			<!-- <RangeSlider name="range-slider" bind:value={value[0]} {min} {max} step={0.01} /> -->

			<input
				type="range"
				name="range-slider"
				class="RNBOslider"
				aria-label={''}
				{min}
				{max}
				step={0.01}
				bind:value={value[0]}
				on:click
				on:change
				on:blur
			/>
		</div>
	</div>
{/if}
