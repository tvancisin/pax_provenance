<script>
	import * as d3 from 'd3';
	import Icons from './Icons.svelte';
	import { original_data } from '$lib/utils';

	let data = JSON.parse(JSON.stringify(original_data));
	let margin = 20;
	let width,
		height,
		details_width = 1,
		details_data = [],
		clicked = null;

	$: innerWidth = width * 0.7;
	$: innerWidthPpl = width * 0.3;
	$: innerHeight = height;
	$: yCenter = innerHeight * 0.8;
	$: upHeight = yCenter;
	$: downHeight = innerHeight - upHeight;
	$: xCenter = innerWidth / 2 - margin * 2;

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

	$: time_line = d3
		.scaleLinear()
		.domain([0, 100])
		.range([20, innerWidth / 4]);

	let nodesUp = [],
		linksUp = [];
	let nodesDown = [],
		linksDown = [];

	$: upwardCluster = d3.cluster().size([innerWidth - 70, upHeight - 80]);
	$: downwardCluster = d3.cluster().size([innerWidth - 70, downHeight]);

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
				d.y -= 40; // You can adjust this value to control vertical gap
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
		innerWidth = width * 0.7;
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
			innerWidth = width / 2;
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

	let segment_height;
	$: if (fullChain.length > 1) {
		const totalBorder = fullChain.length * 4;
		segment_height = (height - totalBorder) / fullChain.length;
	}

	let numCircles = 800;
	let circles = [];

	$: console.log(innerWidthPpl);

	$: if (innerWidthPpl) {
		circles = [];

		// Grid settings
		let cols = Math.ceil(Math.sqrt(numCircles * (innerWidthPpl / innerHeight)));
		let rows = Math.ceil(numCircles / cols);
		let padding = 1;

		// Circle radius based on available space
		let cellWidth = (innerWidthPpl - padding * 2) / cols;
		let cellHeight = (innerHeight - padding * 2) / rows;
		let radius = Math.min(cellWidth, cellHeight) / 2 - 2;

		// Generate circles in grid
		for (let i = 0; i < numCircles; i++) {
			let col = i % cols;
			let row = Math.floor(i / cols);
			let x = padding + col * cellWidth + cellWidth / 2 + Math.random() * 3;
			let y = padding + row * cellHeight + cellHeight / 2 - Math.random() * 2;
			circles.push({ x, y, r: radius });
		}
	}
</script>

<div id="wrapper" bind:clientWidth={width} bind:clientHeight={height}>
	<div class="tree">
		<button id="reset" on:click={reset}>reset</button>
		<!-- <button id="filter" on:click={filter}>filter</button> -->
		{#if width !== undefined || height !== undefined}
			<svg width={innerWidth} {height}>
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
						<!-- <text x="15" y="-10" font-size="12" fill="gray">
							{'Other data'}
						</text> -->
						<Icons which_icon={'img/other_data.png'} />
					</g>
					<!-- Upward Links -->
					{#each linksUp as d}
						<path
							d={`M${d.x},${yCenter - d.y}
						C${d.x},${yCenter - d.parent.y - 50}
						${d.parent.x},${yCenter - d.parent.y - 80}
						${d.parent.x},${yCenter - d.parent.y}`}
							fill="none"
							stroke={highlightedLinks.has(`${d.parent.data.id}→${d.data.id}`)
								? 'orange'
								: d.data.type === 'prog'
									? 'steelblue'
									: d.data.type === 'db'
										? 'steelblue'
										: !d.children
											? 'steelblue'
											: 'steelblue'}
							stroke-width={highlightedLinks.has(`${d.parent.data.id}→${d.data.id}`)
								? 5
								: d.data.name === 'PA-X' ||
									  d.data.name === 'Transcribe' ||
									  d.data.type === 'first_code' ||
									  d.data.name === 'Translate'
									? 10
									: d.data.name === 'PAA-X' ||
										  d.data.name === 'PA-X Gender' ||
										  d.data.type === 'paax_code'
										? 5
										: 2}
						/>
					{/each}

					<!-- Downward Links -->
					{#each linksDown as d}
					{console.log(d)}
						<path
							d={`M${d.x},${yCenter + d.y}
						C${d.x},${yCenter + d.parent.y}
						${d.parent.x},${yCenter + d.parent.y + 60}
						${d.parent.x},${yCenter + d.parent.y}`}
							fill="none"
							stroke={highlightedLinks.has(`${d.parent.data.id}→${d.data.id}`)
								? 'orange'
								: d.data.name == 'conflict'
									? 'red'
									: 'white'}
							stroke-width="1.5"
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
								<!-- <text x="13" y="5" font-size="12" fill="gray"> {'war'}</text> -->
							{:else}
								<Icons {d} which_icon={'img/agt.png'} />
								<!-- <text x="13" y="5" font-size="12" fill="gray"> {'agt'}</text> -->
							{/if}
						</g>
					{/each}
				</g>
			</svg>
		{/if}
	</div>
	<div class="overview">
		<div class="ppl">
			{#if innerWidthPpl}
				<svg width={innerWidthPpl} {height}>
					<g transform={`translate(${-5}, ${-5})`}>
						{#each circles as d, i}
							<!-- <circle cx={d.x} cy={d.y} r={d.r} fill="#4FD1C5" /> -->
							<foreignObject x={d.x} y={d.y} width="12" height="12" aria-label="Icon">
								<div
									class="icon"
									style={i < 100
										? `background-image: url('img/peacerep_person.png');`
										: i < 300
											? `background-image: url('img/peace_person.png');`
											: `background-image: url('img/war_person.png');`}
								/>
							</foreignObject>
						{/each}
					</g>
				</svg>
			{/if}
		</div>
		<!-- <div class="time"></div> -->
	</div>
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
						<svg width="100%" height="100%" preserveAspectRatio="none">
							<g transform={`translate(16, ${segment_height / 2})`}>
								<text x="15" y="-15" font-size="12" fill="white">{d.data.name}</text>
								<line
									x1="1"
									y1="-32"
									x2="1"
									y2={segment_height - 24}
									stroke="orange"
									stroke-width="3"
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
		display: flex;
	}
	.tree {
		width: 70%;
		height: 100vh;
	}
	.overview {
		width: 30%;
		display: flex;
		flex-direction: column;
	}
	.ppl {
		width: 100%;
		height: 100%;
		/* background-color: red; */
	}
	#details {
		position: absolute;
		top: 4px;
		right: 0;
	}
	.detail-segment {
		color: white;
		display: flex;
		background-color: #1f1f1f;
		border-radius: 10px;
		border: solid 2px rgba(106, 106, 106, 0.237);
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
		width: 12px;
		height: 12px;
		background-size: cover;
		border-radius: 0px;
		transition: transform 0.2s ease;
	}
</style>
