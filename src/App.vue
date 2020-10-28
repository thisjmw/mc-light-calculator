<template>
	<div id="app">
		<LightGridControlSelector
			@change-control="changeControl"
			@change-view="changeView"
			@reset="reset"
		/>
		<LightGrid
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
			@increase-elevation="increaseElevation"
			@reset="reset"
		/>
	</div>
</template>

<script>
	import LightGrid from './components/LightGrid'
	import LightGridControlSelector from './components/LightGridControlSelector'
	import {
		blockTypeEnum,
		gridViewEnum
	} from './data'

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
				elevations: {},
				gridWidth: 40,
				gridHeight: 40,
				controlType: blockTypeEnum.LIGHT,
				gridView: gridViewEnum.LIGHT
			}
		},
		methods: {
			addLight(coordinates) {
				this.lights.push({ ...coordinates, strength: 14 })
			},
			removeLight(coordinates) {
				this.lights = this.lights.filter(light => !(light.x === coordinates.x && light.y === coordinates.y))
			},
			addWall(coordinates) {
				this.walls.push({ ...coordinates })
			},
			removeWall(coordinates) {
				this.walls = this.walls.filter(wall => !(wall.x === coordinates.x && wall.y === coordinates.y))
			},
			increaseElevation(coordinates) {
				const key = `${coordinates.x},${coordinates.y}`
				if (this.elevations[key]) {
					this.elevations[key]++
				} else {
					this.$set(this.elevations, key, 1)
				}
				const qLightToUpdate = this.lights.filter(light => light.x === coordinates.x && light.y === coordinates.y)
				if (qLightToUpdate.length) {
					const updatedLight = { ...qLightToUpdate[0] }
					updatedLight.elevation++
					this.removeLight({ x: updatedLight.x, y: updatedLight.y })
					this.addLight({ ...updatedLight })
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
