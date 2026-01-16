<script lang="ts">
	import { T } from '@threlte/core'
	import Player from './Player.svelte'
	import Terrain from './Terrain.svelte'

	let playerX = $state(0)
	let playerZ = $state(0)
</script>

<T.DirectionalLight position={[playerX, 50, playerZ]} intensity={2} />
<T.AmbientLight intensity={0.6} />

<Player bind:positionX={playerX} bind:positionZ={playerZ} />
<Terrain playerX={playerX} playerZ={playerZ} />
