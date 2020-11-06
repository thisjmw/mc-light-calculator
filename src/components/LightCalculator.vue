<template>
	<div id="light-calculator">
		<view-grid-control-selector
			v-show="showControls"
			@change-control="changeControl"
			@change-view="changeView"
			@change-dimensions="changeDimensions"
			@reset="reset"
		/>
		<div class="view-grid-container">
			<div id="view-grid" :class="{ expanded: !showControls }">
				<view-grid
					:width="gridWidth"
					:height="gridHeight"
					:lights="lights"
					:walls="walls"
					:elevations="elevations"
					:control="controlType"
					:grid-view="gridView"
					@add-light="addLight"
					@remove-light="removeLight"
					@add-wall="addWall"
					@remove-wall="removeWall"
					@change-elevation="changeElevation"
					@reset="reset"
				/>
			</div>
		</div>
	</div>
</template>

<script>
	import ViewGrid from './ViewGrid'
	import ViewGridControlSelector from './ViewGridControlSelector'
	import {
		controlTypeEnum,
		gridViewEnum
	} from '../data'
	import { clamp } from '../util'

	const MIN_ELEVATION = 0
	const MAX_ELEVATION = 127

	export default {
		name: 'LightCalculator',
		components: {
			ViewGrid,
			ViewGridControlSelector
		},
		props: {
			showControls: Boolean
		},
		data() {
			return {
				lights: [],
				walls: [],
				elevations: {},
				gridWidth: 40,
				gridHeight: 40,
				controlType: controlTypeEnum.LIGHT,
				gridView: gridViewEnum.LIGHT
			}
		},
		methods: {
			addLight(coordinates) {
				this.lights.push({ x: coordinates.x, y: coordinates.y, strength: 14 })
			},
			removeLight(coordinates) {
				this.lights = this.lights.filter(light => !(light.x === coordinates.x && light.y === coordinates.y))
			},
			addWall(coordinates) {
				this.walls.push({ x: coordinates.x, y: coordinates.y })
			},
			removeWall(coordinates) {
				this.walls = this.walls.filter(wall => !(wall.x === coordinates.x && wall.y === coordinates.y))
			},
			changeElevation(coordinates) {
				const key = `${coordinates.x},${coordinates.y}`
				if (!this.elevations[key] && this.controlType === controlTypeEnum.INCREASE_ELEVATION) {
					this.$set(this.elevations, key, 1)
				} else {
					const elevation = this.elevations[key]
					if (this.controlType === controlTypeEnum.INCREASE_ELEVATION) {
						this.$set(this.elevations, key, clamp(elevation + 1, MIN_ELEVATION, MAX_ELEVATION))
					} else if (this.controlType === controlTypeEnum.DECREASE_ELEVATION) {
						this.$set(this.elevations, key, clamp(elevation - 1, MIN_ELEVATION, MAX_ELEVATION))
					}
				}
			},
			reset() {
				this.lights = []
				this.walls = []
				this.elevations = {}
			},
			changeControl(controlType) {
				this.controlType = controlType
			},
			changeView(view) {
				this.gridView = view
			},
			changeDimensions(dimensions) {
				this.gridWidth = dimensions.width
				this.gridHeight = dimensions.height
			}
		}
	}
</script>

<style scoped>
	#light-calculator {
		max-width: 100%;
		display: flex;
		flex-flow: column;
		height: 100%;
	}

	.view-grid-container {
		flex: 1 1 auto;
		display: flex;
		flex-direction: column;
		justify-content: center;
	}

	#view-grid {
		max-height: calc(100vh - 9rem);
		flex: 0 1 auto;
		max-width: 100vw;
		overflow: auto;
		scrollbar-width: thin !important;
		scrollbar-color: white black;
		display: flex;
		flex-direction: row;
		justify-content: center;
	}

	#view-grid.expanded {
		max-height: calc(100vh - 3rem);
	}
</style>