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
		addedBlocks: Map<string, string>
		getAddedBlockType: (x: number, y: number, z: number) => string | undefined
		isLoaded?: boolean
	}

	let { playerX, playerZ, getHeight, isBlockRemoved, removedBlocks, addedBlocks, getAddedBlockType, isLoaded = $bindable(false) }: Props = $props()

	const chunkSize = 16
	const renderDistance = 2

	const dummy = new Object3D()
	const geometry = new BoxGeometry(1, 1, 1)

	const textureLoader = new TextureLoader()

	const loadTexture = (path: string): Promise<typeof grassTexture> => {
		return new Promise((resolve) => {
			const texture = textureLoader.load(path, () => {
				resolve(texture)
			})
			texture.magFilter = NearestFilter
			texture.minFilter = NearestFilter
			texture.wrapS = RepeatWrapping
			texture.wrapT = RepeatWrapping
		})
	}

	const texturePaths = [
		'/textures/grass-texture.jpg',
		'/textures/dirt-texture.png',
		'/textures/stone-texture.png',
		'/textures/bedrock.png',
		'/textures/cobblestone.png',
		'/textures/darkoaklog.png',
		'/textures/glass.png',
		'/textures/wood.png'
	]

	const texturePromises = texturePaths.map(loadTexture)

	Promise.all(texturePromises).then(() => {
		isLoaded = true
	})

	const loadTextureSync = (path: string) => {
		const texture = textureLoader.load(path)
		texture.magFilter = NearestFilter
		texture.minFilter = NearestFilter
		texture.wrapS = RepeatWrapping
		texture.wrapT = RepeatWrapping
		return texture
	}

	const grassTexture = loadTextureSync('/textures/grass-texture.jpg')
	const dirtTexture = loadTextureSync('/textures/dirt-texture.png')
	const stoneTexture = loadTextureSync('/textures/stone-texture.png')
	const bedrockTexture = loadTextureSync('/textures/bedrock.png')
	const cobblestoneTexture = loadTextureSync('/textures/cobblestone.png')
	const darkoaklogTexture = loadTextureSync('/textures/darkoaklog.png')
	const glassTexture = loadTextureSync('/textures/glass.png')
	const woodTexture = loadTextureSync('/textures/wood.png')

	const grassTopMaterial = new MeshStandardMaterial({ map: grassTexture })
	const dirtMaterial = new MeshStandardMaterial({ map: dirtTexture })
	const stoneMaterial = new MeshStandardMaterial({ map: stoneTexture })
	const bedrockMaterial = new MeshStandardMaterial({ map: bedrockTexture })
	const cobblestoneMaterial = new MeshStandardMaterial({ map: cobblestoneTexture })
	const darkoaklogMaterial = new MeshStandardMaterial({ map: darkoaklogTexture })
	const glassMaterial = new MeshStandardMaterial({ map: glassTexture, transparent: true, opacity: 0.8 })
	const woodMaterial = new MeshStandardMaterial({ map: woodTexture })

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
	let bedrockMesh: InstancedMesh | null = $state(null)
	let cobblestoneMesh: InstancedMesh | null = $state(null)
	let darkoaklogMesh: InstancedMesh | null = $state(null)
	let glassMesh: InstancedMesh | null = $state(null)
	let woodMesh: InstancedMesh | null = $state(null)

	const bedrockY = -10

	const totalSize = chunkSize * (renderDistance * 2 + 1)
	const totalBlocks = totalSize * totalSize + 500

	const getBlockMaterial = (y: number, surfaceHeight: number): 'grass' | 'dirt' | 'stone' | 'bedrock' => {
		if (y <= bedrockY) return 'bedrock'
		if (y === surfaceHeight) return 'grass'
		if (y >= surfaceHeight - 3) return 'dirt'
		return 'stone'
	}

	// Retorna blocos expostos por causa de remoções adjacentes
	const getExposedBlocks = () => {
		const exposed: Array<{ x: number; y: number; z: number }> = []

		for (const key of removedBlocks) {
			const [rx, ry, rz] = key.split(',').map(Number)

			// Bloco abaixo (até bedrock)
			if (ry - 1 >= bedrockY && !isBlockRemoved(rx, ry - 1, rz)) {
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
				if (ry <= neighborHeight && ry >= bedrockY && !isBlockRemoved(n.x, ry, n.z)) {
					exposed.push({ x: n.x, y: ry, z: n.z })
				}
			}
		}

		return exposed
	}

	const updateTerrain = () => {
		if (!grassMesh || !dirtMesh || !stoneMesh || !bedrockMesh || !cobblestoneMesh || !darkoaklogMesh || !glassMesh || !woodMesh) return

		const currentChunkX = Math.floor(playerX / chunkSize)
		const currentChunkZ = Math.floor(playerZ / chunkSize)

		const startX = (currentChunkX - renderDistance) * chunkSize
		const startZ = (currentChunkZ - renderDistance) * chunkSize

		let grassIndex = 0
		let dirtIndex = 0
		let stoneIndex = 0
		let bedrockIndex = 0
		let cobblestoneIndex = 0
		let darkoaklogIndex = 0
		let glassIndex = 0
		let woodIndex = 0

		const hiddenPosition = { x: 0, y: -1000, z: 0 }

		for (let i = 0; i < totalBlocks; i++) {
			dummy.position.set(hiddenPosition.x, hiddenPosition.y, hiddenPosition.z)
			dummy.updateMatrix()
			grassMesh.setMatrixAt(i, dummy.matrix)
			dirtMesh.setMatrixAt(i, dummy.matrix)
			stoneMesh.setMatrixAt(i, dummy.matrix)
			bedrockMesh.setMatrixAt(i, dummy.matrix)
			cobblestoneMesh.setMatrixAt(i, dummy.matrix)
			darkoaklogMesh.setMatrixAt(i, dummy.matrix)
			glassMesh.setMatrixAt(i, dummy.matrix)
			woodMesh.setMatrixAt(i, dummy.matrix)
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

				const material = getBlockMaterial(height, height)
				if (material === 'bedrock') {
					bedrockMesh.setMatrixAt(bedrockIndex++, dummy.matrix)
				} else if (material === 'stone') {
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

			const surfaceHeight = getHeight(block.x, block.z)
			dummy.position.set(block.x, block.y, block.z)
			dummy.updateMatrix()

			const material = getBlockMaterial(block.y, surfaceHeight)
			if (material === 'bedrock') {
				bedrockMesh.setMatrixAt(bedrockIndex++, dummy.matrix)
			} else if (material === 'stone') {
				stoneMesh.setMatrixAt(stoneIndex++, dummy.matrix)
			} else if (material === 'dirt') {
				dirtMesh.setMatrixAt(dirtIndex++, dummy.matrix)
			} else {
				grassMesh.setMatrixAt(grassIndex++, dummy.matrix)
			}
		}

		// Renderiza blocos adicionados
		for (const [key, blockType] of addedBlocks) {
			const [bx, by, bz] = key.split(',').map(Number)

			// Verifica se está na área visível
			if (
				bx < startX ||
				bx >= startX + totalSize ||
				bz < startZ ||
				bz >= startZ + totalSize
			) continue

			dummy.position.set(bx, by, bz)
			dummy.updateMatrix()

			switch (blockType) {
				case 'dirt':
					dirtMesh.setMatrixAt(dirtIndex++, dummy.matrix)
					break
				case 'stone':
					stoneMesh.setMatrixAt(stoneIndex++, dummy.matrix)
					break
				case 'cobblestone':
					cobblestoneMesh.setMatrixAt(cobblestoneIndex++, dummy.matrix)
					break
				case 'darkoaklog':
					darkoaklogMesh.setMatrixAt(darkoaklogIndex++, dummy.matrix)
					break
				case 'glass':
					glassMesh.setMatrixAt(glassIndex++, dummy.matrix)
					break
				case 'wood':
					woodMesh.setMatrixAt(woodIndex++, dummy.matrix)
					break
			}
		}

		grassMesh.instanceMatrix.needsUpdate = true
		dirtMesh.instanceMatrix.needsUpdate = true
		stoneMesh.instanceMatrix.needsUpdate = true
		bedrockMesh.instanceMatrix.needsUpdate = true
		cobblestoneMesh.instanceMatrix.needsUpdate = true
		darkoaklogMesh.instanceMatrix.needsUpdate = true
		glassMesh.instanceMatrix.needsUpdate = true
		woodMesh.instanceMatrix.needsUpdate = true
	}

	$effect(() => {
		playerX
		playerZ
		removedBlocks
		addedBlocks
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

<T.InstancedMesh
	bind:ref={bedrockMesh}
	args={[geometry, bedrockMaterial, totalBlocks]}
	frustumCulled={false}
/>

<T.InstancedMesh
	bind:ref={cobblestoneMesh}
	args={[geometry, cobblestoneMaterial, totalBlocks]}
	frustumCulled={false}
/>

<T.InstancedMesh
	bind:ref={darkoaklogMesh}
	args={[geometry, darkoaklogMaterial, totalBlocks]}
	frustumCulled={false}
/>

<T.InstancedMesh
	bind:ref={glassMesh}
	args={[geometry, glassMaterial, totalBlocks]}
	frustumCulled={false}
/>

<T.InstancedMesh
	bind:ref={woodMesh}
	args={[geometry, woodMaterial, totalBlocks]}
	frustumCulled={false}
/>
