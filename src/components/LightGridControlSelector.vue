<template>
	<div>
		<input type="radio" id="light-control" value="light" v-model="control">
		<label for="light-control">Light</label>
		<br />
		<input type="radio" id="wall-control" value="wall" v-model="control">
		<label for="wall-control">Wall</label>
	</div>
</template>

<script>
	export default {
		name: 'LightGridControlSelector',
		emits: [
			'change-control'
		],
		data() {
			return {
				control: 'light'
			}
		},
		watch: {
			control(controlType) {
				this.$emit('change-control', controlType)
			}
		}
	}
</script>

<style scoped>

</style>