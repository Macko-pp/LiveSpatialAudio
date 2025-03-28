<script lang="ts">
	import "../app.css";
	import { Button } from "$lib/components/ui/button";

	let { children } = $props();
</script>

<div class="w-100% mb-3 h-15 bg-slate-100 p-2 flex justify-center space-x-2">
	<Button class="bg-slate-300 font-bold" href="/">Dynamic 3D Audio</Button>
	<Button class="bg-slate-300 font-bold" href="/3dAudio">Homemade Stereo</Button>
</div>

<div class="flex justify-center">
	<div>{@render children()}</div>
</div>
