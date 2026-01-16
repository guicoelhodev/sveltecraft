<script lang="ts">
	import { T } from '@threlte/core'
	import { InstancedMesh, BoxGeometry, MeshBasicMaterial, Object3D, Matrix4 } from 'three'

	interface Props {
		playerX: number
		playerZ: number
	}

	let { playerX, playerZ }: Props = $props()

	const cloudHeight = 50
	const chunkSize = 32
	const renderDistance = 3

	const dummy = new Object3D()
	const geometry = new BoxGeometry(4, 1.5, 4)
	const material = new MeshBasicMaterial({
		color: '#ffffff',
		transparent: true,
		opacity: 0.9
	})

	let cloudMesh: InstancedMesh | null = $state(null)

	// Número fixo de blocos por nuvem e nuvens totais
	const maxBlocksPerCloud = 25
	const chunksVisible = (renderDistance * 2 + 1) * (renderDistance * 2 + 1)
	const totalBlocks = chunksVisible * maxBlocksPerCloud

	// Cache das nuvens geradas
	const cloudCache = new Map<string, Matrix4[]>()

	// Hash simples
	const hash = (x: number, z: number, seed: number = 0): number => {
		const n = Math.sin(x * 12.9898 + z * 78.233 + seed) * 43758.5453
		return n - Math.floor(n)
	}

	// Gera matrizes de uma nuvem (cached)
	const getCloudMatrices = (cx: number, cz: number): Matrix4[] => {
		const key = `${cx},${cz}`
		if (cloudCache.has(key)) return cloudCache.get(key)!

		const matrices: Matrix4[] = []

		// Só 25% dos chunks têm nuvem
		if (hash(cx, cz, 1) > 0.75) {
			cloudCache.set(key, matrices)
			return matrices
		}

		const centerX = cx * chunkSize + hash(cx, cz, 10) * chunkSize * 0.5
		const centerZ = cz * chunkSize + hash(cx, cz, 20) * chunkSize * 0.5
		const cloudSize = Math.floor(hash(cx, cz, 30) * 3) + 2

		for (let dx = -cloudSize; dx <= cloudSize; dx++) {
			for (let dz = -cloudSize; dz <= cloudSize; dz++) {
				const dist = dx * dx + dz * dz
				const maxDist = cloudSize * cloudSize

				if (dist < maxDist && hash(cx + dx, cz + dz, 50) > 0.3) {
					const matrix = new Matrix4()
					matrix.setPosition(centerX + dx * 4, cloudHeight, centerZ + dz * 4)
					matrices.push(matrix)

					// Segunda camada só no centro
					if (dist < maxDist * 0.3 && hash(dx, dz, 100) > 0.7) {
						const matrix2 = new Matrix4()
						matrix2.setPosition(centerX + dx * 4, cloudHeight + 1.5, centerZ + dz * 4)
						matrices.push(matrix2)
					}
				}
			}
		}

		cloudCache.set(key, matrices)
		return matrices
	}

	// Guarda último chunk para evitar updates desnecessários
	let lastChunkX = -999
	let lastChunkZ = -999

	const hiddenMatrix = new Matrix4().setPosition(0, -1000, 0)

	const updateClouds = () => {
		if (!cloudMesh) return

		const currentChunkX = Math.floor(playerX / chunkSize)
		const currentChunkZ = Math.floor(playerZ / chunkSize)

		// Só atualiza se mudou de chunk
		if (currentChunkX === lastChunkX && currentChunkZ === lastChunkZ) return
		lastChunkX = currentChunkX
		lastChunkZ = currentChunkZ

		let blockIndex = 0

		// Itera pelos chunks visíveis
		for (let cz = -renderDistance; cz <= renderDistance; cz++) {
			for (let cx = -renderDistance; cx <= renderDistance; cx++) {
				const matrices = getCloudMatrices(currentChunkX + cx, currentChunkZ + cz)

				for (const matrix of matrices) {
					if (blockIndex < totalBlocks) {
						cloudMesh.setMatrixAt(blockIndex++, matrix)
					}
				}
			}
		}

		// Esconde o resto
		for (let i = blockIndex; i < totalBlocks; i++) {
			cloudMesh.setMatrixAt(i, hiddenMatrix)
		}

		cloudMesh.instanceMatrix.needsUpdate = true
	}

	$effect(() => {
		playerX
		playerZ
		updateClouds()
	})
</script>

<T.InstancedMesh
	bind:ref={cloudMesh}
	args={[geometry, material, totalBlocks]}
	frustumCulled={false}
	oncreate={updateClouds}
/>
