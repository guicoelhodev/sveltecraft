<script lang="ts">
	import { T, useTask, useThrelte } from '@threlte/core'
	import { onMount, onDestroy } from 'svelte'
	import * as THREE from 'three'

	interface Props {
		positionX?: number
		positionZ?: number
		getHeight: (x: number, z: number) => number
		targetBlock?: { x: number; y: number; z: number } | null
		isBlockRemoved: (x: number, y: number, z: number) => boolean
		removeBlock: (x: number, y: number, z: number) => void
	}

	let {
		positionX = $bindable(0),
		positionZ = $bindable(0),
		getHeight,
		targetBlock = $bindable(null),
		isBlockRemoved,
		removeBlock
	}: Props = $props()

	const { camera, renderer } = useThrelte()

	let position = $state({ x: 0, y: 10, z: 0 })
	let velocity = $state({ x: 0, y: 0, z: 0 })
	let rotation = $state({ x: 0, y: 0 })
	let keys: Record<string, boolean> = {}
	let isLocked = $state(false)
	let isGrounded = $state(false)

	const speed = 0.15
	const jumpForce = 0.3
	const gravity = 0.02
	const playerHeight = 1.7
	const sensitivity = 0.01
	const maxRaycastDistance = 8
	const raycastStep = 0.2
	const bedrockY = -2

	// Retorna a altura efetiva do chão considerando blocos removidos
	const getEffectiveHeight = (x: number, z: number): number => {
		const terrainHeight = getHeight(x, z)
		// Procura o bloco sólido mais alto na coluna (até bedrock)
		for (let y = terrainHeight; y >= bedrockY; y--) {
			if (y <= bedrockY || !isBlockRemoved(x, y, z)) {
				return y
			}
		}
		return bedrockY
	}

	const onKeyDown = (e: KeyboardEvent) => {
		keys[e.key.toLowerCase()] = true
	}

	const onKeyUp = (e: KeyboardEvent) => {
		keys[e.key.toLowerCase()] = false
	}

	const onMouseMove = (e: MouseEvent) => {
		if (!isLocked) return

		rotation.y -= e.movementX * sensitivity
		rotation.x -= e.movementY * sensitivity
		rotation.x = Math.max(-Math.PI / 2, Math.min(Math.PI / 2, rotation.x))
	}

	const onClick = () => {
		if (!isLocked) {
			renderer?.domElement.requestPointerLock()
		} else if (targetBlock) {
			removeBlock(targetBlock.x, targetBlock.y, targetBlock.z)
		}
	}

	const onPointerLockChange = () => {
		isLocked = document.pointerLockElement === renderer?.domElement
	}

	const updateRaycast = (cam: THREE.Camera) => {
		const direction = new THREE.Vector3()
		cam.getWorldDirection(direction)

		const rayOrigin = cam.position.clone()

		for (let dist = 0; dist <= maxRaycastDistance; dist += raycastStep) {
			const checkX = rayOrigin.x + direction.x * dist
			const checkY = rayOrigin.y + direction.y * dist
			const checkZ = rayOrigin.z + direction.z * dist

			const blockX = Math.round(checkX)
			const blockZ = Math.round(checkZ)
			const blockY = Math.round(checkY)
			const terrainHeight = getHeight(blockX, blockZ)

			// Verifica se há um bloco sólido nessa posição exata
			if (blockY >= bedrockY && blockY <= terrainHeight && !isBlockRemoved(blockX, blockY, blockZ)) {
				targetBlock = {
					x: blockX,
					y: blockY,
					z: blockZ
				}
				return
			}
		}
		targetBlock = null
	}

	useTask(() => {
		const cam = $camera
		if (!cam) return

		cam.rotation.order = 'YXZ'
		cam.rotation.x = rotation.x
		cam.rotation.y = rotation.y

		const direction = new THREE.Vector3()
		cam.getWorldDirection(direction)
		direction.y = 0
		direction.normalize()

		const sideVector = new THREE.Vector3()
		sideVector.crossVectors(cam.up, direction).normalize()

		let moveX = 0
		let moveZ = 0

		if (keys['w']) {
			moveX += direction.x * speed
			moveZ += direction.z * speed
		}
		if (keys['s']) {
			moveX -= direction.x * speed
			moveZ -= direction.z * speed
		}
		if (keys['d']) {
			moveX -= sideVector.x * speed
			moveZ -= sideVector.z * speed
		}
		if (keys['a']) {
			moveX += sideVector.x * speed
			moveZ += sideVector.z * speed
		}

		const newX = position.x + moveX
		const newZ = position.z + moveZ

		const effectiveHeightAtNewPos = getEffectiveHeight(Math.round(newX), Math.round(newZ)) + 1
		const playerFeetY = position.y - playerHeight

		if (effectiveHeightAtNewPos <= playerFeetY + 0.3) {
			position.x = newX
			position.z = newZ
		}

		velocity.y -= gravity
		position.y += velocity.y

		const groundHeight = getEffectiveHeight(Math.round(position.x), Math.round(position.z)) + 1 + playerHeight

		if (position.y <= groundHeight) {
			position.y = groundHeight
			velocity.y = 0
			isGrounded = true
		} else {
			isGrounded = false
		}

		if (keys[' '] && isGrounded) {
			velocity.y = jumpForce
			isGrounded = false
		}

		cam.position.set(position.x, position.y, position.z)

		positionX = position.x
		positionZ = position.z

		updateRaycast(cam)
	})

	onMount(() => {
		window.addEventListener('keydown', onKeyDown)
		window.addEventListener('keyup', onKeyUp)
		window.addEventListener('mousemove', onMouseMove)
		document.addEventListener('pointerlockchange', onPointerLockChange)
		renderer?.domElement.addEventListener('click', onClick)
	})

	onDestroy(() => {
		window.removeEventListener('keydown', onKeyDown)
		window.removeEventListener('keyup', onKeyUp)
		window.removeEventListener('mousemove', onMouseMove)
		document.removeEventListener('pointerlockchange', onPointerLockChange)
		renderer?.domElement.removeEventListener('click', onClick)
	})
</script>

<T.PerspectiveCamera makeDefault fov={75} />
