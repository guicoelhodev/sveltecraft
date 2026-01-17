<script lang="ts">
	import { onMount } from 'svelte'
	import { items } from '$lib/data/items'

	interface Props {
		activeSlot?: number
	}

	let { activeSlot = $bindable(0) }: Props = $props()

	const onWheel = (e: WheelEvent) => {
		if (e.deltaY < 0) {
			// Scroll up - move left
			activeSlot = activeSlot <= 0 ? items.length - 1 : activeSlot - 1
		} else {
			// Scroll down - move right
			activeSlot = activeSlot >= items.length - 1 ? 0 : activeSlot + 1
		}
	}

	onMount(() => {
		window.addEventListener('wheel', onWheel)
		return () => {
			window.removeEventListener('wheel', onWheel)
		}
	})
</script>

<div class="pointer-events-none fixed bottom-6 left-1/2 -translate-x-1/2 flex gap-1">
	{#each items as item, i}
		<div
			class="h-12 w-12 border-2 bg-gray-900/80 rounded flex items-center justify-center {activeSlot === i ? 'border-white' : 'border-gray-700'}"
		>
			<img
				src={item.texture}
				alt={item.name}
				class="w-9 h-9 pixelated"
			/>
		</div>
	{/each}
</div>
