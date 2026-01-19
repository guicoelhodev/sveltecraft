<script lang="ts">
	import coverImage from '$lib/assets/svelte_craft.png'

	interface Props {
		isLoaded: boolean
		onPlay: () => void
	}

	let { isLoaded, onPlay }: Props = $props()

	const githubUrl = 'https://github.com/guicoelhodev/sveltecraft'
</script>

<svelte:head>
	<link rel="preconnect" href="https://fonts.googleapis.com" />
	<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin="anonymous" />
	<link
		href="https://fonts.googleapis.com/css2?family=Press+Start+2P&display=swap"
		rel="stylesheet"
	/>
</svelte:head>

<div class="fixed inset-0 z-50 flex items-center justify-center bg-black/40">
	<div class="flex flex-col items-center gap-8">
		<img src={coverImage} alt="SvelteCraft" class="pixelated h-auto w-[90vw] max-w-[600px]" />

		<div class="flex w-full max-w-[300px] flex-col gap-4">
			<button
				class="mc-btn cursor-pointer border-3 border-black px-8 py-4 text-center text-sm text-white transition-[filter] duration-100 disabled:cursor-not-allowed disabled:opacity-60"
				onclick={onPlay}
				disabled={!isLoaded}
			>
				{#if isLoaded}
					Jogar
				{:else}
					<span class="animate-pulse">Carregando...</span>
				{/if}
			</button>

			<a
				href={githubUrl}
				target="_blank"
				rel="noopener noreferrer"
				class="mc-btn mc-btn-secondary block cursor-pointer border-3 border-black px-8 py-4 text-center text-sm text-white no-underline transition-[filter] duration-100"
			>
				Saiba mais
			</a>
		</div>
	</div>
</div>

<style>
	.mc-btn {
		font-family: 'Press Start 2P', monospace;
		background: linear-gradient(180deg, #7a7a7a 0%, #555555 50%, #3d3d3d 50%, #4a4a4a 100%);
		box-shadow:
			inset 2px 2px 0 rgba(255, 255, 255, 0.3),
			inset -2px -2px 0 rgba(0, 0, 0, 0.3);
		text-shadow:
			2px 2px 0 #3f3f3f,
			-1px -1px 0 #3f3f3f;
	}

	.mc-btn:hover:not(:disabled) {
		background: linear-gradient(180deg, #8a8aff 0%, #6565ff 50%, #4a4aff 50%, #5a5aff 100%);
	}

	.mc-btn:active:not(:disabled) {
		box-shadow:
			inset -2px -2px 0 rgba(255, 255, 255, 0.3),
			inset 2px 2px 0 rgba(0, 0, 0, 0.3);
	}

	.mc-btn-secondary {
		background: linear-gradient(180deg, #5a5a5a 0%, #404040 50%, #2d2d2d 50%, #3a3a3a 100%);
	}

	.mc-btn-secondary:hover {
		background: linear-gradient(180deg, #6a6a6a 0%, #505050 50%, #3d3d3d 50%, #4a4a4a 100%);
	}
</style>
