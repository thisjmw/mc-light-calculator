<template>
	<div>
		<input type="radio" id="light-control" :value="blockTypeEnum.LIGHT" v-model="control">
		<label for="light-control">Light</label>
		<br />
		<input type="radio" id="wall-control" :value="blockTypeEnum.WALL" v-model="control">
		<label for="wall-control">Wall</label>
		<br />
		<button @click="$emit('reset')">Reset</button>
	</div>
</template>

<script>
	import { blockTypeEnum } from '../data'

	export default {
		name: 'LightGridControlSelector',
		emits: [
			'change-control',
			'reset'
		],
		data() {
			return {
				control: blockTypeEnum.LIGHT,
				blockTypeEnum
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