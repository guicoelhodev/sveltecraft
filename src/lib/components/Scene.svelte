<script lang="ts">
	import { T } from '@threlte/core'
	import Player from './Player.svelte'
	import Terrain from './Terrain.svelte'
	import { createNoise2D } from 'simplex-noise'

	const noise2D = createNoise2D()
	const maxHeight = 6
	const scale = 0.08

	const getHeight = (x: number, z: number): number => {
		const noiseValue = noise2D(x * scale, z * scale)
		return Math.floor(((noiseValue + 1) / 2) * maxHeight)
	}

	let playerX = $state(0)
	let playerZ = $state(0)
</script>

<T.DirectionalLight position={[playerX, 50, playerZ]} intensity={2} />
<T.AmbientLight intensity={0.6} />

<Player bind:positionX={playerX} bind:positionZ={playerZ} {getHeight} />
<Terrain playerX={playerX} playerZ={playerZ} {getHeight} />
