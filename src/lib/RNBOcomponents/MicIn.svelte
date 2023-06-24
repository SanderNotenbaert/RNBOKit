<script>
	//TODO: create a nice way to visualise the average volume!
	/** @type {AudioContext} */
	export let context;

	/** @type {MediaStream|null} */
	let stream;
	/** @type {AudioNode} */
	export let audio;

	let volumeCallback = null;
	/** @type {number|null} */
	let averageVolume = 0;

	/** @param {MediaStream} stream */
	const handleSuccess = (stream) => {
		const source = context.createMediaStreamSource(stream);
		let analyser = context.createAnalyser();
		analyser.fftSize = 512;
		analyser.minDecibels = -127;
		analyser.maxDecibels = 0;
		analyser.smoothingTimeConstant = 0.4;
		source.connect(analyser);
		const volumes = new Uint8Array(analyser.frequencyBinCount);
		//probe every 100ms for the average volume
		volumeCallback = setInterval(() => {
			analyser.getByteFrequencyData(volumes);
			let volumeSum = 0;
			for (const volume of volumes) volumeSum += volume;
			averageVolume = volumeSum / volumes.length;
			console.log(`brightness-[${averageVolume / 100}]`);
		}, 100);
		source.connect(analyser);
		audio = analyser;
		// analyser.connect(device.node, inlet.index - 1);
	};

	async function connectMic() {
		if (!stream) {
			stream = await navigator.mediaDevices.getUserMedia({ audio: true, video: false });
			handleSuccess(stream);
		} else {
			clearInterval(volumeCallback);
			volumeCallback = null;
			averageVolume = 0;
			stream.getTracks().forEach((track) => track.stop());
			stream = null;
		}
	}
</script>

<div class="RNBOsubcomponent RNBOalign" {...$$restProps}>
	<button on:click={connectMic} type="button" class="RNBObutton">
		<svg
			xmlns="http://www.w3.org/2000/svg"
			viewBox="0 0 352 512"
			class:RNBOfill={stream}
			class={'brightness-[0]'}
		>
			<path
				d="M176 352c53.02 0 96-42.98 96-96V96c0-53.02-42.98-96-96-96S80 42.98 80 96v160c0 53.02 42.98 96 96 96zm160-160h-16c-8.84 0-16 7.16-16 16v48c0 74.8-64.49 134.82-140.79 127.38C96.71 376.89 48 317.11 48 250.3V208c0-8.84-7.16-16-16-16H16c-8.84 0-16 7.16-16 16v40.16c0 89.64 63.97 169.55 152 181.69V464H96c-8.84 0-16 7.16-16 16v16c0 8.84 7.16 16 16 16h160c8.84 0 16-7.16 16-16v-16c0-8.84-7.16-16-16-16h-56v-33.77C285.71 418.47 352 344.9 352 256v-48c0-8.84-7.16-16-16-16z"
			/>
		</svg>
	</button>
	<p>{averageVolume}</p>
</div>

<style>
	.RNBOfill {
		fill: rgb(var(--local-accent-color));
		transition: var(--local-animation) var(--local-animation-duration);
		transition-timing-function: var(--local-animation-function);
	}
	.RNBObutton {
		width: 3rem;
		border-style: none;
		background-color: transparent;
		transition: var(--local-animation) var(--local-animation-duration);
		transition-timing-function: var(--local-animation-function);
	}
</style>
