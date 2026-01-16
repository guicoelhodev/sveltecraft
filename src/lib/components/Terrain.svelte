<script lang="ts">
	import { T } from '@threlte/core'
	import { Collider } from '@threlte/rapier'
	import { createNoise2D } from 'simplex-noise'
	import { InstancedMesh, BoxGeometry, MeshStandardMaterial, Object3D, Color } from 'three'

	interface Props {
		playerX: number
		playerZ: number
	}

	let { playerX, playerZ }: Props = $props()

	const noise2D = createNoise2D()
	const chunkSize = 8
	const renderDistance = 1
	const maxHeight = 6
	const scale = 0.08

	const dummy = new Object3D()
	const geometry = new BoxGeometry(1, 1, 1)
	const material = new MeshStandardMaterial({ color: '#228B22' })

	const getHeight = (x: number, z: number): number => {
		const noiseValue = noise2D(x * scale, z * scale)
		return Math.floor(((noiseValue + 1) / 2) * maxHeight)
	}

	let instancedMesh: InstancedMesh | null = $state(null)

	const totalSize = chunkSize * (renderDistance * 2 + 1)
	const totalBlocks = totalSize * totalSize

	const updateTerrain = () => {
		if (!instancedMesh) return

		const currentChunkX = Math.floor(playerX / chunkSize)
		const currentChunkZ = Math.floor(playerZ / chunkSize)

		const startX = (currentChunkX - renderDistance) * chunkSize
		const startZ = (currentChunkZ - renderDistance) * chunkSize

		let index = 0
		const green = new Color('#228B22')
		const brown = new Color('#8B4513')
		const darkBrown = new Color('#654321')

		for (let z = 0; z < totalSize; z++) {
			for (let x = 0; x < totalSize; x++) {
				const worldX = startX + x
				const worldZ = startZ + z
				const height = getHeight(worldX, worldZ)

				dummy.position.set(worldX, height, worldZ)
				dummy.updateMatrix()
				instancedMesh.setMatrixAt(index, dummy.matrix)

				const color = height <= 1 ? brown : height <= 3 ? darkBrown : green
				instancedMesh.setColorAt(index, color)

				index++
			}
		}

		instancedMesh.instanceMatrix.needsUpdate = true
		if (instancedMesh.instanceColor) instancedMesh.instanceColor.needsUpdate = true
	}

	$effect(() => {
		playerX
		playerZ
		updateTerrain()
	})
</script>

<T.InstancedMesh
	bind:ref={instancedMesh}
	args={[geometry, material, totalBlocks]}
	frustumCulled={false}
	oncreate={updateTerrain}
/>

<Collider shape="cuboid" args={[500, 1, 500]} position={[0, -1, 0]} />
