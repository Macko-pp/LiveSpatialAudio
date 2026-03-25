<script lang="ts">
	import { Input } from "$lib/components/ui/input";
	import { Play, Pause, RefreshCw, Music2 } from "lucide-svelte";
	import BezierEasing from "bezier-easing"

	let audioContext: AudioContext | null = null;
	let audioElement: HTMLAudioElement | null = null;

	let track: MediaElementAudioSourceNode;

	let listener: AudioListener;
	let panner3D: PannerNode;

	let pan3DX: number = 200;
	let pan3DY: number = 100;

	let positionX: number;
	let positionY: number;

	let forwardX: number;
	let forwardY: number;
	let forwardZ: number;
	let upX: number;
	let upY: number;
	let upZ: number;

	function initializeAudio() {
		if (typeof window !== "undefined") {
			audioContext = new AudioContext();
		}

		if (audioContext && audioElement) {
			listener = audioContext.listener;
			positionX = 200;
			positionY = 200;

			forwardX = 0;
			forwardY = 20;
			forwardZ = 0;
			upX = 0;
			upY = 0;
			upZ = 20;

			if (listener.positionX) {
				listener.positionX.value = positionX;
				listener.positionY.value = positionY;
				listener.forwardX.value = forwardX;
				listener.forwardY.value = forwardY;
				listener.forwardZ.value = forwardZ;
				listener.upX.value = upX;
				listener.upY.value = upY;
				listener.upZ.value = upZ;
			} else {
				listener.setPosition(positionX, positionY, 0);
				listener.setOrientation(forwardX, forwardY, forwardZ, upX, upY, upZ);
			}

			panner3D = new PannerNode(audioContext, {
				panningModel: "HRTF",
				distanceModel: "exponential",
				positionX: pan3DX,
				positionY: pan3DY,
				positionZ: 0,
				orientationX: 0,
				orientationY: 0,
				orientationZ: 0,
				refDistance: 50,
				maxDistance: 10000,
				rolloffFactor: 1,
				coneInnerAngle: 360,
				coneOuterAngle: 360,
				coneOuterGain: 1
			});

			if (!track) track = audioContext.createMediaElementSource(audioElement);
			track.connect(panner3D).connect(audioContext.destination);
		}
	}

	let playButton: HTMLElement;
	let buttonIcon: string = "▶";

	function playAudio() {
		if (!audioContext || !track || !audioElement) return;
		if (audioContext.state === "suspended") audioContext.resume();

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
		}
	}

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

	function updatePosition() {
		if (dragging) {
			revolveRadius = parseFloat(
				Math.sqrt((mouseX - fixedCircleX) ** 2 + (mouseY - fixedCircleY) ** 2).toFixed(2)
			);

			pan3DX = mouseX;
			pan3DY = mouseY;

			panner3DControl();
		}
	}

	function moveMouse(e: MouseEvent | TouchEvent) {
		const rect = svg.getBoundingClientRect();
		// Scale factor to handle potential CSS scaling of the SVG
		const scaleX = 400 / rect.width;
		const scaleY = 400 / rect.height;

		if (e instanceof MouseEvent) {
			mouseX = (e.clientX - rect.left) * scaleX;
			mouseY = (e.clientY - rect.top) * scaleY;
		} else if (e.touches && e.touches.length > 0) {
			mouseX = (e.touches[0].clientX - rect.left) * scaleX;
			mouseY = (e.touches[0].clientY - rect.top) * scaleY;
		}

		moveCircle();
		updatePosition();
	}

	let revolving = false;
	let revolveRadius: number = 100;
	let angle = 0;
	let animationFrameId: number;
	let duration = 0.5; // Period

	const durationEasing = BezierEasing(0.00, 0.00, 0.27, 1.00)

	function circleSurround() {
		if (revolving) {
			revolving = false;
			cancelAnimationFrame(animationFrameId);
			return;
		}
		if (!audioContext || !panner3D) return;

		revolving = true;

		// 1. Calculate the starting angle based on current pan positions relative to center (200, 200)
		angle = Math.atan2(pan3DY - 200, pan3DX - 200);

		let lastTime = performance.now();

		function animate(currentTime: number) {
			if (!revolving) return;

			const deltaTime = currentTime - lastTime;
			lastTime = currentTime;

			// 2. Infinite rotation logic: Increment angle based on duration (speed)
			// (2 * Math.PI) / (duration * 1000) = radians per millisecond
			const angleStep = (2 * Math.PI * deltaTime) / ((5 - durationEasing(duration) * 4.9) * 1000);
			angle += angleStep;

			// 3. Update PannerNode
			if (listener.positionX) {
				panner3D.positionX.value = 200 + revolveRadius * Math.cos(angle);
				panner3D.positionY.value = 200 + revolveRadius * Math.sin(angle);
			} else {
				panner3D.positionX.value = 200 + revolveRadius * Math.cos(angle);
				panner3D.positionY.value = 200 + revolveRadius * Math.sin(angle);
			}

			// 4. Synchronize UI
			pan3DX = panner3D.positionX.value;
			pan3DY = panner3D.positionY.value;

			draggableCircle.setAttribute("cx", pan3DX);
			draggableCircle.setAttribute("cy", pan3DY);

			lineLeft.setAttribute("x1", pan3DX);
			lineLeft.setAttribute("y1", pan3DY);
			lineRight.setAttribute("x1", pan3DX);
			lineRight.setAttribute("y1", pan3DY);

			animationFrameId = requestAnimationFrame(animate);
		}

		animationFrameId = requestAnimationFrame(animate);
	}
</script>

<div
	class="flex min-h-screen flex-col items-center justify-center p-4 font-sans sm:p-8 w-100%"
>
	<div
		class="grid w-full max-w-5xl grid-cols-1 gap-6 rounded-2xl border border-slate-200 bg-white p-6 shadow-xl lg:grid-cols-3"
	>
		<!-- Controls Panel -->
		<div class="flex flex-col justify-between space-y-6">
			<div class="space-y-6">
				<div>
					<h2 class="text-2xl font-bold tracking-tight text-slate-900">Audio Studio</h2>
					<p class="text-sm text-slate-500">Spatial 3D Audio Panner</p>
				</div>

				<div class="space-y-4">
					<div class="rounded-xl border border-slate-100 bg-slate-50 p-4">
						<label
							class="mb-2 block text-xs font-semibold tracking-wider text-slate-500 uppercase"
							for="audio-upload">Source Track</label
						>
						<Input
							id="audio-upload"
							type="file"
							accept="audio/*"
							class="bg-white"
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

					<div class="flex gap-2">
						<button
							bind:this={playButton}
							data-playing="false"
							class="flex h-12 flex-1 items-center justify-center gap-2 rounded-lg font-medium transition-all {buttonIcon ===
							'❚❚'
								? 'border border-red-200 bg-red-50 text-red-600'
								: 'bg-slate-900 text-white hover:bg-slate-800'}"
							onclick={playAudio}
						>
							{#if buttonIcon === "▶"}
								<Play size={20} fill="currentColor" /> Play
							{:else}
								<Pause size={20} fill="currentColor" /> Pause
							{/if}
						</button>

						<button
							class="flex h-12 w-12 items-center justify-center rounded-lg border transition-all {revolving
								? 'border-blue-600 bg-blue-600 text-white'
								: 'border-slate-200 bg-white text-slate-600 hover:bg-slate-50'}"
							onclick={circleSurround}
						>
							<RefreshCw size={20} class={revolving ? "animate-spin" : ""} />
						</button>
					</div>
				</div>

				<div class="space-y-6 border-t border-slate-100 pt-2">
					<div class="space-y-3">
						<div class="flex justify-between">
							<label class="text-sm font-medium text-slate-700" for="radius"
								>Distance</label
							>
							<span class="font-mono text-xs text-slate-400">{revolveRadius}px</span>
						</div>
						<input
							id="radius"
							type="range"
							min="20"
							max="180"
							bind:value={revolveRadius}
							autocomplete="off"
							class="h-2 w-full cursor-pointer appearance-none rounded-lg bg-slate-200 accent-blue-600"
						/>
					</div>

					<div class="space-y-3">
						<div class="flex justify-between">
							<label class="text-sm font-medium text-slate-700" for="speed"
								>Orbit Speed (Period)</label
							>
							<span class="font-mono text-xs text-slate-400">{(5 - durationEasing(duration) * 4.9).toFixed(2)}s</span>
						</div>
						<input
							id="speed"
							type="range"
							min="0"
							max="1"
							step="0.01"
							bind:value={duration}
							autocomplete="off"
							class="h-2 w-full cursor-pointer appearance-none rounded-lg bg-slate-200 accent-blue-600"
						/>
					</div>
				</div>
			</div>

			<div class="font-mono text-[10px] tracking-widest text-slate-400 uppercase">
				<div class="rounded bg-slate-50 p-2">Distance: {revolveRadius || "0"}</div>
			</div>
		</div>

		<!-- Visualizer Canvas -->
		<div
			class="relative aspect-square overflow-hidden rounded-2xl bg-slate-900 shadow-inner lg:col-span-2"
		>
			<div
				class="absolute inset-0 opacity-15"
				style="background-image: radial-gradient(#444 2px, transparent 2px); background-size: 20px 20px;"
			></div>

			<!-- svelte-ignore a11y_no_static_element_interactions -->
			<svg
				bind:this={svg}
				viewBox="0 0 400 400"
				class="relative z-10 h-full w-full touch-none"
				onmousemove={moveMouse}
				ontouchmove={moveMouse}
				onmouseup={() => (dragging = false)}
				ontouchend={() => (dragging = false)}
			>
				<!-- Connection Lines -->
				<line
					bind:this={lineLeft}
					x1="200"
					y1="100"
					x2="180"
					y2="200"
					stroke="white"
					stroke-width="1"
					stroke-dasharray="4 4"
					opacity="0.15"
				/>
				<line
					bind:this={lineRight}
					x1="200"
					y1="100"
					x2="220"
					y2="200"
					stroke="white"
					stroke-width="1"
					stroke-dasharray="4 4"
					opacity="0.15"
				/>

				<!-- Listener -->
				<g>
					<circle
						cx="200"
						cy="200"
						r={revolveRadius}
						fill="none"
						stroke="white"
						stroke-width="1"
						stroke-dasharray="8 8"
						opacity="0.1"
					/>
					<circle
						bind:this={fixedCircle}
						cx="200"
						cy="200"
						r="20"
						fill="#1e293b"
						stroke="#3b82f6"
						stroke-width="2"
					/>
					<rect x="174" y={200 - 4} width="50" height="8" rx="2" fill="#323BFF" />
					<rect x="174" y="192" width="6" height="16" rx="2" fill="#323BFF" />
					<rect x="220" y="192" width="6" height="16" rx="2" fill="#323BFF" />
					<rect x={200 - 8} y={200 - 15} width="3" height="8" rx="2" fill="#ffffff" />
					<rect x={200 + 5} y={200 - 15} width="3" height="8" rx="2" fill="#ffffff" />
					<Music2
						x={200 + 25}
						y={200 - 30}
						color="#ffffff"
						size="15"
						transform="rotate(20, {200 + 25}, {200 - 30})"
					/>
					<Music2
						x={200 + 35}
						y={200 - 20}
						color="#ffffff"
						size="10"
						transform="rotate(30, {200 + 35}, {200 - 20})"
					/>
					<path
						d="M {200 - 5} {200 - 10} L {200} {200 - 15} L {200 + 5} {200 - 10}"
						fill="none"
						stroke="white"
						stroke-width="2"
						stroke-linecap="round"
						transform="translate(0, -20)"
					/>
				</g>

				<!-- Audio Source -->
				<!-- svelte-ignore a11y_no_static_element_interactions -->
				<g
					class="cursor-grab active:cursor-grabbing"
					onmousedown={() => (dragging = true)}
					ontouchstart={() => (dragging = true)}
				>
					<!-- The Glow - Now bound to pan3DX/Y -->
					<circle
						cx={pan3DX || 200}
						cy={pan3DY || 100}
						r="35"
						fill="#ef4444"
						fill-opacity="0.1"
					>
						{#if buttonIcon === "❚❚"}
							<animate
								attributeName="r"
								values="25;40;25"
								dur="2s"
								repeatCount="indefinite"
							/>
							<animate
								attributeName="fill-opacity"
								values="0.05;0.2;0.05"
								dur="2s"
								repeatCount="indefinite"
							/>
						{/if}
					</circle>
					<circle
						bind:this={draggableCircle}
						cx="200"
						cy="100"
						r="10"
						fill="#ef4444"
						stroke="white"
						stroke-width="2"
					/>
					<circle
						cx={pan3DX || 200}
						cy={pan3DY || 100}
						r="3"
						fill="white"
						opacity="0.5"
					/>
				</g>
			</svg>

			<!-- <div class="absolute top-4 left-4 flex gap-2">
				<span class="px-2 py-1 bg-black/40 backdrop-blur-md text-[10px] text-white rounded border border-white/10 uppercase font-bold tracking-tighter flex items-center gap-1">
					<Info size={10} /> HRTF Spatial Engine
				</span>
			</div> -->
		</div>
	</div>
</div>

<audio bind:this={audioElement} hidden></audio>

<style>
	/* .slider-input {
		@apply w-full h-2 bg-slate-200 rounded-lg appearance-none cursor-pointer accent-blue-600;
	} */
	input[type="range"]::-webkit-slider-thumb {
		-webkit-appearance: none;
		appearance: none;
		width: 18px;
		height: 18px;
		background: #3b82f6;
		border-radius: 50%;
		cursor: pointer;
		border: 2px solid white;
	}
</style>
