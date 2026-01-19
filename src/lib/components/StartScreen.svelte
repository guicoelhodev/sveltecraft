<script lang="ts">
	import { onMount } from 'svelte'
	import coverImage from '$lib/assets/svelte_craft.png'

	interface Props {
		isLoaded: boolean
		onPlay: () => void
		sensitivity?: number
		onSensitivityChange?: (value: number) => void
		isPaused?: boolean
	}

	let { isLoaded, onPlay, sensitivity = 0.002, onSensitivityChange, isPaused = false }: Props = $props()

	const githubUrl = 'https://github.com/guicoelhodev/sveltecraft'

	let isMobile = $state(false)

	const minSensitivity = 0.0005
	const maxSensitivity = 0.01

	const handleSensitivityChange = (e: Event) => {
		const target = e.target as HTMLInputElement
		const value = parseFloat(target.value)
		onSensitivityChange?.(value)
	}

	onMount(() => {
		const checkMobile = () => {
			const hasTouchScreen = 'ontouchstart' in window || navigator.maxTouchPoints > 0
			const isSmallScreen = window.innerWidth < 768
			isMobile = hasTouchScreen || isSmallScreen
		}
		checkMobile()
		window.addEventListener('resize', checkMobile)
		return () => window.removeEventListener('resize', checkMobile)
	})

	const canPlay = $derived(isLoaded && !isMobile)
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
		{#if !isPaused}
			<img src={coverImage} alt="SvelteCraft" class="pixelated h-auto w-[90vw] max-w-[600px]" />
		{:else}
			<h1 class="mc-title text-2xl text-white">Pausado</h1>
		{/if}


		<div class="flex w-full max-w-[300px] flex-col gap-4">
			{#if isMobile && !isPaused}
				<div class="mc-panel border-3 border-black px-4 py-3 text-center text-xs leading-relaxed text-white">
					Este protótipo está disponível apenas para computador devido ao uso do teclado.
				</div>
			{/if}


			<button
				class="mc-btn cursor-pointer border-3 border-black px-8 py-4 text-center text-sm text-white transition-[filter] duration-100 disabled:cursor-not-allowed disabled:opacity-60"
				onclick={onPlay}
				disabled={!canPlay}
			>
				{#if !isLoaded}
					<span class="animate-pulse">Carregando...</span>
				{:else if isPaused}
					Continuar
				{:else}
					Jogar
				{/if}
			</button>

			<div class="mc-settings border-3 border-black px-4 py-3">
				<label for="sensitivity-slider" class="mc-label mb-2 block text-xs text-white">Sensibilidade do Mouse</label>
				<input
					id="sensitivity-slider"
					type="range"
					min={minSensitivity}
					max={maxSensitivity}
					step="0.0005"
					value={sensitivity}
					oninput={handleSensitivityChange}
					class="mc-slider w-full"
				/>
				<div class="mt-1 text-center text-xs text-white/70">
					{(sensitivity * 1000).toFixed(1)}
				</div>
			</div>

			{#if !isPaused}
				<a
					href={githubUrl}
					target="_blank"
					rel="noopener noreferrer"
					class="mc-btn mc-btn-secondary block cursor-pointer border-3 border-black px-8 py-4 text-center text-sm text-white no-underline transition-[filter] duration-100"
				>
					Saiba mais
				</a>
			{/if}
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

	.mc-panel {
		font-family: 'Press Start 2P', monospace;
		background: linear-gradient(180deg, #8b0000 0%, #5c0000 50%, #3d0000 50%, #4a0000 100%);
		box-shadow:
			inset 2px 2px 0 rgba(255, 255, 255, 0.2),
			inset -2px -2px 0 rgba(0, 0, 0, 0.3);
		text-shadow:
			2px 2px 0 #3f0000,
			-1px -1px 0 #3f0000;
	}

	.mc-title {
		font-family: 'Press Start 2P', monospace;
		text-shadow:
			3px 3px 0 #3f3f3f,
			-1px -1px 0 #3f3f3f;
	}

	.mc-settings {
		font-family: 'Press Start 2P', monospace;
		background: linear-gradient(180deg, #5a5a5a 0%, #404040 50%, #2d2d2d 50%, #3a3a3a 100%);
		box-shadow:
			inset 2px 2px 0 rgba(255, 255, 255, 0.2),
			inset -2px -2px 0 rgba(0, 0, 0, 0.3);
	}

	.mc-label {
		text-shadow:
			1px 1px 0 #3f3f3f,
			-1px -1px 0 #3f3f3f;
	}

	.mc-slider {
		-webkit-appearance: none;
		appearance: none;
		height: 8px;
		background: linear-gradient(180deg, #3a3a3a 0%, #2a2a2a 100%);
		border: 2px solid #1a1a1a;
		cursor: pointer;
	}

	.mc-slider::-webkit-slider-thumb {
		-webkit-appearance: none;
		appearance: none;
		width: 16px;
		height: 20px;
		background: linear-gradient(180deg, #7a7a7a 0%, #555555 50%, #3d3d3d 50%, #4a4a4a 100%);
		border: 2px solid #1a1a1a;
		cursor: pointer;
	}

	.mc-slider::-moz-range-thumb {
		width: 16px;
		height: 20px;
		background: linear-gradient(180deg, #7a7a7a 0%, #555555 50%, #3d3d3d 50%, #4a4a4a 100%);
		border: 2px solid #1a1a1a;
		cursor: pointer;
	}

	.mc-slider::-webkit-slider-thumb:hover {
		background: linear-gradient(180deg, #8a8aff 0%, #6565ff 50%, #4a4aff 50%, #5a5aff 100%);
	}

	.mc-slider::-moz-range-thumb:hover {
		background: linear-gradient(180deg, #8a8aff 0%, #6565ff 50%, #4a4aff 50%, #5a5aff 100%);
	}
</style>
