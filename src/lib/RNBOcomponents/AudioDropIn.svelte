<script>
	//A file drop in/media player for audio
	import { flip } from 'svelte/animate';
	import { crossfade } from 'svelte/transition';
	const [send, receive] = crossfade({
		duration: (d) => Math.sqrt(d * 200)
	});

	import FileDropzone from '../UIcomponents/FileDropzone.svelte';

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
				// console.log('pause');
				audioElement.pause();
				continue;
			}
			// console.log('connect');
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
		audioElements = [];

		files.splice(index, 1);
		files = files;
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
		{#each files as file, i (file)}
			<div
				class="RNBOsection RNBOaudio"
				in:receive={{ key: file }}
				out:send={{ key: file }}
				animate:flip={{ duration: 200 }}
			>
				<p class="RNBOtext">{file.name}</p>
				<audio
					src={URL.createObjectURL(file)}
					bind:this={audioElements[i]}
					on:play={onPlay}
					controls
				/>
				<button
					type="button"
					class="RNBObtn RNBOclose"
					on:click={() => removeFile(i)}
					on:keydown={() => removeFile(i)}
				>
					x
				</button>
			</div>
		{/each}
	{/if}
</div>

<style>
	.RNBOclose {
		position: absolute;
		top: -0.5rem;
		right: -0.5rem;
		padding-top: 0.2rem;
		padding-bottom: 0.2rem;
		padding-left: 0.5rem;
		padding-right: 0.5rem;
		cursor: pointer;
		border-radius: var(--local-rounded-base);
		border-style: none;
		background-color: rgb(var(--local-accent-color));
	}
	.RNBOaudio {
		position: relative;
		display: inline-block;
		padding: 0.5rem;
		margin: 0.5rem;
	}
</style>
