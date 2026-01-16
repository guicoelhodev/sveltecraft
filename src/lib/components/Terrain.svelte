<script lang="ts">
	import { T } from '@threlte/core'
	import {
		InstancedMesh,
		BoxGeometry,
		MeshStandardMaterial,
		Object3D,
		TextureLoader,
		NearestFilter,
		RepeatWrapping
	} from 'three'

	interface Props {
		playerX: number
		playerZ: number
		getHeight: (x: number, z: number) => number
		isBlockRemoved: (x: number, y: number, z: number) => boolean
		removedBlocks: Set<string>
	}

	let { playerX, playerZ, getHeight, isBlockRemoved, removedBlocks }: Props = $props()

	const chunkSize = 16
	const renderDistance = 3

	const dummy = new Object3D()
	const geometry = new BoxGeometry(1, 1, 1)

	const textureLoader = new TextureLoader()

	const loadTexture = (path: string) => {
		const texture = textureLoader.load(path)
		texture.magFilter = NearestFilter
		texture.minFilter = NearestFilter
		texture.wrapS = RepeatWrapping
		texture.wrapT = RepeatWrapping
		return texture
	}

	const grassTexture = loadTexture('/textures/grass-texture.jpg')
	const dirtTexture = loadTexture('/textures/dirt-texture.png')
	const stoneTexture = loadTexture('/textures/stone-texture.png')

	const grassTopMaterial = new MeshStandardMaterial({ map: grassTexture })
	const dirtMaterial = new MeshStandardMaterial({ map: dirtTexture })
	const stoneMaterial = new MeshStandardMaterial({ map: stoneTexture })

	const grassMaterials = [
		dirtMaterial,
		dirtMaterial,
		grassTopMaterial,
		dirtMaterial,
		dirtMaterial,
		dirtMaterial
	]

	let grassMesh: InstancedMesh | null = $state(null)
	let dirtMesh: InstancedMesh | null = $state(null)
	let stoneMesh: InstancedMesh | null = $state(null)

	const totalSize = chunkSize * (renderDistance * 2 + 1)
	const totalBlocks = totalSize * totalSize

	const updateTerrain = () => {
		if (!grassMesh || !dirtMesh || !stoneMesh) return

		const currentChunkX = Math.floor(playerX / chunkSize)
		const currentChunkZ = Math.floor(playerZ / chunkSize)

		const startX = (currentChunkX - renderDistance) * chunkSize
		const startZ = (currentChunkZ - renderDistance) * chunkSize

		let grassIndex = 0
		let dirtIndex = 0
		let stoneIndex = 0

		const hiddenPosition = { x: 0, y: -1000, z: 0 }

		for (let i = 0; i < totalBlocks; i++) {
			dummy.position.set(hiddenPosition.x, hiddenPosition.y, hiddenPosition.z)
			dummy.updateMatrix()
			grassMesh.setMatrixAt(i, dummy.matrix)
			dirtMesh.setMatrixAt(i, dummy.matrix)
			stoneMesh.setMatrixAt(i, dummy.matrix)
		}

		for (let z = 0; z < totalSize; z++) {
			for (let x = 0; x < totalSize; x++) {
				const worldX = startX + x
				const worldZ = startZ + z
				const height = getHeight(worldX, worldZ)

				// Pula blocos removidos
				if (isBlockRemoved(worldX, height, worldZ)) continue

				dummy.position.set(worldX, height, worldZ)
				dummy.updateMatrix()

				if (height <= 1) {
					stoneMesh.setMatrixAt(stoneIndex++, dummy.matrix)
				} else if (height <= 3) {
					dirtMesh.setMatrixAt(dirtIndex++, dummy.matrix)
				} else {
					grassMesh.setMatrixAt(grassIndex++, dummy.matrix)
				}
			}
		}

		grassMesh.instanceMatrix.needsUpdate = true
		dirtMesh.instanceMatrix.needsUpdate = true
		stoneMesh.instanceMatrix.needsUpdate = true
	}

	$effect(() => {
		playerX
		playerZ
		removedBlocks
		updateTerrain()
	})
</script>

<T.InstancedMesh
	bind:ref={grassMesh}
	args={[geometry, grassMaterials, totalBlocks]}
	frustumCulled={false}
	oncreate={updateTerrain}
/>

<T.InstancedMesh
	bind:ref={dirtMesh}
	args={[geometry, dirtMaterial, totalBlocks]}
	frustumCulled={false}
/>

<T.InstancedMesh
	bind:ref={stoneMesh}
	args={[geometry, stoneMaterial, totalBlocks]}
	frustumCulled={false}
/>
