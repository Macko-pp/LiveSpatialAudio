<script lang="ts">
	import "../app.css";
	import { Button } from "$lib/components/ui/button";
	import { page } from '$app/state';

	const currentPath = $derived(page.url.pathname);
	
	let { children } = $props();
</script>

<div class="w-100% mb-3 h-15 bg-slate-100 p-2 flex justify-center space-x-2">

	{#if currentPath === "/"}
		<Button class="bg-slate-300 font-bold border-4 border-red-500" href="/">Dynamic 3D Audio</Button>
	{:else}
		<Button class="bg-slate-300 font-bold" href="/">Dynamic 3D Audio</Button>
	{/if}

	{#if currentPath === "/diy"}
		<Button class="bg-slate-300 font-bold border-4 border-red-500" href="/diy">Homemade Stereo</Button>
	{:else}
		<Button class="bg-slate-300 font-bold" href="/diy">Homemade Stereo</Button>
	{/if}
</div>

<div class="flex justify-center">
	<div>{@render children()}</div>
</div>
