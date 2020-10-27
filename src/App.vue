<template>
	<div id="app">
		<LightGrid
			:width="gridWidth"
			:height="gridHeight"
			:lights="lights"
			@add-light="addLight"
			@remove-light="removeLight"
			@reset="reset"
		></LightGrid>
	</div>
</template>

<script>
	import LightGrid from './components/LightGrid'

	export default {
		name: 'App',
		components: {
			LightGrid
		},
		data() {
			return {
				lights: [],
				gridWidth: 40,
				gridHeight: 40
			}
		},
		methods: {
			addLight(coordinates) {
				this.lights.push({ x: coordinates.x, y: coordinates.y, strength: 14 })
			},
			removeLight(coordinates) {
				this.lights = this.lights.filter(light => !(light.x === coordinates.x && light.y === coordinates.y))
			},
			reset() {
				this.lights = []
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
