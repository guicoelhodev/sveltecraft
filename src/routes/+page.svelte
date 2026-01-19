<script lang="ts">
	import { onMount, onDestroy } from 'svelte'
	import { browser } from '$app/environment'
	import { Canvas } from '@threlte/core'
	import Scene from '$lib/components/Scene.svelte'
	import Hotbar from '$lib/components/Hotbar.svelte'
	import StartScreen from '$lib/components/StartScreen.svelte'
	import { items } from '$lib/data/items'

	let activeSlot = $state(0)
	let selectedBlock = $derived(items[activeSlot].name)
	let gameStarted = $state(false)
	let isLoaded = $state(false)
	let isPaused = $state(false)
	let sensitivity = $state(0.002)

	const handlePlay = () => {
		gameStarted = true
		isPaused = false
		const canvas = document.querySelector('canvas')
		canvas?.requestPointerLock()
	}

	const handleSensitivityChange = (value: number) => {
		sensitivity = value
	}

	const handlePointerLockChange = () => {
		if (gameStarted && !document.pointerLockElement) {
			isPaused = true
		}
	}

	onMount(() => {
		document.addEventListener('pointerlockchange', handlePointerLockChange)
	})

	onDestroy(() => {
		if (browser) {
			document.removeEventListener('pointerlockchange', handlePointerLockChange)
		}
	})
</script>

{#if !gameStarted || isPaused}
	<StartScreen {isLoaded} onPlay={handlePlay} {sensitivity} onSensitivityChange={handleSensitivityChange} {isPaused} />
{/if}

<div class="h-screen w-screen">
	<Canvas>
		<Scene {selectedBlock} bind:isLoaded {sensitivity} />
	</Canvas>
	{#if gameStarted && !isPaused}
		<div class="pointer-events-none fixed inset-0 flex items-center justify-center">
			<div class="h-6 w-0.5 bg-white opacity-70"></div>
			<div class="absolute h-0.5 w-6 bg-white opacity-70"></div>
		</div>
		<Hotbar bind:activeSlot />
	{/if}
</div>
