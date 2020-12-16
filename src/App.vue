<template>
	<div class="App">
		<div class="TitleCard" v-if="!hasStarted">
			<h1 class="TitleCard__artist">COEN</h1>
			<h2 class="TitleCard__songTitle">Planten Queen</h2>
			<button class="TitleCard__start" @click="start">Zingen maar!</button>
		</div>

		<ul
			v-if="hasStarted"
			class="Lyrics"
		>
			<li
				v-for="lyric in lyrics"
				:key="lyric.id"
				class="Lyric"
				:class="{
					'is-past': pastLyrics.includes(lyric),
					'is-active': activeLyrics.includes(lyric),
					'is-next-up': nextUpLyrics.includes(lyric),
				}"
			>
				<span
					class="Lyric__text"
					:style="activeLyrics.includes(lyric) ? {
						backgroundPosition: `${Math.round(Math.max(0, 1 - Math.min(1, (position - lyric.start)/(lyric.end - lyric.start))) * 100)}% 0px`,
					} : null"
				>
					{{ lyric.text }}
				</span>
			</li>
		</ul>

		<div
			v-if="hasStarted"
			class="Sticker"
		>
			<img
				:src="currentSticker"
				class="Sticker__image"
			/>
		</div>

		<audio
			preload
			class="NativePlayer"
			:class="{ 'is-hidden': !hasStarted }"
			ref="audioPlayer"
			controls	
		>
			<source src="/src/assets/plantenqueen.mp3" type="audio/mpeg">
		</audio>
	</div>
</template>

<script>
	import { lyrics } from './lyrics';
	import { choose } from 'roll-the-bones';
	import parseSrt from 'parse-srt';
	import gif1 from '/src/assets/stickers/1.gif';
	import gif2 from '/src/assets/stickers/2.gif';
	import gif3 from '/src/assets/stickers/3.gif';
	import gif4 from '/src/assets/stickers/4.gif';
	import gif5 from '/src/assets/stickers/5.gif';
	import gif6 from '/src/assets/stickers/6.gif';
	import gif9 from '/src/assets/stickers/9.gif';
	import gif10 from '/src/assets/stickers/10.gif';
	import gif11 from '/src/assets/stickers/11.gif';
	import gif12 from '/src/assets/stickers/giphy-1.gif';
	import gif13 from '/src/assets/stickers/giphy-3.gif';
	import gif14 from '/src/assets/stickers/giphy-4.gif';
	import gif15 from '/src/assets/stickers/giphy-7.gif';
	import gif16 from '/src/assets/stickers/giphy-8.gif';
	import gif17 from '/src/assets/stickers/giphy-9.gif';
	import gif18 from '/src/assets/stickers/giphy.gif';


	export default {
		data() {
			return {
				hasStarted: false,
				position: 0,
				lyrics: parseSrt(lyrics),
				timeCodes: [],
				currentSticker: this.getRandomSticker(),
			};
		},
		computed: {
			pastLyrics() {
				return this.lyrics.filter(lyric =>
					this.position > lyric.end
				);
			},
			activeLyrics() {
				return this.lyrics.filter(lyric =>
					this.position >= lyric.start &&
					this.position <= lyric.end
				);
			},
			nextUpLyrics() {
				return this.lyrics.filter(lyric => 
					this.position >= (lyric.start - 3) &&
					this.position < lyric.start
				);
			},
			lyricsToShow() {
				return [
					...this.activeLyrics,
					...this.nextUpLyrics,
				];
			}
		},
		methods: {
			start() {
				this.hasStarted = true;
				this.$refs.audioPlayer.play();
			},
			getRandomSticker() {
				return choose([gif1, gif2, gif3, gif4, gif5, gif6, gif9, gif10, gif11, gif12,gif13,gif14,gif15,gif16,gif17,gif18]);
			},
		},
		mounted() {
			this.$refs.audioPlayer.addEventListener('timeupdate', (event) => {
				this.position = this.$refs.audioPlayer.currentTime;
			});

			this.$refs.audioPlayer.addEventListener('play', (event) => {
				this.hasStarted = true;
			});

			window.setInterval(() => {
				this.currentSticker = this.getRandomSticker();
			}, 8000);
		},
	};
</script>

<style lang="scss">
	html,
	body {
		overflow: hidden;
		height: 100%;
		margin: 0;
		padding: 0;
	}

	ul {
		margin: 0;
		padding: 0;
	}

	.App {
		display: grid;
		align-content: center;
		height: 100%;
		background-image: url('./assets/plants.jpg');
		background-position: center;
		background-size: cover;
		background-repeat: no-repeat;
	}

	.TitleCard {
		text-align: center;
	}

	.TitleCard__artist,
	.TitleCard__songTitle {
		margin: 0;
		color: white;
	}

	.TitleCard__artist {
		font-family: sans-serif;
	}

	.TitleCard__songTitle {
		font-family: Charter, serif;
		font-style: italic;
		font-size: 4em;
	}

	.TitleCard__start {
		cursor: pointer;
		padding: 100px;
		margin: 40px;
		font-size: 3em;
		font-weight: bold;
		border: none;
		color: white;
		transition: transform .2s ease;
		background-color: transparent;
		background-image: url('/src/assets/wateringcan.gif');
		background-position: center center;
		background-size: contain;
		background-repeat: no-repeat;

		&:hover {
			transform: scale(1.2) rotate(-9deg);
		}
	}

	.NativePlayer {
		position: absolute;
		bottom: 0;
		left: 0;
		right: 0;
		width: 100%;

		&.is-hidden {
			display: none;
		}
	}

	.Lyrics {
		pointer-events: none;
		display: flex;
		flex-direction: column;
		transform: translateY(-50%);
		text-align: center;
	}

	.Lyric {
		overflow: hidden;
		height: 0;
		opacity: 0;
		list-style: none;
		font-family: sans-serif;
		font-weight: bold;
		font-size: 3em;
		transition: all 0.5s ease;

		&.is-active,
		&.is-past,
		&.is-next-up {
			height: 80px;
		}

		&.is-past {
			opacity: 0.8;
		}

		&.is-active {
			opacity: 1;
			transform: scale(1.2);
		}

		&.is-next-up {
			opacity: 0.8;
		}
	}

	.Lyric__text {
		--base-color: white;
		--fill-color: rgb(255, 211, 117);
		color: var(--base-color);
		background-position: 100%;
		transition: background-position 0.5s ease;

		.Lyric.is-active & {
			position: relative;
			-webkit-text-stroke: 2px black;
			background: linear-gradient(to right, var(--fill-color), var(--fill-color) 50%, var(--base-color) 50%);
			background-clip: text;
			-webkit-background-clip: text;
			-webkit-text-fill-color: transparent;
			background-size: 200% 100%;
			background-position: 100%;
		}

		.Lyric.is-past & {
			color: var(--fill-color);
		}
	}

	.Sticker {
		display: flex;
		justify-content: center;
		align-items: center;
		position: absolute;
		bottom: 0;
		width: 100%;
		height: 50%;
	}

	.Sticker__image {
		max-height: 80%;
	}
</style>
