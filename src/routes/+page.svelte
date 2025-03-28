<script lang="ts">
	import { Input } from "$lib/components/ui/input";

	let audioContext: AudioContext | null = null;
	let audioElement: HTMLAudioElement | null = null;

	if (typeof window !== "undefined") {
		audioContext = new AudioContext();
	}

	let track: MediaElementAudioSourceNode;

	let gainNode: GainNode | null = null;

	// const pannerOptions = { pan: 0 };
	let panner: StereoPannerNode | null = null;

	function initializeAudio() {
		if (audioContext && audioElement) {
			gainNode = audioContext.createGain();
			panner = new StereoPannerNode(audioContext, { pan: 0 });

			track = audioContext.createMediaElementSource(audioElement);
			track.connect(gainNode).connect(panner).connect(audioContext.destination);
		}
	}

	let playButton: HTMLElement;

	let buttonIcon: string = "▶";

	function playAudio() {
		if (!audioContext || !track || !audioElement) return;

		if (audioContext.state === "suspended") {
			audioContext.resume();
		}

		if (playButton.dataset.playing === "false") {
			audioElement.play();
			playButton.dataset.playing = "true";
			buttonIcon = "❚❚";
		} else if (playButton.dataset.playing === "true") {
			audioElement.pause();
			playButton.dataset.playing = "false";
			buttonIcon = "▶";
		}
	}

	let volume: number;
	let pan: number;

	function volumeControl() {
		if (gainNode) {
			gainNode.gain.value = volume;
			// console.log("volume: ", gainNode.gain.value);
		}
	}

	function pannerControl() {
		if (panner) {
			panner.pan.value = pan;
			// console.log("pan: ", panner.pan.value);
		}
	}

	// -----------------------------------------------------------------------------------------------

	let svg: any;
	let draggableCircle: any;
	let fixedCircle: any;
	let lineLeft: any;
	let lineRight: any;

	let mouseX: number;
	let mouseY: number;

	let fixedCircleX: number;
	let fixedCircleY: number;

	let dragging = false;

	let distanceLeft: string;
	let distanceRight: string;

	function moveCircle() {
		fixedCircleX = parseFloat(fixedCircle.getAttribute("cx"));
		fixedCircleY = parseFloat(fixedCircle.getAttribute("cy"));

		if (dragging) {
			draggableCircle.setAttribute("cx", mouseX);
			draggableCircle.setAttribute("cy", mouseY);

			lineLeft.setAttribute("x1", mouseX);
			lineLeft.setAttribute("y1", mouseY);
			lineLeft.setAttribute("x2", fixedCircleX - 20);
			lineLeft.setAttribute("y2", fixedCircleY);

			lineRight.setAttribute("x1", mouseX);
			lineRight.setAttribute("y1", mouseY);
			lineRight.setAttribute("x2", fixedCircleX + 20);
			lineRight.setAttribute("y2", fixedCircleY);
		}
	}

	function getDistance() {
		if (dragging) {
			distanceLeft = Math.sqrt(
				(mouseX - (fixedCircleX - 20)) ** 2 + (mouseY - fixedCircleY) ** 2
			).toFixed(2);
			distanceRight = Math.sqrt(
				(mouseX - (fixedCircleX + 20)) ** 2 + (mouseY - fixedCircleY) ** 2
			).toFixed(2);

			volume = 2 * (1 - (Math.sqrt((mouseX - fixedCircleX) ** 2 + (mouseY - fixedCircleY) ** 2) / 400));
			pan = (mouseX - fixedCircleX) / 200;

			volumeControl();
			pannerControl();

			return `Left: ${distanceLeft}, Right: ${distanceRight}`;
		}
	}

	function moveMouse(e: MouseEvent | TouchEvent) {
		if (e instanceof MouseEvent) {
			mouseX = e.pageX - (window.innerWidth / 2) + 200;
			mouseY = e.pageY - 77;
		} else if (e.touches && e.touches.length > 0) {
			mouseX = e.touches[0].pageX - (window.innerWidth / 2) + 200;
			mouseY = e.touches[0].pageY - 77;
		}
	
		moveCircle();
		getDistance();
	}
</script>

<!-- svelte-ignore a11y_no_static_element_interactions -->
<svg
	id="mySVG"
	class="border-2 border-black"
	bind:this={svg}
	width="400"
	height="400"
	on:mousemove={moveMouse}
	on:touchmove={moveMouse}
>
	<!-- Fixed circle at the bottom middle -->
	<circle bind:this={fixedCircle} cx="200" cy="350" r="20" fill="blue" id="fixedCircle" />

	<!-- Draggable circle in the center -->
	<!-- svelte-ignore a11y_no_static_element_interactions -->
	<circle
		bind:this={draggableCircle}
		on:mousedown={() => {
			dragging = true;
		}}
		on:mouseup={() => {
			dragging = false;
		}}

		on:touchstart={() => {
			dragging = true;
		}}
		on:touchend={() => {
			dragging = false;
		}}
		cx="200"
		cy="200"
		r="20"
		fill="red"
		class="draggable"
		id="draggableCircle"
	/>

	<!-- Dotted lines -->
	<line bind:this={lineLeft} id="lineLeft" x1="200" y1="200" x2="200" y2="350" stroke="black" />
	<line bind:this={lineRight} id="lineRight" x1="200" y1="200" x2="200" y2="350" stroke="black" />
</svg>

<h1><b>Distance Left:</b> {distanceLeft}</h1>
<h1><b>Distance Right:</b> {distanceRight}</h1>

<div class="flex mt-5">
	<button
		class="ring-offset-background focus-visible:ring-ring inline-flex h-10 items-center justify-center rounded-md bg-slate-900 px-4 py-2 text-sm font-medium whitespace-nowrap text-slate-50 transition-colors focus-visible:ring-2 focus-visible:ring-offset-2 focus-visible:outline-none disabled:pointer-events-none disabled:opacity-50"
		data-playing="false"
		role="switch"
		aria-checked="false"
		bind:this={playButton}
		on:click={playAudio}
	>
		{buttonIcon}
		<!-- Play/Pause -->
	</button>
	
	<Input
		class="w-89"
		type="file"
		on:change={(e) => {
			const file = (e.target as HTMLInputElement)?.files?.[0];
			if (file) {
				const url = URL.createObjectURL(file);
				if (audioElement) {
					audioElement.src = url;
					initializeAudio();
				}
			}
		}}
	/>
</div>

<audio bind:this={audioElement} hidden></audio>


<label class="mt-5" for="volume">Volume</label>
<input
type="range"
id="volume"
class="ml-1 mt-5"
min="0"
max="2"
step="0.01"
bind:value={volume}
on:input={volumeControl}
/>
<br>
<label class="mt-2" for="panner">Panner</label>
<input
	type="range"
	id="panner"
	class="ml-1"
	min="-1"
	max="1"
	step="0.01"
	bind:value={pan}
	on:input={pannerControl}
/>
