<script lang="ts">
	import { T } from '@threlte/core'
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

	const updateTerrain = () => {
		if (!instancedMesh) return

		const currentChunkX = Math.floor(playerX / chunkSize)
		const currentChunkZ = Math.floor(playerZ / chunkSize)

		let index = 0
		const green = new Color('#228B22')
		const brown = new Color('#8B4513')
		const darkBrown = new Color('#654321')

		for (let dx = -renderDistance; dx <= renderDistance; dx++) {
			for (let dz = -renderDistance; dz <= renderDistance; dz++) {
				const chunkX = currentChunkX + dx
				const chunkZ = currentChunkZ + dz

				for (let x = 0; x < chunkSize; x++) {
					for (let z = 0; z < chunkSize; z++) {
						const worldX = chunkX * chunkSize + x
						const worldZ = chunkZ * chunkSize + z
						const height = getHeight(worldX, worldZ)

						dummy.position.set(worldX, height, worldZ)
						dummy.updateMatrix()
						instancedMesh.setMatrixAt(index, dummy.matrix)

						const color = height <= 1 ? brown : height <= 3 ? darkBrown : green
						instancedMesh.setColorAt(index, color)

						index++
					}
				}
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

	const totalBlocks = Math.pow(chunkSize * (renderDistance * 2 + 1), 2)
</script>

<T.InstancedMesh
	bind:ref={instancedMesh}
	args={[geometry, material, totalBlocks]}
	frustumCulled={false}
	oncreate={updateTerrain}
/>
