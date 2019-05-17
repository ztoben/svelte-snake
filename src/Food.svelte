<script>
	import { onMount } from 'svelte';
	import rough from 'roughjs';

	let el;

	onMount(() => {
		const rc = rough.canvas(el);
		rc.ellipse(el.width / 2, el.height / 2, el.width / 2, el.height / 2, { fill: 'red', roughness: 2 });
	});
</script>

<style>
	.food {
		position: absolute;
		width: 100%;
		height: 100%;
	}
</style>

<canvas class="food" bind:this={el}></canvas>
