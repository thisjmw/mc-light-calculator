<template>
	<div id="app">
		<div class="header no-select" @click="toggleControl">
			<page-header :show-controls="showControls" />
		</div>
		<div class="content">
			<light-calculator :show-controls="showControls" />
		</div>
	</div>
</template>

<script>
	import LightCalculator from './components/LightCalculator'
	import PageHeader from './components/PageHeader'

	export default {
		name: 'App',
		components: {
			PageHeader,
			LightCalculator
		},
		data() {
			return {
				showControls: true
			}
		},
		methods: {
			toggleControl() {
				this.showControls = !this.showControls
			}
		}
	}
</script>

<style>
	#app {
		font-family: Avenir,Helvetica, Arial, sans-serif;
		-webkit-font-smoothing: antialiased;
		-moz-osx-font-smoothing: grayscale;
		display: flex;
		flex-flow: column;
		height: 100%;
	}

	.no-select {
		-webkit-touch-callout: none;
		-webkit-user-select: none;
		-khtml-user-select: none;
		-moz-user-select: none;
		-ms-user-select: none;
		user-select: none;
	}

	/*
		App layout:
		https://stackoverflow.com/a/24979148
	*/

	.header {
		flex: 0 1 auto;
		cursor: pointer;
	}

	.content {
		flex: 1 1 auto;
		background-color: #5E5E5E;
	}

	/*.footer {*/
	/*	flex: 0 1 40px;*/
	/*}*/
</style>
