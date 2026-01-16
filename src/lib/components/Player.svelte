<script lang="ts">
	import { T, useTask, useThrelte } from '@threlte/core'
	import { onMount, onDestroy } from 'svelte'
	import * as THREE from 'three'

	interface Props {
		positionX?: number
		positionZ?: number
	}

	let { positionX = $bindable(0), positionZ = $bindable(0) }: Props = $props()

	const { camera, renderer } = useThrelte()

	let position = $state({ x: 0, y: 5, z: 0 })
	let rotation = $state({ x: 0, y: 0 })
	let keys: Record<string, boolean> = {}
	let isLocked = $state(false)

	const speed = 0.15
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

		if (keys['w']) {
			position.x += direction.x * speed
			position.z += direction.z * speed
		}
		if (keys['s']) {
			position.x -= direction.x * speed
			position.z -= direction.z * speed
		}
		if (keys['a']) {
			position.x -= sideVector.x * speed
			position.z -= sideVector.z * speed
		}
		if (keys['d']) {
			position.x += sideVector.x * speed
			position.z += sideVector.z * speed
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
