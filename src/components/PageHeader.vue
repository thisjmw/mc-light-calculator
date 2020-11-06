<template>
	<div id="header">
		<div class="title">Light Calculator</div>
		<div class="message">{{ message }}</div>
	</div>
</template>

<script>
	export default {
		name: 'PageHeader',
		props: {
			showControls: Boolean
		},
		computed: {
			message() {
				return `(${this.showControls ? 'hide' : 'show'} controls)`
			}
		}
	}
</script>

<style scoped>
	#header {
		background-color: #F7F7F7;
		height: 2rem;
		line-height: 2rem;
		padding: 0.5rem;
		display: flex;
		flex-direction: row;
		justify-content: space-between;
	}

	.title {
		font-weight: bold;
	}

	.message {
		font-size: 0.75rem;
		font-weight: bold;
		color: #777;
	}
</style>