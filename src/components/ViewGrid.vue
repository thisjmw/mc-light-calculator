<template>
	<div class="grid-container" :style="gridLayout" @mouseleave="mouseLeft">
		<div
			class="block no-select"
			:class="blockStyle(block)"
			v-for="(block, index) in grid"
			:key="index"
			@mousedown="mouseDown(index)"
			@mouseenter="dragging ? mouseEnter(index) : null"
			@mouseup="mouseUp"
		>
			<template v-if="gridView === gridViewEnum.LIGHT">
				{{ block.type !== blockTypeEnum.WALL ? block.strength : '' }}
			</template>
			<template v-else-if="gridView === gridViewEnum.ELEVATION">
				{{ block.elevation }}
			</template>
		</div>
	</div>
</template>

<script>
	import {
		blockTypeEnum,
		gridViewEnum,
		controlTypeEnum,
		dragBehaviorEnum
	} from '../data'

	export default {
		name: 'LightGrid',
		emits: [
			'add-light',
			'remove-light',
			'add-wall',
			'remove-wall',
			'change-elevation'
		],
		props: {
			width: {
				type: Number,
				validator: n => n > 0
			},
			height: {
				type: Number,
				validator: n => n > 0
			},
			lights: Array,
			walls: Array,
			elevations: Object,
			control: Number, // controlTypeEnum
			gridView: Number // gridViewEnum
		},
		data() {
			return {
				processed: 0,
				updated: 0,
				duplicated: 0,
				blockTypeEnum,
				gridViewEnum,
				dragging: false,
				dragHistory: [],
				dragBehavior: dragBehaviorEnum.NONE
			}
		},
		computed: {
			grid() {
				const blocks = []
				for (let y = 0; y < this.height; y++) {
					blocks[y] = []
					for (let x = 0; x < this.width; x++) {
						const elevation = this.elevations[`${x},${y}`] || 0
						blocks[y][x] = { strength: 0, type: blockTypeEnum.NORMAL, elevation }
					}
				}
				return this.gridView === gridViewEnum.LIGHT ? this.calculateLighting(blocks) : blocks.flat()
			},
			gridLayout() {
				return {
					gridTemplateColumns: `repeat(${this.width}, 1.2rem)`
				}
			},
		},
		methods: {
			// Returns object with [key] -> [value] pairs of [entity coordinate index] -> [entity] for specified entity
			// list, e.g. `lights` or `walls`
			getEntityIndexMap(entityListPropName) {
				const indexMap = {}
				for (const entity of this[entityListPropName]) {
					indexMap[this.coordinatesToIndex(entity.x, entity.y)] = entity
				}
				return indexMap
			},
			calculateLighting(blocks) {
				// TODO: Find ways to make this more efficient
				this.processed = 0
				this.updated = 0
				this.duplicated = 0

				if (!this.lights.length && !this.walls.length) {
					return blocks.flat()
				}

				const queue = []

				// Add walls first so they interfere with lighting
				for (const wall of this.walls) {
					queue.push({ ...wall, type: blockTypeEnum.WALL })
				}

				for (const light of this.lights) {
					queue.push({ ...light, type: blockTypeEnum.LIGHT })
				}

				let currentBlock = queue.shift()

				while (currentBlock) {
					this.processed++
					const {x, y, type} = currentBlock

					// Extract elevation from the underlying grid - entities such as lights will adopt this elevation
					currentBlock.elevation = blocks[y][x].elevation

					if (type === blockTypeEnum.WALL) {
						this.updated++
						blocks[y][x] = { ...blocks[y][x], type }
					} else if (blocks[y][x].strength < currentBlock.strength) {
						this.updated++
						const strength = currentBlock.strength
						blocks[y][x] = { ...blocks[y][x], type, strength }
						if (strength > 1) {
							// Update neighbors starting at top and going clockwise
							updateNeighbor(blocks, queue, x, y - 1, y > 0, currentBlock)
							updateNeighbor(blocks, queue, x + 1, y, x < this.width - 1, currentBlock)
							updateNeighbor(blocks, queue, x, y + 1, y < this.height - 1, currentBlock)
							updateNeighbor(blocks, queue, x - 1, y, x > 0, currentBlock)
						}
					} else {
						this.duplicated++
					}

					currentBlock = queue.shift()
				}

				return blocks.flat()

				function updateNeighbor(blocks, queue, x, y, condition, currentBlock) {
					// Only try to update block[y][x] if it's not already brighter than `currentBlock` could make it
					if (condition && blocks[y][x].strength < currentBlock.strength - 1) {
						const nextElevation = blocks[y][x].elevation
						const dElevation = Math.abs(nextElevation - currentBlock.elevation)
						const nextStrength = currentBlock.strength - 1 - dElevation
						if (nextStrength > 0 && nextStrength > blocks[y][x].strength) {
							queue.push({
								x,
								y,
								elevation: nextElevation,
								strength: nextStrength,
								type: blocks[y][x].type
							})
						}
					}
				}
			},

			clicked(index) {
				let coordinates = this.indexToCoordinates(index)
				switch (this.gridView) {
					case gridViewEnum.LIGHT:
						// TODO: Still ugly :(
						const lightViewEventHelpers = {
							[controlTypeEnum.LIGHT]: {
								targetObjectExistsAtCoordinates: this.isBlockType(blockTypeEnum.LIGHT, index),
								replaceableObjectExistsAtCoordinates: this.isBlockType(blockTypeEnum.WALL, index),
								targetObjectName: 'light',
								replaceableObjectName: 'wall'
							},
							[controlTypeEnum.WALL]: {
								targetObjectExistsAtCoordinates: this.isBlockType(blockTypeEnum.WALL, index),
								replaceableObjectExistsAtCoordinates: this.isBlockType(blockTypeEnum.LIGHT, index),
								targetObjectName: 'wall',
								replaceableObjectName: 'light'
							}
						}
						const helper = lightViewEventHelpers[this.control]
						if (helper.targetObjectExistsAtCoordinates) {
							this.$emit(`remove-${helper.targetObjectName}`, coordinates)
						} else {
							if (helper.replaceableObjectExistsAtCoordinates) {
								this.$emit(`remove-${helper.replaceableObjectName}`, coordinates)
							}
							this.$emit(`add-${helper.targetObjectName}`, coordinates)
						}
						break

					case gridViewEnum.ELEVATION:
						this.$emit('change-elevation', coordinates)
						break
				}
			},
			blockStyle(block) {
				switch (this.gridView) {
					case gridViewEnum.LIGHT:
						const lightClass = `light-${block.strength}`
						return {
							light: block.type === blockTypeEnum.LIGHT,
							[lightClass]: block.type !== blockTypeEnum.WALL,
							wall: block.type === blockTypeEnum.WALL,
							danger: block.type !== blockTypeEnum.WALL && block.strength < 8
						}
					case gridViewEnum.ELEVATION:
						const elevationClass = `elevation-${block.elevation > 3 ? 3 : block.elevation}`
						return {
							elevation: true,
							[elevationClass]: true
						}
				}
			},
			indexToCoordinates(index) {
				const y = Math.floor(index / this.width)
				const x = index - (y * this.width)
				const elevation = this.grid[index].elevation
				return { x, y, elevation }
			},
			coordinatesToIndex(x, y) {
				return x + (y * this.width)
			},
			isBlockType(blockType, index) {
				const propNames = {
					[blockTypeEnum.LIGHT]: 'lights',
					[blockTypeEnum.WALL]: 'walls'
				}
				const propName = propNames[blockType]
				if (propName) {
					return !!this.getEntityIndexMap(propName)[index]
				} else {
					return false
				}
			},
			mouseDown(index) {
				this.dragging = true

				switch (this.gridView) {
					case gridViewEnum.LIGHT:
						switch (this.control) {
							case controlTypeEnum.LIGHT:
								if (this.isBlockType(blockTypeEnum.LIGHT, index)) {
									this.dragBehavior = dragBehaviorEnum.REMOVE_LIGHT
								} else {
									this.dragBehavior = dragBehaviorEnum.ADD_LIGHT
								}
								break
							case controlTypeEnum.WALL:
								if (this.isBlockType(blockTypeEnum.WALL, index)) {
									this.dragBehavior = dragBehaviorEnum.REMOVE_WALL
								} else {
									this.dragBehavior = dragBehaviorEnum.ADD_WALL
								}
								break
						}
						break
					case gridViewEnum.ELEVATION:
						this.dragBehavior = dragBehaviorEnum.INCREASE_ELEVATION
						break
				}

				if (this.dragBehavior !== dragBehaviorEnum.INCREASE_ELEVATION) {
					this.dragHistory.push(index)
				}
				this.clicked(index)
			},
			mouseUp() {
				this.dragging = false
				this.dragHistory = []
				this.dragBehavior = dragBehaviorEnum.NONE
			},
			mouseEnter(index) {
				if (this.dragBehavior === dragBehaviorEnum.INCREASE_ELEVATION) {
					this.clicked(index)
				} else {
					if (!~this.dragHistory.indexOf(index)) {
						this.dragHistory.push(index)
						switch (this.dragBehavior) {
							case dragBehaviorEnum.ADD_LIGHT:
								if (!this.isBlockType(blockTypeEnum.LIGHT, index)) {
									this.clicked(index)
								}
								break
							case dragBehaviorEnum.REMOVE_LIGHT:
								if (this.isBlockType(blockTypeEnum.LIGHT, index)) {
									this.clicked(index)
								}
								break
							case dragBehaviorEnum.ADD_WALL:
								if (!this.isBlockType(blockTypeEnum.WALL, index)) {
									this.clicked(index)
								}
								break
							case dragBehaviorEnum.REMOVE_WALL:
								if (this.isBlockType(blockTypeEnum.WALL, index)) {
									this.clicked(index)
								}
								break
						}
					}
				}
			},
			mouseLeft() {
				if (this.dragging) {
					this.mouseUp()
				}
			}
		}
	}
</script>

<style scoped>
	.grid-container {
		display: grid;
	}

	.block {
		height: 1.2rem;
		cursor: pointer;
		text-align: center;
		font-size: 0.8rem;
		line-height: 1.2rem;
		vertical-align: middle;
	}

	.wall {
		background-color: #555555;
	}

	.block:hover {
		background-color: white;
	}

	/* TODO: Remove unused selector code warnings */

	.light {
		background-color: white !important;
	}

	.light-15 { background-color: #fddf6c; }
	.light-14 { background-color: #efca5b; }
	.light-13 { background-color: #e1b54b; }
	.light-12 { background-color: #d3a13c; }
	.light-11 { background-color: #c48d2d; }
	.light-10 { background-color: #b5791f; }
	.light-9  { background-color: #a66610; }
	.light-8  { background-color: #965300; }
	.light-7  { background-color: #88450d; }
	.light-6  { background-color: #783914; }
	.light-5  { background-color: #662e17; }
	.light-4  { background-color: #542518; }
	.light-3  { background-color: #421d16; }
	.light-2  { background-color: #2f1613; }
	.light-1  { background-color: #1d0e0c; }
	.light-0  { background-color: #000000; }

	.elevation-0  { background-color: #000000; }
	.elevation-1  { background-color: #222222; }
	.elevation-2  { background-color: #444444; }
	.elevation-3  { background-color: #666666; }

	.danger, .elevation {
		color: white;
	}
</style>