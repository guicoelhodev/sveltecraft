<script lang="ts">
	import { T, useThrelte } from '@threlte/core'
	import Player from './Player.svelte'
	import Terrain from './Terrain.svelte'
	import { createNoise2D } from 'simplex-noise'
	import { Color } from 'three'

	const { scene } = useThrelte()
	scene.background = new Color('#87CEEB')

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

<T.HemisphereLight args={['#ffffff', '#444444', 0.6]} />
<T.DirectionalLight position={[50, 100, 25]} intensity={0.8} />
<T.AmbientLight intensity={0.3} />

<Player bind:positionX={playerX} bind:positionZ={playerZ} {getHeight} />
<Terrain playerX={playerX} playerZ={playerZ} {getHeight} />
