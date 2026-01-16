<script lang="ts">
	import { Canvas } from '@threlte/core'
	import Scene from '$lib/components/Scene.svelte'
</script>

<div class="h-screen w-screen">
	<Canvas>
		<Scene />
	</Canvas>
	<!-- Crosshair -->
	<div class="pointer-events-none fixed inset-0 flex items-center justify-center">
		<div class="h-6 w-0.5 bg-white opacity-70"></div>
		<div class="absolute h-0.5 w-6 bg-white opacity-70"></div>
	</div>
</div>
