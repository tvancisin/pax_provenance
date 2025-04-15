<script>
	import * as d3 from 'd3';
	import Icons from './Icons.svelte';

	let width,
		height,
		details_width = 1,
		details_data = [],
		clicked = null;
	const data = {
		name: 'Collect',
		ppl: 2,
		time: '20',
		children: [
			{
				name: 'Translate',
				ppl: 5,
				time: '10',
				children: [
					{
						name: 'Transcribe',
						ppl: 10,
						time: '40',
						children: [
							{
								name: 'Code',
								ppl: 3,
								time: '100',
								children: [
									{
										name: 'PA-X',
										type: 'db',
										ppl: 5,
										time: '100',
										children: [
											{
												name: 'Code',
												ppl: 3,
												time: '50',
												children: [
													{
														name: 'PA-X Gender',
														type: 'db',
														ppl: 3,
														time: '10',
														children: [
															{
																name: 'd3',
																type: 'prog',
																ppl: 1,
																time: '3',
																children: [
																	{ name: 'Scrollytelling', type: 'vis', ppl: 4, time: '2' }
																]
															},
															{ name: 'PeaceFem' }
														]
													}
												]
											},
											{
												name: 'Code',
												children: [
													{
														name: 'Children & Youth',
														type: 'db',
														children: [
															{
																name: 'PBi',
																type: 'prog',
																children: [{ name: 'PowerBi', type: 'vis' }]
															}
														]
													}
												]
											},
											{
												name: 'd3',
												type: 'prog',
												children: [{ name: 'Messy Timeline', type: 'vis' }]
											},
											{
												name: 'd3',
												type: 'prog',
												children: [{ name: 'Time & Space', type: 'vis' }]
											},
											{
												name: 'd3',
												type: 'prog',
												children: [{ name: 'Sequence Comparison', type: 'vis' }]
											},
											{
												name: 'd3',
												type: 'prog',
												children: [{ name: 'Data Overview', type: 'vis' }]
											},
											{
												name: 'PBi',
												type: 'prog',
												children: [{ name: 'PA-X Tracker', type: 'vis' }]
											},
											{
												name: 'd3',
												type: 'prog',
												children: [{ name: 'Infographics', type: 'vis' }]
											},
											{
												name: 'Code',
												children: [
													{
														name: 'PA-X Local',
														type: 'db',
														children: [
															{
																name: 'd3',
																type: 'prog',
																children: [{ name: 'Map', type: 'vis' }]
															}
														]
													}
												]
											},
											{
												name: 'Code',
												children: [
													{
														name: 'PAA-X',
														type: 'db',
														children: [
															{
																name: 'PBi',
																type: 'prog',
																children: [{ name: 'Tracker', type: 'vis' }]
															},
															{
																name: 'd3',
																type: 'prog',
																children: [{ name: 'Actors-Processes', type: 'vis' }]
															},
															{
																name: 'd3',
																type: 'prog',
																children: [{ name: 'Third Parties', type: 'vis' }]
															},
															{
																name: 'd3',
																type: 'prog',
																children: [{ name: 'Network', type: 'vis' }]
															}
														]
													}
												]
											}
										]
									}
								]
							}
						]
					}
				]
			}
		],
		downward: [
			{ name: 'agt', children: [{ name: 'conflict' }] },
			{ name: 'agt', children: [{ name: 'conflict' }] },
			{ name: 'agt', children: [{ name: 'conflict' }] },
			{ name: 'agt', children: [{ name: 'conflict' }] },
			{ name: 'agt', children: [{ name: 'conflict' }] },
			{ name: 'agt', children: [{ name: 'conflict' }] },
			{ name: 'agt', children: [{ name: 'conflict' }] },
			{ name: 'agt', children: [{ name: 'conflict' }] },
			{ name: 'agt', children: [{ name: 'conflict' }] },
			{ name: 'agt', children: [{ name: 'conflict' }] },
			{ name: 'agt', children: [{ name: 'conflict' }] },
			{ name: 'agt', children: [{ name: 'conflict' }] },
			{ name: 'agt', children: [{ name: 'conflict' }] },
			{ name: 'agt', children: [{ name: 'conflict' }] }
		]
	};

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
		.range([0, innerWidth / 2]);

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

	function reset() {
		innerWidth = width * 0.9;
		details_width = 1;
		fullChain = [];
		clicked = null;
		highlightedLinks = new Set();
	}
</script>

<div id="wrapper" bind:clientWidth={width} bind:clientHeight={height}>
	<button id="reset" on:click={reset}>reset</button>
	{#if width !== undefined || height !== undefined}
		<svg {width} {height}>
			<g transform={`translate(${0}, ${margin})`}>
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
								which_icon="vis.png"
								on:hover={handleHoverEvent}
								on:click={handleClickEvents}
							/>
						{:else if d.data.type == 'db'}
							<Icons
								{d}
								which_icon="data.png"
								on:hover={handleHoverEvent}
								on:click={handleClickEvents}
							/>
						{:else if d.data.type == 'prog'}
							<Icons
								{d}
								which_icon="prog.png"
								on:hover={handleHoverEvent}
								on:click={handleClickEvents}
							/>
						{:else if d.data.name == 'Collect'}
							<Icons
								{d}
								which_icon="collect.png"
								on:hover={handleHoverEvent}
								on:click={handleClickEvents}
							/>
						{:else if d.data.name == 'Translate'}
							<Icons
								{d}
								which_icon="translate.png"
								on:hover={handleHoverEvent}
								on:click={handleClickEvents}
							/>
						{:else if d.data.name == 'Transcribe'}
							<Icons
								{d}
								which_icon="transcribe.png"
								on:hover={handleHoverEvent}
								on:click={handleClickEvents}
							/>
						{:else if d.data.name == 'Code'}
							<Icons
								{d}
								which_icon="annotation.png"
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
							<Icons {d} which_icon={'war.png'} />
							<text x="13" y="5" font-size="12" fill="gray"> {'war'}</text>
						{:else}
							<Icons {d} which_icon={'agt.png'} />
							<text x="13" y="5" font-size="12" fill="gray"> {'agt'}</text>
						{/if}
					</g>
				{/each}
			</g>
		</svg>
	{/if}
	<div id="details">
		<svg width={details_width} {height}>
			<line x1="10" y1={margin} x2="10" y2={height - margin} stroke="white" stroke-width="3" />

			{#each fullChain as d, i}
				<g transform={`translate(${10}, ${(height / (fullChain.length + 1)) * (i + 1)})`}>
					<!-- Calculate y position by dividing total height evenly -->
					<!-- <text x="50" y="5" fill="white" font-size="12">
						{'remove/limit color. switch between two colors?'}
					</text> -->
					<!-- Draw one rectangle per person -->
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
							<div class="icon" style={`background-image: url('person.png');`}></div>
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

					{#if d.data.type === 'vis'}
						<Icons {d} which_icon="vis.png" />
					{:else if d.data.type == 'db'}
						<Icons {d} which_icon="data.png" />
					{:else if d.data.type == 'prog'}
						<Icons {d} which_icon="prog.png" />
					{:else if d.data.name == 'Collect'}
						<Icons {d} which_icon="collect.png" />
					{:else if d.data.name == 'Translate'}
						<Icons {d} which_icon="translate.png" />
					{:else if d.data.name == 'Transcribe'}
						<Icons {d} which_icon="transcribe.png" />
					{:else if d.data.name == 'Code'}
						<Icons {d} which_icon="annotation.png" />
					{:else if d.data.name == 'agt'}
						<Icons {d} which_icon="agt.png" />
					{:else if d.data.name == 'conflict'}
						<Icons {d} which_icon="war.png" />
					{:else}
						<circle r="5" fill="gray" />
					{/if}
				</g>
			{/each}
		</svg>
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
	}
	#reset {
		position: absolute;
		top: 0;
		left: 0;
	}
	iframe {
		width: 90%;
		height: 100vh;
		border: none;
	}
	p {
		color: white;
	}
	.icon {
		position: absolute;
		width: 10px;
		height: 10px;
		background-size: cover;
		border-radius: 4px;
		transition: transform 0.2s ease;
	}
</style>
