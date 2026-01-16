<script lang="ts">
	import { T, useTask, useThrelte } from '@threlte/core'
	import { RigidBody, Collider } from '@threlte/rapier'
	import { onMount, onDestroy } from 'svelte'
	import * as THREE from 'three'
	import type Rapier from '@dimforge/rapier3d-compat'

	interface Props {
		positionX?: number
		positionZ?: number
	}

	let { positionX = $bindable(0), positionZ = $bindable(0) }: Props = $props()

	const { camera, renderer } = useThrelte()

	let rigidBody: Rapier.RigidBody | undefined = $state()
	let rotation = $state({ x: 0, y: 0 })
	let keys: Record<string, boolean> = {}
	let isLocked = $state(false)

	const speed = 8
	const jumpForce = 8
	const sensitivity = 0.005

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
		if (!cam || !rigidBody) return

		const position = rigidBody.translation()

		if (position.y < -50) {
			rigidBody.setTranslation({ x: 0, y: 10, z: 0 }, true)
			rigidBody.setLinvel({ x: 0, y: 0, z: 0 }, true)
			return
		}

		cam.rotation.order = 'YXZ'
		cam.rotation.x = rotation.x
		cam.rotation.y = rotation.y

		const direction = new THREE.Vector3()
		cam.getWorldDirection(direction)
		direction.y = 0
		direction.normalize()

		const sideVector = new THREE.Vector3()
		sideVector.crossVectors(cam.up, direction).normalize()

		const velocity = rigidBody.linvel()
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
		if (keys['a']) {
			moveX -= sideVector.x * speed
			moveZ -= sideVector.z * speed
		}
		if (keys['d']) {
			moveX += sideVector.x * speed
			moveZ += sideVector.z * speed
		}

		rigidBody.setLinvel({ x: moveX, y: velocity.y, z: moveZ }, true)

		if (keys[' '] && Math.abs(velocity.y) < 0.5) {
			rigidBody.setLinvel({ x: velocity.x, y: jumpForce, z: velocity.z }, true)
		}

		cam.position.set(position.x, position.y + 1.5, position.z)

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

<RigidBody bind:rigidBody type="dynamic" position={[0, 5, 0]} lockRotations>
	<Collider shape="cuboid" args={[0.3, 0.9, 0.3]} />
</RigidBody>
