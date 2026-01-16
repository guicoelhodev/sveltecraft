<script lang="ts">
	import { T, useTask, useThrelte } from '@threlte/core'
	import { onMount, onDestroy } from 'svelte'
	import * as THREE from 'three'

	interface Props {
		positionX?: number
		positionZ?: number
		getHeight: (x: number, z: number) => number
	}

	let { positionX = $bindable(0), positionZ = $bindable(0), getHeight }: Props = $props()

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
		renderer?.domElement.requestPointerLock()
	}

	const onPointerLockChange = () => {
		isLocked = document.pointerLockElement === renderer?.domElement
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

		const terrainHeightAtNewPos = getHeight(Math.round(newX), Math.round(newZ)) + 1
		const playerFeetY = position.y - playerHeight

		if (terrainHeightAtNewPos <= playerFeetY + 0.3) {
			position.x = newX
			position.z = newZ
		}

		velocity.y -= gravity
		position.y += velocity.y

		const groundHeight = getHeight(Math.round(position.x), Math.round(position.z)) + 1 + playerHeight

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
