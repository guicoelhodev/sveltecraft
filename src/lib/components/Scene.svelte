<script lang="ts">
	import { T, useThrelte } from '@threlte/core'
	import Player from './Player.svelte'
	import Terrain from './Terrain.svelte'
	import Sun from './Sun.svelte'
	import Clouds from './Clouds.svelte'
	import BlockHighlight from './BlockHighlight.svelte'
	import { createNoise2D } from 'simplex-noise'
	import { Color } from 'three'

	interface Props {
		selectedBlock: string
		isLoaded?: boolean
	}

	let { selectedBlock, isLoaded = $bindable(false) }: Props = $props()

	const { scene } = useThrelte()
	scene.background = new Color('#84BEE3')

	const noise2D = createNoise2D()
	const maxHeight = 6
	const scale = 0.05
	const bedrockY = -2

	// Set de blocos removidos: "x,y,z"
	let removedBlocks = $state(new Set<string>())
	// Map de blocos adicionados: "x,y,z" -> blockType
	let addedBlocks = $state(new Map<string, string>())

	const getBlockKey = (x: number, y: number, z: number) => `${x},${y},${z}`

	const isBlockRemoved = (x: number, y: number, z: number): boolean => {
		return removedBlocks.has(getBlockKey(x, y, z))
	}

	const isBlockAdded = (x: number, y: number, z: number): boolean => {
		return addedBlocks.has(getBlockKey(x, y, z))
	}

	const getAddedBlockType = (x: number, y: number, z: number): string | undefined => {
		return addedBlocks.get(getBlockKey(x, y, z))
	}

	const removeBlock = (x: number, y: number, z: number) => {
		// Não permite remover bedrock
		if (y <= bedrockY) return
		const key = getBlockKey(x, y, z)
		// Se for um bloco adicionado, remove do map de adicionados
		if (addedBlocks.has(key)) {
			const newAdded = new Map(addedBlocks)
			newAdded.delete(key)
			addedBlocks = newAdded
		} else {
			// Se for um bloco do terreno, adiciona ao set de removidos
			removedBlocks = new Set([...removedBlocks, key])
		}
	}

	const placeBlock = (x: number, y: number, z: number, blockType: string) => {
		const key = getBlockKey(x, y, z)
		// Se o bloco foi removido antes, tira do set de removidos
		if (removedBlocks.has(key)) {
			const newRemoved = new Set(removedBlocks)
			newRemoved.delete(key)
			removedBlocks = newRemoved
		}
		// Adiciona o bloco
		addedBlocks = new Map([...addedBlocks, [key, blockType]])
	}

	const getHeight = (x: number, z: number): number => {
		const noiseValue = noise2D(x * scale, z * scale)
		return Math.floor(((noiseValue + 1) / 2) * maxHeight)
	}

	// Verifica se há um bloco sólido em uma posição específica
	const isSolidBlock = (x: number, y: number, z: number): boolean => {
		// Bloco adicionado
		if (isBlockAdded(x, y, z)) return true
		// Bloco do terreno (não removido)
		const terrainHeight = getHeight(x, z)
		if (y >= bedrockY && y <= terrainHeight && !isBlockRemoved(x, y, z)) return true
		return false
	}

	let playerX = $state(0)
	let playerZ = $state(0)
	let targetBlock: { x: number; y: number; z: number } | null = $state(null)
</script>

<T.HemisphereLight args={['#ffffff', '#444444', 0.6]} />
<T.AmbientLight intensity={0.3} />

<Player bind:positionX={playerX} bind:positionZ={playerZ} bind:targetBlock {getHeight} {isBlockRemoved} {removeBlock} {placeBlock} {selectedBlock} {isBlockAdded} {isSolidBlock} />
<Terrain playerX={playerX} playerZ={playerZ} {getHeight} {isBlockRemoved} {removedBlocks} {addedBlocks} {getAddedBlockType} bind:isLoaded />
<Sun playerX={playerX} playerZ={playerZ} />
<Clouds playerX={playerX} playerZ={playerZ} />
<BlockHighlight {targetBlock} />
