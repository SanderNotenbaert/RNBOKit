<img width="493" alt="Screenshot 2023-06-26 at 22 35 49" src="https://github.com/SanderNotenbaert/RNBOKit/assets/34664737/a8394d39-6c23-49c4-bd6b-a977f5bd0cff">

# RNBOKit

An easily extendable Sveltekit component library for seamlessly importing Cycling74's Max RNBO~ patches into your Svelte app.

## Disclaimer

~ warning #1: this package is still work in progress ~

~ warning #2: this is my very first library & open source project, mistakes are bound to be made, so please do correct me on any mistake or practice I can do better! ~

## Getting Started

First set up a [Sveltekit project](https://kit.svelte.dev/docs/quick-start).

Once you've created and installed your project add RNBOKit to your packages

```bash
npm install rnbokit
```

Or using pnpm:

```bash
pnpm install rnbokit
```

## Using RNBOKit in your project

To start running RNBO patches in your Sveltekit project, you will need to export your RNBO~ patch for the web.

<img width="597" alt="Screenshot 2023-05-26 at 17 20 39" src="https://github.com/SanderNotenbaert/RNBOKit/assets/34664737/c48b537f-79a1-4a4f-beef-cc5bf72addfc">

It will create a folder which contains a .JSON file. You can export it directly into your project directory.

<img width="567" alt="Screenshot 2023-05-26 at 17 22 03" src="https://github.com/SanderNotenbaert/RNBOKit/assets/34664737/534588d2-7bfe-4fa3-b686-3a2d5dca54d4">

By default the <RNBO> component will look for a `patch.export.json` file in a `src/RNBO/` folder.
Next you will need to import the RNBO component into your `+page.svelte` file.

```svelte
<script>
	import { RNBO } from 'rnbokit';
</script>

<RNBO />
```

This is enough to start running the RNBO patch in your Sveltekit project. It will generate all the parameters and visualise any number messages from the outlet (0.-1.)

## opening different patches

to open different RNBO patch exports, export them to different directories, and use the built-in `dir` prop in the RNBO components to open them:

```svelte
<script>
	import { RNBO } from 'rnbokit';
</script>

<RNBO dir="/src/RNBO/Patch1" />
<RNBO dir="/src/RNBO/Patch2" />
```

## extending with custom slots

You can customize how parameters or messages are handled by adding your own components in the RNBO tag. You can pass information from the RNBO component down to the child components by using Svelte's `let:` syntax. Some available objects are the `patcher` file (IPatcher), `dependencyFileCorrected` (ExternalDataInfo[]), `context` (AudioContext), `device` (Device), or `parameters` (Parameter[]). You can find more information on these in the [RNBO JS documentation](https://rnbo.cycling74.com/js).

```svelte
<script>
	import { RNBO, RNBOParam } from 'rnbokit';
</script>

<RNBO let:parameters let:patcher>
	<p>used Max version: {patcher.desc.meta.maxversion}</p>
	{#each parameters as parameter}
		<!-- create custom components -->
		<p>{parameter.name}</p>
		<p>{parameter.value}</p>
		<!-- or use the included UI components -->
		<RNBOParam {parameter} />
	{/each}
</RNBO>
```

Here is the full list of all available variables:

```svelte
<script>
	import { RNBO } from 'rnbokit';
</script>

<RNBO let:patcher <!-- type: IPatcher - represents the json exported from Max -->
	let:device <!-- type: Device - the device object, check RNBO js documentation -->
	let:dir <!-- type: string - the directory the patch was exported to -->
	let:patchName <!-- type: string - the name of the patch file -->
	let:dependencyFileCorrected <!-- type: ExternalDataInfo[] - represents the json with dependencies of the patch, check RNBO js documentation -->
	let:parameters <!-- type: Parameter[] - an array containing all RNBO parameters, check RNBO js documentation -->
	let:context <!-- type: AudioContext - the audio context, see https://developer.mozilla.org/Web/API/AudioContext -->
	let:inports <!-- type: MessageInfo[] - an array of objects containing info on each message inlet -->
	let:inlets <!-- type: signalPort[] - an array of objects containing info on each signal inlet -->
	let:outports <!-- type: MessageInfo[] - an array of objects containing info on each message outlet -->
	let:outlets <!-- type: signalPort[] - an array of objects containing info on each signal outlet -->
	let:midiInports <!-- type: Array<Number> - an array representing each midiInport as an index number -->
	let:midiOutports <!-- type: Array<Number> - an array representing each midiOutport as an index number -->
	>
	<!-- do something with it here -->
</RNBO>
```

In order to be able to change parameter values or send messages to inlets with custom parameter components, you'll need to bind `parameters` or `device` in the `<RNBO\>` component to a declared variable in the top level script. To control the value, bind the value of your own component to it as well.

```svelte
<script>
	import { RNBO } from 'rnbokit';
	//this example uses JSDoc for typing
	/** @type {import ('@rnbo/js').Parameter[]} */
	let patchParams;
</script>

<RNBO bind:parameters={patchParams}>
	<div class="slidecontainer">
		<input
			type="range"
			min={patchParams[1].min}
			max={patchParams[1].max}
			id={patchParams[1].name}
			bind:value={patchParams[1].value}
			class="RNBOslider"
		/>
		<p class="RNBOval">{patchParams[1].value}</p>
	</div>
</RNBO>
```

## Available components

### RNBO specific components

Here is the list of available RNBO components, to be used as slots of the `<RNBO>` component:

```javascript
import { RNBOParam } from 'rnbokit'; //required props: {parameters}
import { RNBOInport } from 'rnbokit'; //required props: {inport} {device} - optional props: {min}, {max}
import { RNBOOutport } from 'rnbokit'; //required props: {outport} {device} - optional props: {min}, {max}
import { RNBOInlet } from 'rnbokit'; //required props: {inlet} {device} - optional props: {mode} options: 'mic', 'dropIn'
import { RNBOMidiIn } from 'rnbokit'; //required props: {port} {device} - optional props: {mode} options: 'xy', 'external'
//not working, RNBO bug
//import { RNBOMidiOut } from 'rnbokit'; //required props: {port} {device}
```

### Utility components

These are some internally used utility components which are not dependent on RNBO:

```javascript
import { AudioDropIn } from 'rnbokit';
import { MicIn } from 'rnbokit';
import { XYMidiIn } from 'rnbokit';
import { ExtMidiIn } from 'rnbokit';
// work in progress
// import { ExtMidiOut } from 'rnbokit';
```

### UI components

Internally used UI components, that can also be reused in different contexts:

```javascript
import { FileDropZone } from 'rnbokit';
import { ProgressBar } from 'rnbokit';
import { Radiogroup, RadioItem } from 'rnbokit';
import { RangeSlider } from 'rnbokit';
```

## Styling

The default styling is done with the internal RNBO.css stylesheet. If you want to change really fundamental styling you can edit/replace that one, but you can also simply declare the following css variables in your own global stylesheet:

```css
:root {
	/* =~= Theme Properties =~= */
	--theme-font-family: system-ui;
	--theme-rounded-base: 9999px;
	--theme-rounded-container: 6px;
	--theme-border-base: 1px;
	/* =~= Color Properties =~= */
	--theme-accent-color: 39, 145, 142;
	--theme-surface-color: 38, 38, 38;
	--theme-bg-color: 255, 255, 255;
	--theme-font-color-base: 38, 38, 38;
	/* =~= Animation Properties =~= */
	--theme-animation-duration: 0.1s;
	--theme-animation: all;
	--theme-animation-function: ease-out;
}
```

## Defaults

Here is what it could look like if you'd recreate the RNBO component's default slot, for inspiration:

```svelte
<RNBO let:patchName let:device let:midiInports let:inlets let:inports let:parameters let:outports>
	<div class="RNBOsection">
		<!-- use the json file name as header -->
		<h1>{patchName}</h1>

		<!-- create input for each MIDI input port -->
		{#if midiInports.length > 0}
			<div class="RNBOsection">
				<h2>MIDI inputs</h2>
				{#each midiInports as port}
					<RNBOMidiIn {port} {device} />
				{/each}
			</div>
		{/if}

		<!-- handle all signal inlets -->
		{#if inlets.length > 0}
			<div class="RNBOsection">
				<h2>signal inlets</h2>
				{#each inlets as inlet}
					<RNBOInlet {inlet} {device} />
				{/each}
			</div>
		{/if}

		<!-- handle all event inlets -->
		{#if inports.length > 0}
			<div class="RNBOsection">
				<!-- list only the event inlets (skip signal ones) -->
				<h2>message inlets</h2>
				{#each inports as inport}
					<RNBOInport {inport} {device} />
				{/each}
			</div>
		{/if}

		<!-- handle all parameters -->
		{#if parameters.length > 0}
			<div class="RNBOsection">
				<h2>parameters</h2>
				{#each parameters as parameter}
					<RNBOParam {parameter} />
				{/each}
			</div>
		{/if}

		<!-- handle all event outlets -->
		{#if outports.length > 0}
			<div class="RNBOsection">
				<h2>outport events</h2>
				{#each outports as outport}
					<RNBOOutport {outport} {device} />
				{/each}
			</div>
		{/if}
	</div>
</RNBO>
```

## TODO

- [x] add inlets & outlets in default RNBO component
- [x] add handling of midi in & out in default RNBO component
- [x] create reusable UI components (external midi in & out, audio dropzone)
- [x] get rid of Skeleton/tailwind dependencies?
- [ ] add external midi out. (doesn't work due to apparent RNBO bug, waiting for tech support)
