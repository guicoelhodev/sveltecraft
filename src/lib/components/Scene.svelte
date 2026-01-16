<script lang="ts">
	import { T } from '@threlte/core'
	import { OrbitControls } from '@threlte/extras'
	import Player from './Player.svelte'
</script>

<T.PerspectiveCamera makeDefault position={[10, 10, 10]} fov={75}>
	<OrbitControls />
</T.PerspectiveCamera>

<T.DirectionalLight position={[5, 10, 5]} intensity={1} />
<T.AmbientLight intensity={0.4} />

<Player />

<T.Mesh rotation.x={-Math.PI / 2} position={[0, 0, 0]}>
	<T.PlaneGeometry args={[20, 20]} />
	<T.MeshStandardMaterial color="#228B22" />
</T.Mesh>
