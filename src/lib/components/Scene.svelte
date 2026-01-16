<script lang="ts">
	import { T } from '@threlte/core'
	import Player from './Player.svelte'
	import Terrain from './Terrain.svelte'

	let playerX = $state(0)
	let playerZ = $state(0)
</script>

<T.DirectionalLight position={[5, 10, 5]} intensity={1} />
<T.AmbientLight intensity={0.4} />

<Player bind:positionX={playerX} bind:positionZ={playerZ} />
<Terrain playerX={playerX} playerZ={playerZ} />
