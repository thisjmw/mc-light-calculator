<template>
	<div>
		<input type="radio" id="light-control" :value="blockTypeEnum.LIGHT" v-model="control">
		<label for="light-control">Light</label>
		<br />
		<input type="radio" id="wall-control" :value="blockTypeEnum.WALL" v-model="control">
		<label for="wall-control">Wall</label>
		<br />
		<button @click="$emit('reset')">Reset</button>
		<br />
		<input type="radio" id="light-view-control" :value="gridViewEnum.LIGHT" v-model="gridView">
		<label for="light-view-control">Light</label>
		<br />
		<input type="radio" id="elevation-view-control" :value="gridViewEnum.ELEVATION" v-model="gridView">
		<label for="elevation-view-control">Elevation</label>
	</div>
</template>

<script>
	import {
		blockTypeEnum,
		gridViewEnum
	} from '../data'

	export default {
		name: 'LightGridControlSelector',
		emits: [
			'change-control',
			'change-view',
			'reset'
		],
		data() {
			return {
				control: blockTypeEnum.LIGHT,
				gridView: gridViewEnum.LIGHT,
				blockTypeEnum,
				gridViewEnum
			}
		},
		watch: {
			control(controlType) {
				this.$emit('change-control', controlType)
			},
			gridView(view) {
				this.$emit('change-view', view)
			}
		}
	}
</script>

<style scoped>

</style>