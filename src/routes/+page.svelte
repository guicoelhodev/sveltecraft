<script lang="ts">
	import { Canvas } from '@threlte/core'
	import Scene from '$lib/components/Scene.svelte'
	import { onMount, onDestroy } from 'svelte'

	let activeSlot = $state(0)

	const onWheel = (e: WheelEvent) => {
		if (e.deltaY < 0) {
			// Scroll up - move left
			activeSlot = activeSlot <= 0 ? 7 : activeSlot - 1
		} else {
			// Scroll down - move right
			activeSlot = activeSlot >= 7 ? 0 : activeSlot + 1
		}
	}

	onMount(() => {
		window.addEventListener('wheel', onWheel)
	})

	onDestroy(() => {
		window.removeEventListener('wheel', onWheel)
	})
</script>

<div class="h-screen w-screen">
	<Canvas>
		<Scene />
	</Canvas>
	<!-- Crosshair -->
	<div class="pointer-events-none fixed inset-0 flex items-center justify-center">
		<div class="h-6 w-0.5 bg-white opacity-70"></div>
		<div class="absolute h-0.5 w-6 bg-white opacity-70"></div>
	</div>
	<!-- Hotbar HUD -->
	<div class="pointer-events-none fixed bottom-6 left-1/2 -translate-x-1/2 flex gap-1">
		{#each Array(8) as _, i}
			<div
				class="h-12 w-12 border-2 bg-gray-900/80 rounded flex items-center justify-center {activeSlot === i ? 'border-white' : 'border-gray-700'}"
			>
				<img
					src="/textures/dirt-texture.png"
					alt="dirt"
					class="w-9 h-9 pixelated"
				/>
			</div>
		{/each}
	</div>
</div>
