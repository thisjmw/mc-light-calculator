<template>
	<div id="app">
		<LightGridControlSelector
			@change-control="changeControl"
			@reset="reset"
		/>
		<LightGrid
			:width="gridWidth"
			:height="gridHeight"
			:lights="lights"
			:walls="walls"
			:control="controlType"
			@add-light="addLight"
			@remove-light="removeLight"
			@add-wall="addWall"
			@remove-wall="removeWall"
			@reset="reset"
		/>
	</div>
</template>

<script>
	import LightGrid from './components/LightGrid'
	import LightGridControlSelector from './components/LightGridControlSelector'
	import { blockTypeEnum } from './data'

	export default {
		name: 'App',
		components: {
			LightGrid,
			LightGridControlSelector
		},
		data() {
			return {
				lights: [],
				walls: [],
				gridWidth: 40,
				gridHeight: 40,
				controlType: blockTypeEnum.LIGHT
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
			reset() {
				this.lights = []
				this.walls = []
			},
			changeControl(controlType) {
				this.controlType = controlType
			}
		}
	}
</script>

<style>
	#app {
		font-family: Avenir,Helvetica, Arial, sans-serif;
		-webkit-font-smoothing: antialiased;
		-moz-osx-font-smoothing: grayscale;
	}
</style>
