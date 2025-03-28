<script lang="ts">
    import { Input } from "$lib/components/ui/input";
	import { Button } from "$lib/components/ui/button/index.js";

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
        fixedCircleX = parseFloat(fixedCircle.getAttribute('cx'));
        fixedCircleY = parseFloat(fixedCircle.getAttribute('cy'));

        if (dragging) {
            draggableCircle.setAttribute('cx', mouseX);
            draggableCircle.setAttribute('cy', mouseY);

            lineLeft.setAttribute('x1', mouseX);
            lineLeft.setAttribute('y1', mouseY);
            lineLeft.setAttribute('x2', fixedCircleX - 20);
            lineLeft.setAttribute('y2', fixedCircleY);

            lineRight.setAttribute('x1', mouseX);
            lineRight.setAttribute('y1', mouseY);
            lineRight.setAttribute('x2', fixedCircleX + 20);
            lineRight.setAttribute('y2', fixedCircleY);
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

            return `Left: ${distanceLeft}, Right: ${distanceRight}`;
        }
    }

    function moveMouse(e: { pageX: number; pageY: number }) {
        mouseX = e.pageX;
        mouseY = e.pageY;

        moveCircle();
        getDistance();
    }

    // Create and Manage Audio -----------------------------------------------------------------------------------------------
    let audioContext: AudioContext | null = null;
    let audioElement: HTMLAudioElement | null = null;

    if (typeof window !== "undefined") {
        audioContext = new AudioContext();
    }

    let track: MediaElementAudioSourceNode | null = null;

    function initializeAudio() {
        if (audioContext && audioElement) {
            track = audioContext.createMediaElementSource(audioElement);
            track.connect(audioContext.destination);
        }
    }

    let playButton: any;

	// let buttonIcon: string;

    function playAudio() {
        if (!audioContext || !track || !audioElement) return;

        if (audioContext.state === "suspended") {
            audioContext.resume();
        }

        if (playButton.dataset.playing === "false") {
            audioElement.play();
            playButton.dataset.playing = "true";
			// buttonIcon = "❚❚";
        } else if (playButton.dataset.playing === "true") {
            audioElement.pause();
            playButton.dataset.playing = "false";
			// buttonIcon = "▶";
        }
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

<Input
    class="mt-5 w-96"
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

<audio bind:this={audioElement} hidden></audio>

<button class="bg-slate-900 text-slate-50 ring-offset-background focus-visible:ring-ring inline-flex items-center justify-center whitespace-nowrap rounded-md text-sm font-medium transition-colors focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-offset-2 disabled:pointer-events-none disabled:opacity-50 h-10 px-4 py-2" data-playing="false" role="switch" aria-checked="false" bind:this={playButton} on:click={playAudio}>
  	<!-- {buttonIcon} -->
	Play/Pause
</button>