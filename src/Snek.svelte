<script>
	import { onMount } from 'svelte';
	import rough from 'roughjs';

	let el;

	onMount(() => {
		const rc = rough.canvas(el);
		rc.rectangle(10, 10, el.width - 20, el.height - 20, { fill: 'black', roughness: 4 });
	});
</script>

<style>
	.snek {
		position: absolute;
		width: 100%;
		height: 100%;
	}
</style>

<canvas class="snek" bind:this={el}></canvas>
