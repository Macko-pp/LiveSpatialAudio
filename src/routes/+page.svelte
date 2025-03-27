<script lang="ts">
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
		fixedCircleX = parseFloat(fixedCircle.getAttribute('cx'));
		fixedCircleY = parseFloat(fixedCircle.getAttribute('cy'));

		if (dragging) {
			// Update the position of the draggable circle
			draggableCircle.setAttribute('cx', mouseX);
			draggableCircle.setAttribute('cy', mouseY);

			// Update the lines
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
			const distanceLeft = Math.sqrt(
				(mouseX - (fixedCircleX - 20)) ** 2 + (mouseY - fixedCircleY) ** 2
			);
			const distanceRight = Math.sqrt(
				(mouseX - (fixedCircleX + 20)) ** 2 + (mouseY - fixedCircleY) ** 2
			);

			return `Left: ${distanceLeft.toFixed(2)}, Right: ${distanceRight.toFixed(2)}`;
		}
	}

	function moveMouse(e: { pageX: number; pageY: number }) {
		mouseX = e.pageX;
		mouseY = e.pageY;

		if (dragging) {
			moveCircle();
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

<h1>Mouse: ({mouseX}, {mouseY})</h1>
<!-- <hr /> -->
<h1>Distance: {getDistance()}</h1>
