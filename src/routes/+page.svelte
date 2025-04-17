<script>
	import * as d3 from 'd3';
	import Icons from './Icons.svelte';
	import { original_data } from '$lib/utils';

	let data = JSON.parse(JSON.stringify(original_data)); // deep clone

	let width,
		height,
		details_width = 1,
		details_data = [],
		clicked = null;

	// adding id's to each node
	let idCounter = 0;
	function assignIds(node) {
		node.id = `node-${idCounter++}`;
		if (node.children) {
			node.children.forEach(assignIds);
		}
	}

	assignIds(data);
	data.downward.forEach(assignIds);

	// dimensions
	let margin = 60;
	$: innerWidth = width * 0.9;
	$: innerHeight = height - 2 * margin;
	$: yCenter = margin + innerHeight * 0.8;
	$: upHeight = yCenter - margin;
	$: downHeight = innerHeight - upHeight;
	$: xCenter = innerWidth / 2;
	$: time_line = d3
		.scaleLinear()
		.domain([0, 100])
		.range([20, innerWidth / 4]);

	let nodesUp = [],
		linksUp = [];
	let nodesDown = [],
		linksDown = [];

	$: upwardCluster = d3.cluster().size([innerWidth - margin, upHeight]);
	$: downwardCluster = d3.cluster().size([innerWidth - margin, downHeight]);

	$: rootUp = d3.hierarchy(data, (d) => d.children);
	$: rootDown = d3.hierarchy({ ...data, children: data.downward }, (d) => d.children);

	function centerTree(root, centerX) {
		const dx = centerX - root.x;
		root.each((d) => (d.x += dx));
	}

	$: {
		upwardCluster(rootUp);
		downwardCluster(rootDown);
		centerTree(rootUp, xCenter);
		centerTree(rootDown, xCenter);

		rootUp.each((d) => {
			if (d.data.name === 'Translate') {
				d.y -= 25; // Bring transcribe node closer to collect
			}
			if (d.data.name === 'd3' || d.data.name === 'PBi') {
				d.y -= 10; // Bring transcribe node closer to collect
			}
			if (d.data.name === 'Transcribe') {
				d.y -= 50; // Bring transcribe node closer to collect
			}
			if (d.data.name === 'Code') {
				d.y -= 75; // Bring transcribe node closer to collect
			}
			if (d.data.name === 'PA-X') {
				d.y -= 100; // Bring pax node closer to transcribe
			}
			if (
				d.data.name === 'PAA-X' ||
				d.data.name === 'PA-X Gender' ||
				d.data.name === 'PA-X Local' ||
				d.data.name === 'Children & Youth'
			) {
				d.y -= 50; // Bring paax node closer to transcribe
			}
		});

		rootDown.each((d) => {
			if (d.data.name === 'conflict') {
				d.y -= 30; // You can adjust this value to control vertical gap
			}
		});

		nodesUp = rootUp.descendants();
		linksUp = nodesUp.slice(1);
		nodesDown = rootDown.descendants().slice(1); // remove duplicate root
		linksDown = nodesDown.filter((d) => d.parent); // just in case
	}

	function removeNodesAndPrune(node, targetNames) {
		if (!node.children) return { ...node };

		const newChildren = [];

		for (const child of node.children) {
			// If this child should be removed, skip it
			if (targetNames.has(child.name)) continue;

			// Recursively process the child
			const updatedChild = removeNodesAndPrune(child, targetNames);

			// Keep it if not pruned
			if (updatedChild) newChildren.push(updatedChild);
		}

		// If all children were removed and at least one was a target, prune this node
		const hadTargetChild = node.children.some((c) => targetNames.has(c.name));
		if (newChildren.length === 0 && hadTargetChild) {
			return null;
		}

		return {
			...node,
			children: newChildren.length > 0 ? newChildren : undefined
		};
	}

	function reset() {
		// data = original_data
		innerWidth = width * 0.9;
		details_width = 1;
		fullChain = [];
		clicked = null;
		highlightedLinks = new Set();
	}

	function filter() {
		const targets = new Set(['Scrollytelling', 'Tracker', 'Network']);
		data = removeNodesAndPrune(data, targets);
	}

	// mouseover
	let highlightedLinks = new Set();
	function handleHoverEvent(e) {
		// console.log(e.detail.node);
		if (!clicked) {
			let node = e.detail.node;
			if (!node) {
				highlightedLinks = new Set();
				return;
			}

			const links = new Set();

			// 1. Highlight upward ancestry
			while (node.parent) {
				const key = `${node.parent.data.id}→${node.data.id}`;
				links.add(key);
				node = node.parent;
			}

			// 2. Always add all downward links
			linksDown.forEach((d) => {
				const key = `${d.parent.data.id}→${d.data.id}`;
				links.add(key);
			});

			highlightedLinks = links;
		}
	}

	// click
	let fullChain = [];
	function handleClickEvents(e) {
		if (clicked == null) {
			innerWidth = innerWidth / 2;
		}
		clicked = true;
		fullChain = [];
		details_width = innerWidth;
		details_data = e.detail.node;
		let current = details_data;
		let downCurrent = nodesDown[16];
		let downCurrentChain = []; // Temporary array to store downCurrent hierarchy

		// Walk up to collect all parents from details_data
		while (current) {
			fullChain.push(current); // push keeps order as leaf → root
			current = current.parent;
		}

		// Walk up to collect all parents from downCurrent, except the last one
		while (downCurrent && downCurrent.parent !== null) {
			downCurrentChain.push(downCurrent); // Collect downCurrent nodes in order
			downCurrent = downCurrent.parent;
		}

		// Push the downCurrent chain in reverse order
		fullChain = fullChain.concat(downCurrentChain.reverse());
	}

	$: paaxNode = nodesUp.find((d) => d.data.name === 'PAA-X');
	$: transNode = nodesUp.find((d) => d.data.choose === 'last');
	$: trackerNode = nodesUp.find((d) => d.data.name === 'PA-X Tracker');

	$: curvePath = (node_one, node_two) => {
		const x1 = node_one.x;
		const y1 = node_one.y;

		const x2 = node_two.x;
		const y2 = innerHeight - margin - node_two.y;

		const dx = x2 - x1;
		const dy = y2 - y1;

		// You can tweak these multipliers for different shapes
		const cx1 = x1 - dx * 0; // bend left if x2 < x1
		const cy1 = y1 + dy * 0.5; // drop down

		const cx2 = x2 + dx * 0;
		const cy2 = y2 - dy * 0.5;

		return `M${x1},${y1} C${cx1},${cy1} ${cx2},${cy2} ${x2},${y2}`;
	};

	let segment_height = 10;
	$: if (fullChain.length > 1) {
		segment_height = height / fullChain.length;
	}
</script>

<div id="wrapper" bind:clientWidth={width} bind:clientHeight={height}>
	<button id="reset" on:click={reset}>reset</button>
	<button id="filter" on:click={filter}>filter</button>
	{#if width !== undefined || height !== undefined}
		<svg {width} {height}>
			<g transform={`translate(${0}, ${margin})`}>
				<!-- ACLED and UCDP -->
				<path
					d={curvePath(transNode, trackerNode)}
					fill="none"
					stroke="gray"
					stroke-opacity="0.4"
					stroke-width="2"
				/>
				<g transform={`translate(${transNode.x}, ${transNode.y})`}>
					<text x="15" y="-10" font-size="12" fill="gray">
						{"Other data"}
					</text>
					<Icons which_icon={'img/other_data.png'} />
				</g>
				<!-- Upward Links -->
				{#each linksUp as d}
					<!-- <path
						d={`M${d.x},${yCenter - d.y}
					L${d.x},${yCenter - d.parent.y}
					L${d.parent.x},${yCenter - d.parent.y}`}
						fill="none"
						stroke={d.children &&
						['PAA-X', 'PA-X Gender', 'PA-X Local', 'Children & Youth'].includes(d.data.name)
							? 'white'
							: d.children
								? 'gray'
								: 'orange'}
						stroke-width={d.data.name === 'PA-X' ||
						d.data.name === 'Transcribe' ||
						d.data.name === 'Code' ||
						d.data.name === 'Translate'
							? 10
							: d.data.name === 'PAA-X'
								? 3
								: d.data.name === 'PA-X Gender'
									? 3
									: 3}
					/> -->
					<path
						d={`M${d.x},${yCenter - d.y}
						C${d.x},${yCenter - d.parent.y - 30}
						${d.parent.x},${yCenter - d.parent.y - 70}
						${d.parent.x},${yCenter - d.parent.y}`}
						fill="none"
						stroke={highlightedLinks.has(`${d.parent.data.id}→${d.data.id}`)
							? 'white'
							: d.data.type === 'prog'
								? '#4360AC'
								: d.data.type === 'db'
									? 'white'
									: !d.children
										? 'orange'
										: 'gray'}
						stroke-width={highlightedLinks.has(`${d.parent.data.id}→${d.data.id}`) ? '4' : '3'}
					/>
				{/each}

				<!-- Downward Links -->
				{#each linksDown as d}
					<!-- <path
						d={`M${d.x},${yCenter + d.y}
					L${d.x},${yCenter + d.parent.y}
					L${d.parent.x},${yCenter + d.parent.y}`}
						fill="none"
						stroke="steelblue"
					/> -->

					<path
						d={`M${d.x},${yCenter + d.y}
						C${d.x},${yCenter + d.parent.y}
						${d.parent.x},${yCenter + d.parent.y + 60}
						${d.parent.x},${yCenter + d.parent.y}`}
						fill="none"
						stroke={highlightedLinks.has(`${d.parent.data.id}→${d.data.id}`) ? 'white' : 'gray'}
						stroke-width="1"
					/>
				{/each}

				<!-- Upward Nodes -->
				{#each nodesUp as d}
					<g transform={`translate(${d.x}, ${yCenter - d.y})`}>
						<text
							x={d.children ? 15 : 5}
							y={d.children ? 5 : -10}
							font-size="12"
							fill={d.children && d.data.type == 'db' ? 'gray' : d.children ? 'gray' : 'gray'}
							transform={d.children ? 'rotate(0)' : 'rotate(-35)'}
						>
							{d.data.name}
						</text>
						{#if d.data.type === 'vis'}
							<Icons
								{d}
								which_icon="img/vis.png"
								on:hover={handleHoverEvent}
								on:click={handleClickEvents}
							/>
						{:else if d.data.type == 'db'}
							<Icons
								{d}
								which_icon="img/data.png"
								on:hover={handleHoverEvent}
								on:click={handleClickEvents}
							/>
						{:else if d.data.type == 'prog'}
							<Icons
								{d}
								which_icon="img/prog.png"
								on:hover={handleHoverEvent}
								on:click={handleClickEvents}
							/>
						{:else if d.data.name == 'Collect'}
							<Icons
								{d}
								which_icon="img/collect.png"
								on:hover={handleHoverEvent}
								on:click={handleClickEvents}
							/>
						{:else if d.data.name == 'Translate'}
							<Icons
								{d}
								which_icon="img/translate.png"
								on:hover={handleHoverEvent}
								on:click={handleClickEvents}
							/>
						{:else if d.data.name == 'Transcribe'}
							<Icons
								{d}
								which_icon="img/transcribe.png"
								on:hover={handleHoverEvent}
								on:click={handleClickEvents}
							/>
						{:else if d.data.name == 'Code'}
							<Icons
								{d}
								which_icon="img/annotation.png"
								on:hover={handleHoverEvent}
								on:click={handleClickEvents}
							/>
						{:else}
							<circle r="5" fill="gray" />
						{/if}
					</g>
				{/each}

				<!-- Downward Nodes -->
				{#each nodesDown as d}
					<g transform={`translate(${d.x}, ${yCenter + d.y})`}>
						{#if d.data.name == 'conflict'}
							<Icons {d} which_icon={'img/war.png'} />
							<text x="13" y="5" font-size="12" fill="gray"> {'war'}</text>
						{:else}
							<Icons {d} which_icon={'img/agt.png'} />
							<text x="13" y="5" font-size="12" fill="gray"> {'agt'}</text>
						{/if}
					</g>
				{/each}

				<!-- <path
					id="example"
					d={`M ${xScale(agt_path_year)},${innerHeight - 10} 
				C ${xScale(agt_path_year)},${innerHeight - 100} 
				  ${imageX - margin.left},${lineEnd + 100} 
				  ${imageX - margin.left},${lineEnd}`}
					fill="none"
					stroke="white"
					stroke-width="1"
					opacity="0"
				/> -->
			</g>
		</svg>
	{/if}
	<div id="details" style="margin-top: {0 + 'px'};">
		{#each fullChain as d}
			<a href={d.data.link} target="_blank">
				<div
					class="detail-segment"
					style="
			height: {segment_height}px;
			width: {details_width}px;
			display: flex;
		  "
				>
					<div
						class="segment-left"
						style="flex: 1; border-right: 1px solid rgba(128, 128, 128, 0.333);"
					>
						<!-- <p>{d.data.name}</p> -->
						<svg width="100%" height="100%" preserveAspectRatio="none">
							<g transform={`translate(16, ${segment_height / 2})`}>
								<text x="15" y="-15" font-size="12" fill="white">{d.data.name}</text>
								<line
									x1="1"
									y1="-32"
									x2="1"
									y2={segment_height - 24}
									stroke="white"
									stroke-width="2"
								/>
								{#if d.data.type === 'vis'}
									<Icons {d} which_icon="img/vis.png" />
								{:else if d.data.type == 'db'}
									<Icons {d} which_icon="img/data.png" />
								{:else if d.data.type == 'prog'}
									<Icons {d} which_icon="img/prog.png" />
								{:else if d.data.name == 'Collect'}
									<Icons {d} which_icon="img/collect.png" />
								{:else if d.data.name == 'Translate'}
									<Icons {d} which_icon="img/translate.png" />
								{:else if d.data.name == 'Transcribe'}
									<Icons {d} which_icon="img/transcribe.png" />
								{:else if d.data.name == 'Code'}
									<Icons {d} which_icon="img/annotation.png" />
								{:else if d.data.name == 'agt'}
									<Icons {d} which_icon="img/agt.png" />
								{:else if d.data.name == 'conflict'}
									<Icons {d} which_icon="img/war.png" />
								{:else}
									<circle r="5" fill="gray" />
								{/if}
								{#each Array(d.data.ppl) as _, j}
									<foreignObject
										x={20 + j * 10}
										y={-8 + Math.random()}
										width="10"
										height="10"
										role="button"
										tabindex="0"
										aria-label="Icon"
									>
										<div class="icon" style={`background-image: url('img/person.png');`}></div>
									</foreignObject>
								{/each}
								<line
									x1="20"
									y1="10"
									x2={20 + time_line(d.data.time)}
									y2="10"
									stroke="gray"
									stroke-width="1"
								/>
							</g>
						</svg>
					</div>
					<div
						class="segment-right"
						style="
				  flex: 1;
				  background-image: url('{d.data.url}');
				  background-size: contain;
				  <!-- background-repeat: no-repeat; -->
				  background-position: center;
				"
					></div>
				</div>
			</a>
		{/each}
	</div>
	<!-- <div style="text-align: center;">
		<p>TODO</p>
		<p>alternate white and gray for branches</p>
		<p>Hover over a node, highlight all its parents.</p>
		<p>How to interact with links and nodes?</p>
		<p>What filtering to apply?</p>
		<p>How to restructure the tree for different queries?</p>
		<p>Which part to quantify and which to qualify?</p>
		<p>How do the same icons differ and how to show?</p>
		<p>What to highlight before user goes to individual vis?</p>
		<iframe
			title="Miro Board"
			src="https://miro.com/app/live-embed/uXjVIYrT328=/?moveToViewport=-208202,-17495,146090,66767&embedId=356675018147"
			frameborder="0"
			scrolling="no"
			allow="fullscreen; clipboard-read; clipboard-write"
			allowfullscreen
		></iframe>
	</div> -->
</div>

<style>
	#wrapper {
		width: 100%;
		height: 100vh;
	}
	#details {
		position: absolute;
		top: 0;
		right: 0;
		margin: 10px;
	}
	.detail-segment {
		color: white;
		display: flex;
		background-color: #001c23;
		border-radius: 10px;
		border: solid 2px rgba(128, 128, 128, 0.1);
		transition: border-color 0.2s ease;
	}

	.detail-segment:hover {
		border-color: rgba(255, 255, 255, 0.8); /* brighter border */
	}

	.segment-left,
	.segment-right {
		flex: 1;
		padding: 0;
		box-sizing: border-box;
	}

	.segment-right {
		border-radius: 10px;
	}

	button {
		width: 50px;
	}

	#reset {
		position: absolute;
		top: 0px;
		left: 0px;
	}

	#filter {
		position: absolute;
		top: 25px;
		left: 0px;
	}
	iframe {
		width: 90%;
		height: 100vh;
		border: none;
	}
	.icon {
		width: 10px;
		height: 10px;
		background-size: cover;
		border-radius: 4px;
		transition: transform 0.2s ease;
	}
</style>
