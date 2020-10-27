<template>
	<div class="grid-container" :style="gridLayout">
		<div
			class="block"
			:class="lightStyle(block)"
			v-for="(block, index) in grid"
			:key="index"
			@click="clicked(index)"
		>
			{{ block.type !== blockTypeEnum.WALL ? block.strength : '' }}
		</div>
	</div>
</template>

<script>
	import { blockTypeEnum } from '../data'

	export default {
		name: 'LightGrid',
		emits: [
			'add-light',
			'remove-light',
			'add-wall',
			'remove-wall'
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
			control: Number
		},
		data() {
			return {
				processed: 0,
				updated: 0,
				duplicated: 0,
				blockTypeEnum
			}
		},
		computed: {
			grid() {
				const blocks = []
				for (let y = 0; y < this.height; y++) {
					blocks[y] = []
					for (let x = 0; x < this.width; x++) {
						blocks[y][x] = { strength: 0, type: blockTypeEnum.NORMAL }
					}
				}
				return this.calculateLighting(blocks)
			},
			gridLayout() {
				return {
					gridTemplateColumns: `repeat(${this.width}, 1.2rem)`
				}
			},
			lightCoordinates() {
				const lightCoordinatesMap = {}
				for (const light of this.lights) {
					lightCoordinatesMap[this.coordinatesToIndex(light.x, light.y)] = light
				}
				return lightCoordinatesMap
			},
			wallCoordinates() {
				const wallCoordinatesMap = {}
				for (const wall of this.walls) {
					wallCoordinatesMap[this.coordinatesToIndex(wall.x, wall.y)] = wall
				}
				return wallCoordinatesMap
			}
		},
		methods: {
			calculateLighting(blocks) {
				// TODO: Find ways to make this more efficient
				this.processed = 0
				this.updated = 0
				this.duplicated = 0

				if (!this.lights.length && !this.walls.length) {
					return blocks.flat()
				}

				const queue = []

				for (const wall of this.walls) {
					queue.push({ type: blockTypeEnum.WALL, x: wall.x, y: wall.y })
				}

				for (const light of this.lights) {
					queue.push({ type: blockTypeEnum.LIGHT, x: light.x, y: light.y, strength: light.strength })
				}

				let currentBlock = queue.shift()

				while (currentBlock) {
					this.processed++
					const {x, y, type} = currentBlock

					if (type === blockTypeEnum.WALL) {
						this.updated++
						blocks[y][x] = { type }
					} else if (blocks[y][x].strength < currentBlock.strength) {
						this.updated++
						const strength = currentBlock.strength
						blocks[y][x] = { strength, type }
						if (strength > 1) {
							const nextStrength = strength - 1
							if (y > 0 && blocks[y - 1][x].strength < nextStrength) {
								queue.push({ x, y: y - 1, strength: nextStrength, type: blocks[y - 1][x].type })
							}
							if (x < this.width - 1 && blocks[y][x + 1].strength < nextStrength) {
								queue.push({ x: x + 1, y, strength: nextStrength, type: blocks[y][x + 1].type })
							}
							if (y < this.height - 1 && blocks[y + 1][x].strength < nextStrength) {
								queue.push({ x, y: y + 1, strength: nextStrength, type: blocks[y + 1][x].type })
							}
							if (x > 0 && blocks[y][x - 1].strength < nextStrength) {
								queue.push({ x: x - 1, y, strength: nextStrength, type: blocks[y][x - 1].type })
							}
						}
					} else {
						this.duplicated++
					}

					currentBlock = queue.shift()
				}

				return blocks.flat()
			},
			clicked(index) {
				const coordinates = this.indexToCoordinates(index)
				switch (this.control) {
					case blockTypeEnum.LIGHT:
						if (this.isLight(index)) {
							this.$emit('remove-light', coordinates)
						} else {
							this.$emit('add-light', coordinates)
							if (this.isWall(index)) {
								this.$emit('remove-wall', coordinates)
							}
						}
						break
					case blockTypeEnum.WALL:
						if (this.isWall(index)) {
							this.$emit('remove-wall', coordinates)
						} else {
							this.$emit('add-wall', coordinates)
							if (this.isLight(index)) {
								this.$emit('remove-light', coordinates)
							}
						}
						break
				}
			},
			lightStyle(block) {
				const lightClass = `light-${block.strength}`
				return {
					light: block.type === blockTypeEnum.LIGHT,
					[lightClass]: block.type !== blockTypeEnum.WALL,
					wall: block.type === blockTypeEnum.WALL,
					danger: block.type !== blockTypeEnum.WALL && block.strength < 8
				}
			},
			indexToCoordinates(index) {
				const y = Math.floor(index / this.width)
				const x = index - (y * this.width)
				return { x, y }
			},
			coordinatesToIndex(x, y) {
				return x + (y * this.width)
			},
			isLight(index) {
				return !!this.lightCoordinates[index]
			},
			isWall(index) {
				return !!this.wallCoordinates[index]
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

	.danger {
		color: white;
	}
</style>