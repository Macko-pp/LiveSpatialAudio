<script lang="ts">
	import { Input } from "$lib/components/ui/input";

	let audioContext: AudioContext | null = null;
	let audioElement: HTMLAudioElement | null = null;

	if (typeof window !== "undefined") {
		audioContext = new AudioContext();
	}

	let track: MediaElementAudioSourceNode;

	let listener: AudioListener;
	let panner3D: PannerNode;

	let pan3DX: number;
	let pan3DY: number;

	function initializeAudio() {
		if (audioContext && audioElement) {
			listener = audioContext.listener;
			listener.positionX.value = 200;
			listener.positionY.value = 200;

			listener.forwardX.value = 0;
			listener.forwardY.value = 20; // radius of head
			listener.forwardZ.value = 0;
			listener.upX.value = 0;
			listener.upY.value = 0;
			listener.upZ.value = 20;

			const panningModel = "HRTF";
			const innerCone = 360;
			const outerCone = 360;
			const outerGain = 1;
			const distanceModel = "exponential";
			const maxDistance = 10000;
			const refDistance = 50;
			const rollOff = 1;
			const positionX = pan3DX;
			const positionY = pan3DY;
			const positionZ = 0;
			const orientationX = 0.0;
			const orientationY = 0.0;
			const orientationZ = 0.0;

			panner3D = new PannerNode(audioContext, {
				panningModel,
				distanceModel,
				positionX,
				positionY,
				positionZ,
				orientationX,
				orientationY,
				orientationZ,
				refDistance,
				maxDistance,
				rolloffFactor: rollOff,
				coneInnerAngle: innerCone,
				coneOuterAngle: outerCone,
				coneOuterGain: outerGain
			});

			track = audioContext.createMediaElementSource(audioElement);
			track.connect(panner3D).connect(audioContext.destination);
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

	function panner3DControl() {
		if (panner3D) {
			panner3D.positionX.value = pan3DX;
			panner3D.positionY.value = pan3DY;
			// console.log(`pan3D: (${panner3D.positionX.value}, ${panner3D.positionY.value})`);
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

			pan3DX = mouseX;
			pan3DY = mouseY;

			panner3DControl();

			return `Left: ${distanceLeft}, Right: ${distanceRight}`;
		}
	}

	function moveMouse(e: MouseEvent | TouchEvent) {
		if (e instanceof MouseEvent) {
			mouseX = e.pageX - window.innerWidth / 2 + 200;
			mouseY = e.pageY - 77;
		} else if (e.touches && e.touches.length > 0) {
			mouseX = e.touches[0].pageX - window.innerWidth / 2 + 200;
			mouseY = e.touches[0].pageY - 77;
		}

		moveCircle();
		getDistance();
	}

	function circleSurround() {
		let angle = 0;
		let radius = 100;
		let revolutions = 3;
		let duration = 3; // seconds per revolution
		let totalTime = revolutions * duration * 1000; // total time in milliseconds
		let interval = 10; // update interval in milliseconds

		if (!audioContext || !panner3D) return;

		let startTime = performance.now();

		function updatePosition() {
			let elapsedTime = performance.now() - startTime;

			if (elapsedTime > totalTime) {
				// Stop after completing the revolutions
				return;
			}

			// Calculate the angle based on elapsed time
			angle = (2 * Math.PI * (elapsedTime % (duration * 1000))) / (duration * 1000);

			// Update the position of the sound source
			panner3D.positionX.value = listener.positionX.value + radius * Math.cos(angle);
			panner3D.positionY.value = listener.positionY.value + radius * Math.sin(angle);

			// Update the draggable circle's position to match the sound source
			draggableCircle.setAttribute("cx", panner3D.positionX.value);
			draggableCircle.setAttribute("cy", panner3D.positionY.value);

			// Update the lines to connect the draggable circle and fixed circle
			lineLeft.setAttribute("x1", panner3D.positionX.value);
			lineLeft.setAttribute("y1", panner3D.positionY.value);
			lineLeft.setAttribute("x2", fixedCircleX - 20);
			lineLeft.setAttribute("y2", fixedCircleY);

			lineRight.setAttribute("x1", panner3D.positionX.value);
			lineRight.setAttribute("y1", panner3D.positionY.value);
			lineRight.setAttribute("x2", fixedCircleX + 20);
			lineRight.setAttribute("y2", fixedCircleY);

			// Schedule the next update
			setTimeout(updatePosition, interval);
		}

		updatePosition();
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
	<circle bind:this={fixedCircle} cx="200" cy="200" r="20" fill="blue" id="fixedCircle" />

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
		cy="100"
		r="20"
		fill="red"
		class="draggable"
		id="draggableCircle"
	/>

	<!-- Dotted lines -->
	<line bind:this={lineLeft} id="lineLeft" x1="200" y1="100" x2="180" y2="200" stroke="black" />
	<line bind:this={lineRight} id="lineRight" x1="200" y1="100" x2="220" y2="200" stroke="black" />
</svg>

<!-- <h1><b>Distance Left:</b> {distanceLeft}</h1> -->
<!-- <h1><b>Distance Right:</b> {distanceRight}</h1> -->

<div class="mt-5 flex">
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
<button
	class="ring-offset-background focus-visible:ring-ring inline-flex h-10 items-center justify-center rounded-md bg-slate-900 px-4 py-2 text-sm font-medium whitespace-nowrap text-slate-50 transition-colors focus-visible:ring-2 focus-visible:ring-offset-2 focus-visible:outline-none disabled:pointer-events-none disabled:opacity-50"
	on:click={circleSurround}>🔁</button
>
