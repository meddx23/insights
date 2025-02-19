<script setup>
import widgets from '@/widgets/widgets'
import { computed, inject } from 'vue'
import ChartActionButtons from './ChartActionButtons.vue'
import ChartSectionEmptySvg from './ChartSectionEmptySvg.vue'
import ChartTypeSelector from './ChartTypeSelector.vue'

const query = inject('query')

const showChart = computed(() => {
	return (
		query.chart.doc?.name &&
		query.formattedResults?.length &&
		query.doc.status !== 'Pending Execution'
	)
})

const emptyMessage = computed(() => {
	if (query.doc.status == 'Pending Execution') {
		return 'Execute the query to see the chart'
	}
	if (!query.formattedResults?.length) {
		return 'No results found'
	}
	return 'Pick a chart type to get started'
})

const chart = computed(() => {
	const chart_type = query.chart.doc.chart_type
	const guessedChart = query.chart.getGuessedChart(chart_type)
	const options = Object.assign({}, guessedChart.options, query.chart.doc.options)
	return {
		type: guessedChart.chart_type,
		data: query.chart.data,
		options: chart_type == 'Auto' ? guessedChart.options : options,
		component: widgets.getComponent(guessedChart.chart_type),
	}
})
</script>

<template>
	<div v-if="query.chart.doc?.name" class="flex flex-1 flex-col gap-4 overflow-hidden">
		<div
			v-if="!showChart"
			class="flex flex-1 flex-col items-center justify-center rounded border"
		>
			<ChartSectionEmptySvg></ChartSectionEmptySvg>
			<span class="text-gray-500">{{ emptyMessage }}</span>
		</div>
		<template v-else>
			<div class="flex w-full flex-shrink-0 justify-between">
				<ChartTypeSelector></ChartTypeSelector>
				<ChartActionButtons></ChartActionButtons>
			</div>
			<div class="flex w-full flex-1 overflow-hidden rounded border">
				<component
					v-if="chart.type"
					ref="widget"
					:is="chart.component"
					:options="chart.options"
					:data="chart.data"
					:key="JSON.stringify(query.chart.doc)"
				/>
			</div>
		</template>
	</div>
</template>
