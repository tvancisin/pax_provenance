<script>
	import * as d3 from 'd3';
	import Icons from './Icons.svelte';

	let width, height;
	const data = {
		name: 'Collect',
		children: [
			{
				name: 'Translate',
				children: [
					{
						name: 'Transcribe',
						children: [
							{
								name: 'Code',
								children: [
									{
										name: 'PA-X',
										children: [
											{
												name: 'Code',
												children: [
													{
														name: 'PA-X Gender',
														children: [
															{
																name: 'd3',
																children: [{ name: 'Scrollytelling', type: 'vis' }]
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
														children: [
															{
																name: 'd3',
																children: [{ name: 'PowerBi', type: 'vis' }]
															}
														]
													}
												]
											},
											{
												name: 'd3',
												children: [{ name: 'Messy Timeline', type: 'vis' }]
											},
											{
												name: 'd3',
												children: [{ name: 'Time & Space', type: 'vis' }]
											},
											{
												name: 'd3',
												children: [{ name: 'Sequence Comparison', type: 'vis' }]
											},
											{
												name: 'd3',
												children: [{ name: 'Data Overview', type: 'vis' }]
											},
											{
												name: 'PBi',
												children: [{ name: 'PA-X Tracker', type: 'vis' }]
											},
											{
												name: 'd3',
												children: [{ name: 'Infographics', type: 'vis' }]
											},
											{
												name: 'Code',
												children: [
													{
														name: 'PA-X Local',
														children: [
															{
																name: 'd3',
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
														children: [
															{
																name: 'PBi',
																children: [{ name: 'Tracker', type: 'vis' }]
															},
															{
																name: 'd3',
																children: [{ name: 'Actors-Processes', type: 'vis' }]
															},
															{
																name: 'd3',
																children: [{ name: 'Third Parties', type: 'vis' }]
															},
															{
																name: 'd3',
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
			{ name: 'branch A', children: [{ name: 'conflict' }] },
			{ name: 'branch B', children: [{ name: 'conflict' }] },
			{ name: 'branch C', children: [{ name: 'conflict' }] },
			{ name: 'branch D', children: [{ name: 'conflict' }] },
			{ name: 'branch E', children: [{ name: 'conflict' }] },
			{ name: 'branch F', children: [{ name: 'conflict' }] },
			{ name: 'branch G', children: [{ name: 'conflict' }] },
			{ name: 'branch H', children: [{ name: 'conflict' }] },
			{ name: 'branch I', children: [{ name: 'conflict' }] },
			{ name: 'branch J', children: [{ name: 'conflict' }] },
			{ name: 'branch K', children: [{ name: 'conflict' }] },
			{ name: 'branch L', children: [{ name: 'conflict' }] },
			{ name: 'branch M', children: [{ name: 'conflict' }] },
			{ name: 'branch N', children: [{ name: 'conflict' }] },
			{ name: 'branch O', children: [{ name: 'conflict' }] },
			{ name: 'branch P', children: [{ name: 'conflict' }] },
			{ name: 'branch R', children: [{ name: 'conflict' }] },
			{ name: 'branch S', children: [{ name: 'conflict' }] }
		]
	};

	let idCounter = 0;

	function assignIds(node) {
		node.id = `node-${idCounter++}`;
		if (node.children) {
			node.children.forEach(assignIds);
		}
	}

	$: assignIds(data);

	console.log(data);

	let margin = 60;

	$: innerWidth = width * 0.8;
	$: innerHeight = height - 2 * margin;

	$: yCenter = margin + innerHeight * 0.8;
	$: upHeight = yCenter - margin;
	$: downHeight = innerHeight - upHeight;
	$: xCenter = margin + innerWidth / 2;

	let nodesUp = [],
		linksUp = [];
	let nodesDown = [],
		linksDown = [];

	$: upwardCluster = d3.cluster().size([innerWidth, upHeight]);
	$: downwardCluster = d3.cluster().size([innerWidth, downHeight]);

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
	let highlightedLinks = new Set();

	function handleHoverEvent(e) {
		let node = e.detail.node;
		if (!node) {
			highlightedLinks = new Set();
			return;
		}

		const links = new Set();
		while (node.parent) {
			const key = `${node.parent.data.id}→${node.data.id}`;
			links.add(key);
			node = node.parent;
		}
		highlightedLinks = links;
	}

	$: console.log(highlightedLinks);
	
</script>

<div id="wrapper" bind:clientWidth={width} bind:clientHeight={height}>
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
						stroke={highlightedLinks.has(`${d.parent.data.id}→${d.data.id}`) ? 'white' : 'gray'}
						stroke-width={highlightedLinks.has(`${d.parent.data.id}→${d.data.id}`) ? 3 : 2}
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
						stroke={d.data.name === 'conflict' ? 'gray' : 'gray'}
					/>
				{/each}

				<!-- Upward Nodes -->
				{#each nodesUp as d}
					<g transform={`translate(${d.x}, ${yCenter - d.y})`}>
						{#if d.data.type === 'vis'}
							<Icons {d} which_icon="vis.png" on:hover={handleHoverEvent} />
						{:else if d.data.name == 'PAA-X' || d.data.name == 'PA-X' || d.data.name == 'PA-X Local' || d.data.name == 'Children & Youth' || d.data.name == 'PA-X Gender'}
							<Icons {d} which_icon="data.png" on:hover={handleHoverEvent} />
						{:else if d.data.name == 'Collect'}
							<Icons {d} which_icon={'collect.png'} />
						{:else if d.data.name == 'Translate'}
							<Icons {d} which_icon={'translate.png'} />
						{:else if d.data.name == 'Transcribe'}
							<Icons {d} which_icon={'transcribe.png'} />
						{:else if d.data.name == 'PBi'}
							<Icons {d} which_icon={'pbi.png'} />
						{:else if d.data.name == 'd3'}
							<Icons {d} which_icon={'d3.png'} />
						{:else if d.data.name == 'Code'}
							<Icons {d} which_icon={'annotation.png'} />
						{:else}
							<circle
								r="5"
								fill={d.children &&
								['PAA-X', 'PA-X Gender', 'PA-X', 'PA-X Local', 'Children & Youth'].includes(
									d.data.name
								)
									? 'gray'
									: d.children
										? 'gray'
										: 'gray'}
							/>
						{/if}
						<text
							x={d.children ? 15 : 5}
							y={d.children ? 5 : -10}
							font-size="12"
							fill={d.children &&
							['PAA-X', 'PA-X Gender', 'PA-X', 'PA-X Local', 'Children & Youth'].includes(
								d.data.name
							)
								? 'white'
								: d.children
									? 'gray'
									: 'orange'}
							transform={d.children ? 'rotate(0)' : 'rotate(-45)'}
						>
							{d.data.name}
						</text>
					</g>
				{/each}

				<!-- Downward Nodes -->
				{#each nodesDown as d}
					<g transform={`translate(${d.x}, ${yCenter + d.y})`}>
						{#if d.data.name == 'conflict'}
							<Icons {d} which_icon={'war.png'} />
							<text x="13" y="5" font-size="12" fill="red"> {'war'}</text>
						{:else}
							<Icons {d} which_icon={'agt.png'} />
							<text x="13" y="5" font-size="12" fill="steelblue"> {'agt'}</text>
						{/if}
					</g>
				{/each}
			</g>
		</svg>
	{/if}
	<div style="text-align: center;">
		<p>TODO</p>
		<p>alternate white and gray for branches</p>
		<p>Hover over a node, highlight all its parents.</p>
		<p>How to interact with links and nodes?</p>
		<p>What filtering to apply?</p>
		<p>How to restructure the tree for different queries?</p>
		<p>Which part to quantify and which to qualify?</p>
		<p>How do the same icons differ and how to show?</p>
		<p>What to highlight before user goes to individual vis?</p>
		<!-- <iframe
			title="Miro Board"
			src="https://miro.com/app/live-embed/uXjVIYrT328=/?moveToViewport=-208202,-17495,146090,66767&embedId=356675018147"
			frameborder="0"
			scrolling="no"
			allow="fullscreen; clipboard-read; clipboard-write"
			allowfullscreen
		></iframe> -->
	</div>
</div>

<style>
	#wrapper {
		width: 100%;
		height: 100vh;
	}
	iframe {
		width: 90%;
		height: 100vh;
		border: none;
	}
	p {
		color: white;
	}
</style>
