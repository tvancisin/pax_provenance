<script>
	import * as d3 from 'd3';

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

		// rootUp.each((d) => {
		// 	if (d.data.name === 'Translate') {
		// 		d.y -= 25; // Bring transcribe node closer to collect
		// 	}
		// 	if (d.data.name === 'Transcribe') {
		// 		d.y -= 45; // Bring transcribe node closer to collect
		// 	}
		// 	if (d.data.name === 'Code') {
		// 		d.y -= 70; // Bring transcribe node closer to collect
		// 	}
		// 	if (d.data.name === 'PA-X') {
		// 		d.y -= 100; // Bring pax node closer to transcribe
		// 	}
		// 	if (
		// 		d.data.name === 'PAA-X' ||
		// 		d.data.name === 'PA-X Gender' ||
		// 		d.data.name === 'PA-X Local' ||
		// 		d.data.name === 'Children & Youth'
		// 	) {
		// 		d.y -= 50; // Bring paax node closer to transcribe
		// 	}
		// });

		// Find the "PAA-X" node and apply arch positioning to its children
		const paxNode = rootUp.descendants().find((d) => d.data.name === 'PA-X');
		// const paaxNode = rootUp.descendants().find((d) => d.data.name === 'PAA-X');
		// const genderNode = rootUp.descendants().find((d) => d.data.name === 'PA-X Gender');

		// if (paxNode) {
		// 	const levelY =
		// 		paxNode.children?.find((d) => d.data.name === 'PA-X Gender')?.y ?? paxNode.y + 50;
		// 		console.log(levelY);
				
		// 	const targetNames = [
		// 		'd3', 'PBi'
		// 		// 'Time & Space',
		// 		// 'Sequence Comparison',
		// 		// 'Data Overview',
		// 		// 'PA-X Tracker',
		// 		// 'Infographics'
		// 	];

		// 	paxNode.children?.forEach((child) => {
		// 		if (targetNames.includes(child.data.name)) {
		// 			child.y = levelY;
		// 		}
		// 	});
		// }

		// if (paaxNode) {
		// 	const children = paaxNode.children || [];
		// 	const radius = 100;
		// 	const total = children.length;
		// 	const startAngle = Math.PI; // Start from left
		// 	const endAngle = 0; // End at right
		// 	const angleStep = (endAngle - startAngle) / (total + 1);

		// 	children.forEach((child, i) => {
		// 		const angle = startAngle + angleStep * (i + 1);
		// 		child.x = paaxNode.x + radius * Math.cos(angle);
		// 		child.y = paaxNode.y + radius * Math.sin(angle);
		// 	});
		// }

		// if (genderNode) {
		// 	const children = genderNode.children || [];
		// 	const radius = 80;
		// 	const total = children.length;
		// 	const startAngle = Math.PI; // Start from left
		// 	const endAngle = 0; // End at right
		// 	const angleStep = (endAngle - startAngle) / (total + 1);

		// 	children.forEach((child, i) => {
		// 		const angle = startAngle + angleStep * (i + 1);
		// 		child.x = genderNode.x + radius * Math.cos(angle);
		// 		child.y = genderNode.y + radius * Math.sin(angle);
		// 	});
		// }

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
							? 2
							: d.data.name === 'PAA-X'
								? 2
								: d.data.name === 'PA-X Gender'
									? 2
									: 2}
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
						stroke="steelblue"
					/>
				{/each}

				<!-- Upward Nodes -->
				{#each nodesUp as d}
					<g transform={`translate(${d.x}, ${yCenter - d.y})`}>
						{#if d.data.type === 'vis'}
							<!-- <rect x="-5" y="-5" width="10" height="10" fill="orange" /> -->
							<foreignObject x="-8" y="-8" width="18" height="18">
								<div
									xmlns="http://www.w3.org/1999/xhtml"
									style="
										width: 18px;
										height: 18px;
										background-image: url('vis.png');
										background-size: cover;
										border-radius: 4px;
									"
								></div>
							</foreignObject>
						{:else if d.data.name == 'PAA-X' || d.data.name == 'PA-X' || d.data.name == 'PA-X Local' || d.data.name == 'Children & Youth' || d.data.name == 'PA-X Gender'}
							<foreignObject x="-8" y="-8" width="18" height="18">
								<div
									xmlns="http://www.w3.org/1999/xhtml"
									style="
										width: 18px;
										height: 18px;
										background-image: url('data.png');
										background-size: cover;
										border-radius: 4px;
									"
								></div>
							</foreignObject>
						{:else if d.data.name == 'Collect'}
							<foreignObject x="-8" y="-8" width="18" height="18">
								<div
									xmlns="http://www.w3.org/1999/xhtml"
									style="
										width: 18px;
										height: 18px;
										background-image: url('collect.png');
										background-size: cover;
										border-radius: 4px;
									"
								></div>
							</foreignObject>
						{:else if d.data.name == 'Translate'}
							<foreignObject x="-8" y="-8" width="18" height="18">
								<div
									xmlns="http://www.w3.org/1999/xhtml"
									style="
										width: 18px;
										height: 18px;
										background-image: url('translate.png');
										background-size: cover;
										border-radius: 4px;
									"
								></div>
							</foreignObject>
						{:else if d.data.name == 'Transcribe'}
							<foreignObject x="-8" y="-8" width="18" height="18">
								<div
									xmlns="http://www.w3.org/1999/xhtml"
									style="
										width: 18px;
										height: 18px;
										background-image: url('transcribe.png');
										background-size: cover;
										border-radius: 4px;
									"
								></div>
							</foreignObject>
						{:else if d.data.name == 'Code'}
							<foreignObject x="-8" y="-8" width="18" height="18">
								<div
									xmlns="http://www.w3.org/1999/xhtml"
									style="
										width: 18px;
										height: 18px;
										background-image: url('annotation.png');
										background-size: cover;
										border-radius: 4px;
									"
								></div>
							</foreignObject>
						{:else}
							<circle
								r="5"
								fill={d.children &&
								['PAA-X', 'PA-X Gender', 'PA-X', 'PA-X Local', 'Children & Youth'].includes(
									d.data.name
								)
									? 'white'
									: d.children
										? 'gray'
										: 'orange'}
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
							<foreignObject x="-8" y="-8" width="18" height="18">
								<div
									xmlns="http://www.w3.org/1999/xhtml"
									style="
										width: 18px;
										height: 18px;
										background-image: url('war.png');
										background-size: cover;
										border-radius: 4px;
									"
								></div>
							</foreignObject>
							<text x="13" y="5" font-size="12" fill="red"> {'war'}</text>
						{:else}
							<foreignObject x="-8" y="-8" width="18" height="18">
								<div
									xmlns="http://www.w3.org/1999/xhtml"
									style="
										width: 18px;
										height: 18px;
										background-image: url('agt.png');
										background-size: cover;
										border-radius: 4px;
									"
								></div>
							</foreignObject>
							<text x="13" y="5" font-size="12" fill="steelblue"> {'agt'}</text>
						{/if}
					</g>
				{/each}

				<!-- Central node -->
				<!-- <g transform={`translate(${xCenter}, ${yCenter})`}>
					<circle r="5" fill="gray" />
				</g> -->
			</g>
		</svg>
	{/if}
	<div style="text-align: center;">
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
