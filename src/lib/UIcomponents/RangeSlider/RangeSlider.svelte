<script>
	export let name;
	export let value = 0;
	export let min = 0;
	export let max = 100;
	export let step = 0.01;
	export let label = '';
</script>

<input
	type="range"
	{name}
	class="RNBOslider"
	aria-label={label}
	{min}
	{max}
	{step}
	bind:value
	on:click
	on:change
	on:blur
/>
