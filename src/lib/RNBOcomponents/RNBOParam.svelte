<script>
	import RangeSlider from '$lib/UIcomponents/RangeSlider/RangeSlider.svelte';
	import RadioGroup from '$lib/UIcomponents/Radio/RadioGroup.svelte';
	import RadioItem from '$lib/UIcomponents/Radio/RadioItem.svelte';
	/** @type {import ('@rnbo/js').Parameter} */
	export let parameter;
	/** @type {number} */
	let value = parameter.value;
	$: parameter.value = value;
</script>

<div class="RNBOcomponent RNBOsection" {...$$restProps}>
	{#if parameter.enumValues}
		<div class="RNBOtag">{parameter.name}</div>
		<RadioGroup>
			{#each parameter.enumValues as enumValue, i}
				<RadioItem bind:group={value} name="justify" value={i}>{enumValue}</RadioItem>
			{/each}
		</RadioGroup>
	{:else}
		<div class="RNBOalign">
			<div class="RNBOtag">{parameter.name}</div>
			<!-- some hack to make the value max 2 decimal floats: -->
			<div class="RNBOval">{+parseFloat(value).toFixed(2)} / {parameter.max}</div>
		</div>
		<RangeSlider
			name="range-slider"
			bind:value
			min={parameter.min}
			max={parameter.max}
			step={0.01}
		/>
	{/if}
</div>
