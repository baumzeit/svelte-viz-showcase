<script lang="ts" context="module">
	export type ContourChartData = { x: number; y: number }[];
</script>

<script lang="ts">
	import { extent } from 'd3-array';
	import { axisLeft, axisBottom } from 'd3-axis';
	import { contourDensity } from 'd3-contour';
	import { geoPath } from 'd3-geo';
	import { scaleLinear } from 'd3-scale';

	import 'd3-transition';

	import type { ChartProps } from '$components/chart/Chart.svelte';

	export let config: ChartProps<ContourChartData> & {
		xDomain?: [number, number];
		yDomain?: [number, number];
		bandwidth?: number;
		threshold?: number;
	};

	$: ({ data, width, height, root, bandwidth = 25, threshold = 30 } = config);
	$: xDomain = config.xDomain || (extent(data, (d) => d.x) as any);
	$: yDomain = config.yDomain || (extent(data, (d) => d.y) as any);

	$: xScale = scaleLinear().domain(xDomain).nice().rangeRound([0, config.width]);
	$: yScale = scaleLinear().domain(yDomain).nice().rangeRound([config.height, 0]);

	$: {
		const contours = contourDensity<ContourChartData[number]>()
			.x((d) => xScale(d.x))
			.y((d) => yScale(d.y))
			.size([width, height])
			.bandwidth(bandwidth)
			.thresholds(threshold)(data);

		root
			.selectAll('path')
			.data(contours)
			.join('path')
			.classed('stroke-cyan-700', true)
			.attr('stroke-width', (d, i) => (i % 5 ? 0.25 : 1))
			.attr('d', geoPath());
	}

	$: {
		root
			.selectAll('g.axisLeft')
			.data([data])
			.join('g')
			.classed('axisLeft', true)
			.call(axisLeft(yScale).tickSizeOuter(0) as any)
			.call((g) => g.select('.domain').remove())
			.call((g) =>
				g
					.select('.tick:last-of-type text')
					.clone()
					.attr('x', 3)
					.attr('text-anchor', 'start')
					.attr('font-weight', 'bold')
					.text('Erupting (min.)')
			);

		root
			.selectAll('g.axisBottom')
			.data([data])
			.join('g')
			.classed('axisBottom', true)
			.attr('transform', `translate(0,${height})`)
			.call(axisBottom(xScale).tickSizeOuter(0) as any)
			.call((g) => g.select('.domain').remove())
			.call((g) =>
				g
					.select('.tick:last-of-type text')
					.clone()
					.attr('y', -3)
					.attr('dy', null)
					.attr('font-weight', 'bold')
					.text('Idle (min.)')
			);
		// .selectAll('text');
		// .call((el) => {
		//   const text = el.text()
		//   el.attr('data-value', text)
		// })
	}
</script>

<!-- 
{#each $contours as path, i}
	<path class="stroke-cyan-700" stroke-width={i % 5 ? 0.25 : 1} d={geoPath()(path)} />
{/each} -->
