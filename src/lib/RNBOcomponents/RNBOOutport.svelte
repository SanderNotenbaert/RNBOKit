<script>
	import ProgressBar from '$lib/UIcomponents/ProgressBar/ProgressBar.svelte';
	/** @type {import ('@rnbo/js').MessageInfo} */
	export let outport;

	/** @type {import ('@rnbo/js').Device} */
	export let device;

	/** @type {import ('@rnbo/js').MessagePayload } */
	let value = 0;

	/** @type {number} */
	export let min = 0;

	/** @type {number} */
	export let max = 1;

	$: device.messageEvent.subscribe((ev) => {
		if (ev.tag == outport.tag) {
			value = ev.payload;
		}
	});
</script>

<div class="RNBOcomponent RNBOsection" {...$$restProps}>
	<div class="RNBOalign">
		<div class="RNBOtag">{outport.tag}</div>
		<!-- some hack to make the value max 2 decimal floats: -->
		<div class="RNBOval">{+parseFloat(value).toFixed(2)}</div>
	</div>

	<p class="RNBOtext">{outport.meta.slice(1, -1)}</p>
	<ProgressBar value={Array.isArray(value) ? value[0] : value} {max} {min} />
</div>
