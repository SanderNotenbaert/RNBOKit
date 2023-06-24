<script>
	export let group;
	export let name;
	export let value;
	export let title = '';
	export let label = '';
	let elemInput;
	function onKeyDown(event) {
		if (['Enter', 'Space'].includes(event.code)) {
			event.preventDefault();
			elemInput.click();
		}
	}
	$: checked = value === group;
</script>

<label>
	<!-- A11y attributes are not allowed on <label> -->
	<div
		class="RNBOradio-item {checked ? 'RNBOchecked' : 'RNBOhover'}"
		data-testid="radio-item"
		role="radio"
		aria-checked={checked}
		aria-label={label}
		tabindex="0"
		{title}
		on:keydown={onKeyDown}
		on:keydown
		on:keyup
		on:keypress
	>
		<!-- NOTE: Don't use `hidden` as it prevents `required` from operating -->
		<div class="RNBOradio-input">
			<input
				bind:this={elemInput}
				type="radio"
				bind:group
				{name}
				{value}
				tabindex="-1"
				on:click
				on:change
				id="test"
			/>
		</div>
		<slot />
	</div>
</label>

<style>
	.RNBOradio-item {
		font-size: 1rem;
		line-height: 1.5rem;
		text-align: center;
		flex: 1 1 auto;
		cursor: pointer;
		padding: 0.2rem;
		padding-left: 0.75rem;
		padding-right: 0.75rem;
		margin: 0.25rem;
		background-color: rgba(var(--local-accent-color), 0);
		border-radius: var(--local-rounded-base);
	}
	.RNBOhover:hover {
		background-color: rgba(var(--local-accent-color), 0.2);
		transition: var(--local-animation) var(--local-animation-duration);
	}
	.RNBOchecked {
		background-color: rgba(var(--local-accent-color), 1);
		transition: var(--local-animation) var(--local-animation-duration);
	}
	.RNBOradio-input {
		overflow: hidden;
		width: 0;
		height: 0;
	}
</style>
