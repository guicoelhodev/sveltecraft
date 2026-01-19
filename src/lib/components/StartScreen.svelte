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

<div class="start-screen">
	<div class="content">
		<img src={coverImage} alt="SvelteCraft" class="cover-image" />

		<div class="buttons">
			<button class="mc-button" onclick={onPlay} disabled={!isLoaded}>
				{#if isLoaded}
					Jogar
				{:else}
					<span class="loading-text">Carregando...</span>
				{/if}
			</button>

			<a href={githubUrl} target="_blank" rel="noopener noreferrer" class="mc-button secondary">
				Saiba mais
			</a>
		</div>
	</div>
</div>

<style>
	.start-screen {
		position: fixed;
		inset: 0;
		background: linear-gradient(180deg, #1a1a2e 0%, #0f0f1a 100%);
		display: flex;
		align-items: center;
		justify-content: center;
		z-index: 100;
	}

	.content {
		display: flex;
		flex-direction: column;
		align-items: center;
		gap: 2rem;
	}

	.cover-image {
		max-width: 500px;
		width: 90vw;
		height: auto;
		image-rendering: pixelated;
	}

	.buttons {
		display: flex;
		flex-direction: column;
		gap: 1rem;
		width: 100%;
		max-width: 300px;
	}

	.mc-button {
		font-family: 'Press Start 2P', monospace;
		font-size: 0.875rem;
		padding: 1rem 2rem;
		background: linear-gradient(180deg, #7a7a7a 0%, #555555 50%, #3d3d3d 50%, #4a4a4a 100%);
		border: 3px solid #000;
		box-shadow:
			inset 2px 2px 0 rgba(255, 255, 255, 0.3),
			inset -2px -2px 0 rgba(0, 0, 0, 0.3);
		color: #fff;
		text-shadow:
			2px 2px 0 #3f3f3f,
			-1px -1px 0 #3f3f3f;
		cursor: pointer;
		transition: filter 0.1s;
		text-align: center;
		text-decoration: none;
		display: block;
	}

	.mc-button:hover:not(:disabled) {
		background: linear-gradient(180deg, #8a8aff 0%, #6565ff 50%, #4a4aff 50%, #5a5aff 100%);
	}

	.mc-button:active:not(:disabled) {
		box-shadow:
			inset -2px -2px 0 rgba(255, 255, 255, 0.3),
			inset 2px 2px 0 rgba(0, 0, 0, 0.3);
	}

	.mc-button:disabled {
		opacity: 0.6;
		cursor: not-allowed;
	}

	.mc-button.secondary {
		background: linear-gradient(180deg, #5a5a5a 0%, #404040 50%, #2d2d2d 50%, #3a3a3a 100%);
	}

	.mc-button.secondary:hover {
		background: linear-gradient(180deg, #6a6a6a 0%, #505050 50%, #3d3d3d 50%, #4a4a4a 100%);
	}

	.loading-text {
		animation: pulse 1.5s ease-in-out infinite;
	}

	@keyframes pulse {
		0%,
		100% {
			opacity: 1;
		}
		50% {
			opacity: 0.5;
		}
	}
</style>
