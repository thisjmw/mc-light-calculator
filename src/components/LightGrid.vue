<template>
	<div class="grid-container" :style="gridLayout">
		<div
			class="block"
			:class="lightStyle(block)"
			v-for="(block, index) in grid"
			:key="index"
			@click="clicked(index)"
		>
			{{ block }}
		</div>
	</div>
</template>

<script>
	export default {
		name: 'LightGrid',
		emits: [
			'add-light',
			'remove-light'
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
			lights: Array
		},
		data() {
			return {
				processed: 0,
				updated: 0,
				duplicated: 0
			}
		},
		computed: {
			grid() {
				let blocks = []
				for (let y = 0; y < this.height; y++) {
					blocks[y] = []
					for (let x = 0; x < this.width; x++) {
						blocks[y][x] = 0
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
			}
		},
		methods: {
			calculateLighting(blocks) {
				// TODO: Find ways to make this more efficient
				let retBlocks = [...blocks]
				this.processed = 0
				this.updated = 0
				this.duplicated = 0

				if (!this.lights.length) {
					return blocks.flat()
				}

				const queue = []

				for (const light of this.lights) {
					queue.push({ x: light.x, y: light.y, strength: light.strength })
				}

				let currentBlock = queue.shift()

				while (currentBlock) {
					this.processed++
					const {x, y, strength} = currentBlock
					const blockLight = retBlocks[y][x]

					if (blockLight < strength) {
						this.updated++
						retBlocks[y][x] = strength
						if (strength > 1) {
							const nextStrength = strength - 1
							if (y > 0 && retBlocks[y - 1][x] < nextStrength) {
								queue.push({ x, y: y - 1, strength: nextStrength })
							}
							if (x < this.width - 1 && retBlocks[y][x + 1] < nextStrength) {
								queue.push({ x: x + 1, y, strength: nextStrength })
							}
							if (y < this.height - 1 && retBlocks[y + 1][x] < nextStrength) {
								queue.push({ x, y: y + 1, strength: nextStrength })
							}
							if (x > 0 && retBlocks[y][x - 1] < nextStrength) {
								queue.push({ x: x - 1, y, strength: nextStrength })
							}
						}
					} else {
						this.duplicated++
					}

					currentBlock = queue.shift()
				}

				return retBlocks.flat()
			},
			clicked(index) {
				if (this.isLight(index)) {
					this.$emit('remove-light', this.indexToCoordinates(index))
				} else {
					this.$emit('add-light', this.indexToCoordinates(index))
				}
			},
			lightStyle(lightLevel) {
				const lightClass = `light-${lightLevel}`
				return {
					[lightClass]: true,
					danger: lightLevel < 8
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

	.block:hover {
		background-color: white;
	}

	/* TODO: Remove unused selector code warnings */

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