<script lang="ts">
	import { T, useThrelte } from '@threlte/core'
	import Player from './Player.svelte'
	import Terrain from './Terrain.svelte'
	import Sun from './Sun.svelte'
	import Clouds from './Clouds.svelte'
	import BlockHighlight from './BlockHighlight.svelte'
	import { createNoise2D } from 'simplex-noise'
	import { Color } from 'three'

	const { scene } = useThrelte()
	scene.background = new Color('#84BEE3')

	const noise2D = createNoise2D()
	const maxHeight = 6
	const scale = 0.08
	const bedrockY = -2

	// Set de blocos removidos: "x,y,z"
	let removedBlocks = $state(new Set<string>())

	const getBlockKey = (x: number, y: number, z: number) => `${x},${y},${z}`

	const isBlockRemoved = (x: number, y: number, z: number): boolean => {
		return removedBlocks.has(getBlockKey(x, y, z))
	}

	const removeBlock = (x: number, y: number, z: number) => {
		// Não permite remover bedrock
		if (y <= bedrockY) return
		removedBlocks = new Set([...removedBlocks, getBlockKey(x, y, z)])
	}

	const getHeight = (x: number, z: number): number => {
		const noiseValue = noise2D(x * scale, z * scale)
		return Math.floor(((noiseValue + 1) / 2) * maxHeight)
	}

	let playerX = $state(0)
	let playerZ = $state(0)
	let targetBlock: { x: number; y: number; z: number } | null = $state(null)
</script>

<T.HemisphereLight args={['#ffffff', '#444444', 0.6]} />
<T.AmbientLight intensity={0.3} />

<Player bind:positionX={playerX} bind:positionZ={playerZ} bind:targetBlock {getHeight} {isBlockRemoved} {removeBlock} />
<Terrain playerX={playerX} playerZ={playerZ} {getHeight} {isBlockRemoved} {removedBlocks} />
<Sun playerX={playerX} playerZ={playerZ} />
<Clouds playerX={playerX} playerZ={playerZ} />
<BlockHighlight {targetBlock} />
