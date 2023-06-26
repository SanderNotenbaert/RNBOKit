<script>
	export let files = void 0;
	export let name;
	let dragOver = false;
</script>

<div class="RNBOdropzone" data-testid="file-dropzone">
	<!-- Input: File (hidden) -->
	<!-- NOTE: keep `bind:files` here, unlike FileButton -->
	<input
		bind:files
		type="file"
		{name}
		{...$$restProps}
		class="RNBOdropzone-input"
		class:RNBOdrag={dragOver}
		on:dragenter={() => {
			dragOver = true;
		}}
		on:dragleave={() => {
			dragOver = false;
		}}
		on:change
		on:dragenter
		on:dragover
		on:dragleave
		on:drop
		on:click
		on:keydown
		on:keyup
		on:keypress
	/>
	<!-- Interface -->
	<div class="RNBOdropzone-interface">
		<div class="dropzone-interface-text">
			<!-- Lead -->
			{#if $$slots.lead}<div class="RNBOdropzone-lead"><slot name="lead" /></div>{/if}
			<!-- Message -->
			<div class="RNBOdropzone-message">
				<slot name="message"><strong>Upload files</strong> or drag and drop</slot>
			</div>
			<!-- Meta Text -->
			{#if $$slots.meta}<small class="RNBOdropzone-meta"><slot name="meta" /></small>{/if}
		</div>
	</div>
</div>

<style>
	.RNBOdropzone {
		display: flex;
		position: relative;
		padding: 1rem;
		padding-top: 2rem;
		padding-bottom: 2rem;
		justify-content: center;
		align-items: center;
		border-width: 2px;
		border-style: dashed;
		border-radius: var(--local-rounded-container);
		border-color: rgb(var(--local-surface-color));
		transition: var(--local-animation) var(--local-animation-duration);
		transition-timing-function: var(--local-animation-function);
	}
	.RNBOdrag {
		background-color: rgba(var(--local-accent-color), 0.2);
		transition: var(--local-animation) var(--local-animation-duration);
		transition-timing-function: var(--local-animation-function);
	}
	.RNBOdropzone:hover {
		background-color: rgba(var(--local-accent-color), 0.2);
		transition: var(--local-animation) var(--local-animation-duration);
		transition-timing-function: var(--local-animation-function);
	}
	.RNBOdropzone:active {
		background-color: rgba(var(--local-accent-color), 0.5);
		transition: var(--local-animation) var(--local-animation-duration);
		transition-timing-function: var(--local-animation-function);
	}
	.RNBOdropzone-input {
		position: absolute;
		top: 0;
		right: 0;
		bottom: 0;
		left: 0;
		width: 100%;
		cursor: pointer;
		opacity: 0;
	}
	.RNBOdropzone-interface {
		display: flex;
		text-align: center;
		justify-content: center;
		align-items: center;
	}
	/* .RNBOdropzone-interface-text { */
	/* } */
	.RNBOdropzone-meta {
		opacity: 0.75;
	}
	.RNBOdropzone-lead {
		margin-bottom: 1rem;
	}
</style>
