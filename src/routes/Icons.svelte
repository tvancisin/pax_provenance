<script>
	import { createEventDispatcher } from 'svelte';
	export let which_icon, d;

	const dispatch = createEventDispatcher();

	function handle_mouse_enter() {
		dispatch('hover', { node: d });
	}
</script>

<foreignObject
	x="-12"
	y="-12"
	width="24"
	height="24"
	role="button"
	tabindex="0"
	aria-label="Icon"
	on:mouseenter={() => {
		handle_mouse_enter(d);
	}}
	on:mouseleave={() => dispatch('hover', { node: null })}
	on:click={() => dispatch('click', { node: d })}
>
	<div class="icon" style={`background-image: url('${which_icon}');`}></div>
</foreignObject>

<style>
	.icon {
		width: 24px;
		height: 24px;
		background-size: cover;
		border-radius: 4px;
		transition: transform 0.2s ease;
	}

	foreignObject:hover .icon {
		transform: scale(2);
	}
	foreignObject {
		overflow: visible;
	}
</style>
