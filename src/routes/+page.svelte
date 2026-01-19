<script lang="ts">
	import { Canvas } from '@threlte/core'
	import Scene from '$lib/components/Scene.svelte'
	import Hotbar from '$lib/components/Hotbar.svelte'
	import StartScreen from '$lib/components/StartScreen.svelte'
	import { items } from '$lib/data/items'

	let activeSlot = $state(0)
	let selectedBlock = $derived(items[activeSlot].name)
	let gameStarted = $state(false)
	let isLoaded = $state(false)

	const handlePlay = () => {
		gameStarted = true
		const canvas = document.querySelector('canvas')
		canvas?.requestPointerLock()
	}
</script>

{#if !gameStarted}
	<StartScreen {isLoaded} onPlay={handlePlay} />
{/if}

<div class="h-screen w-screen">
	<Canvas>
		<Scene {selectedBlock} bind:isLoaded />
	</Canvas>
	{#if gameStarted}
		<div class="pointer-events-none fixed inset-0 flex items-center justify-center">
			<div class="h-6 w-0.5 bg-white opacity-70"></div>
			<div class="absolute h-0.5 w-6 bg-white opacity-70"></div>
		</div>
		<Hotbar bind:activeSlot />
	{/if}
</div>
