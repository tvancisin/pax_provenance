<script>
	import { createEventDispatcher } from 'svelte';
	export let which_icon, d;

	const dispatch = createEventDispatcher();

	function handle_mouse_enter() {
		dispatch('hover', { node: d });
	}
</script>

<foreignObject
	x="-8"
	y="-8"
	width="18"
	height="18"
    role="button"
    tabindex="0"
    aria-label="Icon"
	on:mouseenter={() => {
		handle_mouse_enter(d);
	}}
	on:mouseleave={() => dispatch('hover', { node: null })}
>
	<div
		style={`width: 18px;
				height: 18px;
				background-image: url('${which_icon}');
				background-size: cover;
				border-radius: 4px;`}
	></div>
</foreignObject>
