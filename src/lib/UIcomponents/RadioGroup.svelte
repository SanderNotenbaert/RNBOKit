<script>
	export let labelledby = '';
</script>

<div
	class="RNBOradio-group"
	data-testid="radio-group"
	role="radiogroup"
	aria-labelledby={labelledby}
>
	<slot />
</div>

<style>
	.RNBOradio-group {
		display: inline-flex;
		flex-wrap: wrap;
		margin-left: 0.25rem;
		background-color: rgba(var(--local-surface-color), 0);
		border-width: var(--local-border-base);
		border-color: rgb(var(--local-surface-color));
		border-style: solid;
		border-radius: var(--local-rounded-base);
	}
	/* .RNBOradio-group:hover { */
	/* 	background-color: rgba(var(--local-surface-color), 0.2); */
	/* } */
</style>
