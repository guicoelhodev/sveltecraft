<script lang="ts">
	import { T, useTask } from '@threlte/core'
	import { onMount, onDestroy } from 'svelte'

	let position = $state({ x: 0, y: 0.5, z: 0 })
	let keys: Record<string, boolean> = {}

	const speed = 0.1

	const onKeyDown = (e: KeyboardEvent) => {
		keys[e.key.toLowerCase()] = true
	}

	const onKeyUp = (e: KeyboardEvent) => {
		keys[e.key.toLowerCase()] = false
	}

	useTask(() => {
		if (keys['w']) position.z -= speed
		if (keys['s']) position.z += speed
		if (keys['a']) position.x -= speed
		if (keys['d']) position.x += speed
	})

	onMount(() => {
		window.addEventListener('keydown', onKeyDown)
		window.addEventListener('keyup', onKeyUp)
	})

	onDestroy(() => {
		window.removeEventListener('keydown', onKeyDown)
		window.removeEventListener('keyup', onKeyUp)
	})
</script>

<T.Mesh position.x={position.x} position.y={position.y} position.z={position.z}>
	<T.BoxGeometry args={[1, 1, 1]} />
	<T.MeshStandardMaterial color="#8B4513" />
</T.Mesh>
