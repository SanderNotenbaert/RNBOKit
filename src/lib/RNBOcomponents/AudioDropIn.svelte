<script>
	//A file drop in/media player for audio
	//TODO: handle multiple files and only allow audio files
	import FileDropzone from '../UIcomponents/FileDropzone/FileDropzone.svelte';

	/** @type {AudioContext} */
	export let context;
	/** @type {AudioNode|undefined} */
	export let audio;

	/** @type {Array<File>} */
	let files = [];
	/** @type {HTMLMediaElement[]} */
	let audioElements = [];

	/** @type {HTMLMediaElement|undefined} */
	let current;

	/**
	 * pause other running audio and connect new playing source on play
	 * @param {Event} e
	 */
	function onPlay(e) {
		if (current === e.target) return;

		for (const audioElement of audioElements) {
			if (audioElement !== e.target) {
				console.log('pause');
				audioElement.pause();
				continue;
			}
			console.log('connect');
			current = audioElement;
		}
		try {
			audio = context.createMediaElementSource(current);
		} catch (e) {
			//ignore DOMexception error, as the correct source will still be used,
			// no easy way around arror (need to investigate)
			console.log('failed to (re)connect MediaElementSource');
		}
	}

	/**
	 * put files in an array, so we can do array operations like splice. e.target.files is read-only
	 * @param {Event} e
	 */
	function loadFiles(e) {
		// @ts-expect-error - quick fix for: TS2339: Property 'files' does not exist on type 'Event'.
		files = [...e.target.files];
	}

	/**
	 * remove a file from the files array when we click the 'x'. If no more files,
	 * show audio drop in again
	 * @param {Number} index
	 */
	function removeFile(index) {
		if (!files) return;
		console.log(files);
		console.log(index);
		audioElements = [];

		files.splice(index, 1);
		files = files;
		console.log(files);
		if (files.length === 0) {
			files = [];
			audioElements = [];
		}
	}
</script>

<div class="RNBOsubcomponent" {...$$restProps}>
	{#if files.length === 0}
		<FileDropzone name="audio files" on:change={loadFiles} multiple accept="audio/*" />
	{:else}
		{#each files as file, i}
			<button
				type="button"
				class="RNBObtn"
				on:click={() => removeFile(i)}
				on:keydown={() => removeFile(i)}
			>
				x
			</button>
			<audio
				src={URL.createObjectURL(file)}
				bind:this={audioElements[i]}
				on:play={onPlay}
				controls
			/>
		{/each}
	{/if}
</div>
