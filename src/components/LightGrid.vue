<template>
	<div>
		<table>
			<tbody>
				<tr v-for="(row, y) in grid" :key="'row' + y">
					<td v-for="(block, x) in row" :key="x + ',' + y" @click="addLight(x, y)">
						{{ block }}
					</td>
				</tr>
			</tbody>
		</table>
	</div>
</template>

<script>
	export default {
		name: 'LightGrid',
		emits: ['add-light'],
		props: {
			width: {
				type: Number,
				validator: n => n > 0
			},
			height: {
				type: Number,
				validator: n => n > 0
			},
			lights: {
				type: Array
			}
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

				return this.calculateLightning(blocks)
			}
		},
		methods: {
			calculateLightning(blocks) {
				let retBlocks = [...blocks]
				this.processed = 0
				this.updated = 0
				this.duplicated = 0

				if (!this.lights.length) {
					return blocks
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

				return retBlocks
			},
			addLight(x, y) {
				this.$emit('add-light', { x, y })
			}
		}
	}
</script>

<style scoped>
td {
	cursor: pointer
}
</style>