<script lang="ts">
	import { T } from '@threlte/core'
	import { BoxGeometry, EdgesGeometry, LineBasicMaterial } from 'three'

	interface Props {
		targetBlock: { x: number; y: number; z: number } | null
	}

	let { targetBlock }: Props = $props()

	const boxGeometry = new BoxGeometry(1.01, 1.01, 1.01)
	const edgesGeometry = new EdgesGeometry(boxGeometry)
	const lineMaterial = new LineBasicMaterial({ color: 0x000000, linewidth: 2 })
</script>

{#if targetBlock}
	<T.LineSegments
		geometry={edgesGeometry}
		material={lineMaterial}
		position.x={targetBlock.x}
		position.y={targetBlock.y}
		position.z={targetBlock.z}
	/>
{/if}
