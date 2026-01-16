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
	const renderDistance = 2

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
	// Blocos extras para os expostos por remoção
	const totalBlocks = totalSize * totalSize + 500

	const getBlockMaterial = (y: number): 'grass' | 'dirt' | 'stone' => {
		if (y <= 1) return 'stone'
		if (y <= 3) return 'dirt'
		return 'grass'
	}

	// Retorna blocos expostos por causa de remoções adjacentes
	const getExposedBlocks = () => {
		const exposed: Array<{ x: number; y: number; z: number }> = []

		for (const key of removedBlocks) {
			const [rx, ry, rz] = key.split(',').map(Number)

			// Bloco abaixo
			if (ry > 0 && !isBlockRemoved(rx, ry - 1, rz)) {
				exposed.push({ x: rx, y: ry - 1, z: rz })
			}

			// Blocos laterais (apenas se estiverem na mesma altura ou abaixo do terreno)
			const neighbors = [
				{ x: rx + 1, z: rz },
				{ x: rx - 1, z: rz },
				{ x: rx, z: rz + 1 },
				{ x: rx, z: rz - 1 }
			]

			for (const n of neighbors) {
				const neighborHeight = getHeight(n.x, n.z)
				// Se o vizinho tem bloco nessa altura e não foi removido
				if (ry <= neighborHeight && !isBlockRemoved(n.x, ry, n.z)) {
					exposed.push({ x: n.x, y: ry, z: n.z })
				}
			}
		}

		return exposed
	}

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

		// Renderiza camada de cima
		for (let z = 0; z < totalSize; z++) {
			for (let x = 0; x < totalSize; x++) {
				const worldX = startX + x
				const worldZ = startZ + z
				const height = getHeight(worldX, worldZ)

				if (isBlockRemoved(worldX, height, worldZ)) continue

				dummy.position.set(worldX, height, worldZ)
				dummy.updateMatrix()

				const material = getBlockMaterial(height)
				if (material === 'stone') {
					stoneMesh.setMatrixAt(stoneIndex++, dummy.matrix)
				} else if (material === 'dirt') {
					dirtMesh.setMatrixAt(dirtIndex++, dummy.matrix)
				} else {
					grassMesh.setMatrixAt(grassIndex++, dummy.matrix)
				}
			}
		}

		// Renderiza blocos expostos por remoção
		const exposedBlocks = getExposedBlocks()
		const rendered = new Set<string>()

		for (const block of exposedBlocks) {
			const key = `${block.x},${block.y},${block.z}`
			if (rendered.has(key)) continue
			rendered.add(key)

			// Verifica se está na área visível
			if (
				block.x < startX ||
				block.x >= startX + totalSize ||
				block.z < startZ ||
				block.z >= startZ + totalSize
			) continue

			dummy.position.set(block.x, block.y, block.z)
			dummy.updateMatrix()

			const material = getBlockMaterial(block.y)
			if (material === 'stone') {
				stoneMesh.setMatrixAt(stoneIndex++, dummy.matrix)
			} else if (material === 'dirt') {
				dirtMesh.setMatrixAt(dirtIndex++, dummy.matrix)
			} else {
				grassMesh.setMatrixAt(grassIndex++, dummy.matrix)
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
