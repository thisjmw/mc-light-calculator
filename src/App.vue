<template>
	<div id="app">
		<LightGrid :width="40" :height="40" :lights="lights" @add-light="addLight"></LightGrid>
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
				lights: []
			}
		},
		methods: {
			addLight(data) {
				this.lights.push({ x: data.x, y: data.y, strength: 14 })
			}
		}
	}
</script>

<style>

</style>
