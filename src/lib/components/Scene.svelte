<script lang="ts">
	import { T } from '@threlte/core'
	import { OrbitControls } from '@threlte/extras'
</script>

<T.PerspectiveCamera makeDefault position={[10, 10, 10]} fov={75}>
	<OrbitControls />
</T.PerspectiveCamera>

<T.DirectionalLight position={[5, 10, 5]} intensity={1} />
<T.AmbientLight intensity={0.4} />

<T.Mesh position={[0, 0.5, 0]}>
	<T.BoxGeometry args={[1, 1, 1]} />
	<T.MeshStandardMaterial color="#8B4513" />
</T.Mesh>

<T.Mesh rotation.x={-Math.PI / 2} position={[0, 0, 0]}>
	<T.PlaneGeometry args={[20, 20]} />
	<T.MeshStandardMaterial color="#228B22" />
</T.Mesh>
