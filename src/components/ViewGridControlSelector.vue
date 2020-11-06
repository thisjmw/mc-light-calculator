<template>
	<div class="control-container">
		<div class="container-row top">
			<div class="container-col left">
				<div class="control">
					<label for="view-control">View</label>
					<select id="view-control" v-model="gridView" :required="true">
						<option
							v-for="option in viewOptions.options"
							:key="option.id"
							:value="option.value"
							:selected="option.value === viewOptions.default"
						>
							{{ option.name }}
						</option>
					</select>
				</div>
				<div class="control">
					<label for="light-control">Control</label>
					<select id="light-control" v-model="control" :required="true">
						<option
							v-for="option in scopedControlOptions.options"
							:key="option.id"
							:value="option.value"
							:selected="option.value === scopedControlOptions.default(gridView)"
						>
							{{ option.name }}
						</option>
					</select>
				</div>
			</div>
			<div class="container-col right">
				<div class="control">
					<label for="height-control">Height</label>
					<input
						type="number"
						id="height-control"
						v-model="height"
						:min="dimensions.height.min"
						:max="dimensions.height.max"
						@change="dimensionsChanged"
					/>
				</div>
				<div class="control">
					<label for="width-control">Width</label>
					<input
						type="number"
						id="width-control"
						v-model="width"
						:min="dimensions.width.min"
						:max="dimensions.width.max"
						@change="dimensionsChanged"
					/>
				</div>
			</div>
		</div>
		<div class="container-row bottom">
			<button @click="reset">Reset</button>
		</div>


	</div>
</template>

<script>
	import { clamp } from '../util'
	import {
		blockTypeEnum,
		controlTypeEnum,
		gridViewEnum
	} from '../data'

	const viewOptions = {
		default: gridViewEnum.LIGHT,
		options: [
			{
				id: 'light-view',
				name: 'Light',
				value: gridViewEnum.LIGHT
			},
			{
				id: 'elevation-view',
				name: 'Elevation',
				value: gridViewEnum.ELEVATION
			}
		]
	}

	const controlOptions = {
		default(view) {
			switch (view) {
				case gridViewEnum.LIGHT: return controlTypeEnum.LIGHT
				case gridViewEnum.ELEVATION: return controlTypeEnum.INCREASE_ELEVATION
			}
		},
		options: [
			{
				id: 'light-control',
				name: 'Light',
				value: controlTypeEnum.LIGHT,
				viewFilter: gridViewEnum.LIGHT
			},
			{
				id: 'wall-control',
				name: 'Wall',
				value: controlTypeEnum.WALL,
				viewFilter: gridViewEnum.LIGHT
			},
			{
				id: 'increase-elevation-control',
				name: 'Increase Elevation',
				value: controlTypeEnum.INCREASE_ELEVATION,
				viewFilter: gridViewEnum.ELEVATION
			},
			{
				id: 'decrease-elevation-control',
				name: 'Decrease Elevation',
				value: controlTypeEnum.DECREASE_ELEVATION,
				viewFilter: gridViewEnum.ELEVATION
			}
		]
	}

	const dimensions = {
		height: {
			default: 40,
			min: 1,
			max: 100
		},
		width: {
			default: 40,
			min: 1,
			max: 100
		}
	}

	export default {
		name: 'LightGridControlSelector',
		emits: [
			'change-control',
			'change-view',
			'change-dimensions',
			'reset'
		],
		data() {
			return {
				controlOptions,
				viewOptions,
				control: controlOptions.default(viewOptions.default),
				gridView: viewOptions.default,
				height: dimensions.height.default,
				width: dimensions.width.default,
				dimensions,
				blockTypeEnum,
				gridViewEnum
			}
		},
		computed: {
			scopedControlOptions() {
				return {
					default: this.controlOptions.default,
					options: this.controlOptions.options.filter(option => option.viewFilter === this.gridView)
				}
			}
		},
		watch: {
			control(controlType) {
				this.$emit('change-control', controlType)
			},
			gridView(view) {
				this.viewChanged()
				this.$emit('change-view', view)
			}
		},
		methods: {
			viewChanged() {
				this.control = controlOptions.default(this.gridView)
			},
			dimensionsChanged() {
				this.height = clamp(this.height, dimensions.height.min, dimensions.height.max)
				this.width = clamp(this.width, dimensions.width.min, dimensions.width.max)
				this.$emit('change-dimensions', { height: this.height, width: this.width })
			},
			reset() {
				this.height = dimensions.height.default
				this.width = dimensions.width.default
				this.gridView = viewOptions.default
				this.viewChanged()
				this.dimensionsChanged()
				this.$emit('reset')
			}
		}
	}
</script>

<style scoped>
	.control-container {
		flex: 0 1 auto;
		height: 6rem;
		display: flex;
		flex-flow: column;
		padding: 0.25rem 0;
		box-sizing: border-box;
		background-color: #E7E7E7;
	}

	.container-row {
		display: flex;
		flex-flow: row;
		justify-content: space-between;
	}

	.container-row.top {
		flex: 1 1 auto;
	}

	.container-row.bottom {
		flex: 0 1 auto;
		justify-content: center;
	}

	.container-col {
		display: flex;
		flex-flow: column;
	}

	.control {
		flex: 1 1 auto;
		padding: 0 0.5rem;
		display: flex;
		align-items: center;
	}

	label {
		display: inline-block;
		width: 5rem;
		font-weight: bold;
	}

	select {
		margin-left: 0.5rem;
	}

	button {
		width: 10rem;
		line-height: 1rem;
		padding: 0.1rem;
	}

	input[type=number] {
		max-width: 3rem;
	}
</style>