<template>
	<div class="App">
		<audio
			class="NativePlayer"
			ref="audioPlayer"
			controls
			src="/src/assets/plantenqueen.mp3"
		/>

		<transition name="slide-fade">
			<div class="TitleCard" v-if="!hasStarted">
				<h1 class="TitleCard__artist">COEN</h1>
				<h2 class="TitleCard__songTitle">Planten Queen</h2>
				<button class="TitleCard__start" @click="start">Zingen maar!</button>
			</div>
		</transition>

		<transition name="slide-fade">
			<ul class="Lyrics" v-if="hasStarted">
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
		</transition>
	</div>
</template>

<script>
	import { lyrics } from './lyrics';
	import parseSrt from 'parse-srt';

	export default {
		data() {
			return {
				hasStarted: false,
				position: 0,
				lyrics: parseSrt(lyrics),
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
			}
		},
		mounted() {
			this.$refs.audioPlayer.addEventListener('timeupdate', (event) => {
				this.position = this.$refs.audioPlayer.currentTime;
			});

			this.$refs.audioPlayer.addEventListener('play', (event) => {
				this.hasStarted = true;
			});
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
		text-shadow: 2px 2px 24px gold, 0px 0px 10px black;
		-webkit-text-stroke: 1px gold;
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
		padding: 20px;
		margin: 20px;
		font-size: 3em;
		font-weight: bold;
		background-color: white;
		border-radius: 20px;
		border: none;
		transition: transform .2s ease;

		&:hover {
			transform: scale(1.2);
		}
	}

	.NativePlayer {
		position: absolute;
		bottom: 0;
		left: 0;
		right: 0;
		width: 100%;
	}

	.Lyrics {
		display: flex;
		flex-direction: column;
		transform: translateY(-40%);
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
</style>
