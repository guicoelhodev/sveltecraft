<script lang="ts">
	import { T } from '@threlte/core'
	import { SphereGeometry, MeshBasicMaterial } from 'three'

	interface Props {
		playerX: number
		playerZ: number
	}

	let { playerX, playerZ }: Props = $props()

	const sunDistance = 200
	const sunHeight = 150
	const sunSize = 15

	const geometry = new SphereGeometry(sunSize, 32, 32)
	const material = new MeshBasicMaterial({
		color: '#FFEA87'
	})

	// Posição do sol relativa ao player
	$effect(() => {
		sunX = playerX + sunDistance
		sunZ = playerZ - sunDistance
	})

	let sunX = $state(sunDistance)
	let sunZ = $state(-sunDistance)
</script>

<T.Mesh {geometry} {material} position.x={sunX} position.y={sunHeight} position.z={sunZ} />


