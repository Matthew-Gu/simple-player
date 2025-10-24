<script setup lang="ts">
import { ref, useTemplateRef, onMounted } from 'vue';
import { useMediaControls, useFullscreen, useEventListener } from '@vueuse/core';
import PlayPause from './controls/PlayPause.vue';
import TimeDisplay from './controls/TimeDisplay.vue';
import RateCtrl from './controls/RateCtrl.vue';
import VolumeCtrl from './controls/VolumeCtrl.vue';
import Settings from './controls/Settings.vue';
import FullScreen from './controls/FullScreen.vue';
import ProgressBar from './controls/ProgressBar.vue';
import ControlsWrapper from './controls/ControlsWrapper.vue';

const props = defineProps({
	src: {
		type: String,
		required: true
	},
	poster: {
		type: String,
		default: ''
	}
});

const videoRef = useTemplateRef('video');
const videoPlayer = useTemplateRef('videoPlayer');

// 媒体控制
const controls = useMediaControls(videoRef, {
	src: props.src
});

const mirror = ref(false);
const { toggle: toggleFullscreen } = useFullscreen(videoPlayer);
const { playing, currentTime, duration, rate, volume, muted, isPictureInPicture, togglePictureInPicture } = controls;

const togglePlay = () => (playing.value = !playing.value);
const toggleMuted = () => (muted.value = !muted.value);
const toggleMirror = () => (mirror.value = !mirror.value);

const controlsVisible = ref(true);
const isOverControls = ref(false);
const HIDE_DELAY = 1500;
let hideTimer: NodeJS.Timeout | null = null;

const clearHideTimer = () => {
	if (hideTimer) {
		clearTimeout(hideTimer);
		hideTimer = null;
	}
};

const showControls = () => {
	controlsVisible.value = true;
	clearHideTimer();
};

const hideControls = () => {
	if (isOverControls.value || !playing.value) return;
	controlsVisible.value = false;
};

const scheduleHideControls = () => {
	clearHideTimer();
	hideTimer = setTimeout(() => hideControls(), HIDE_DELAY);
};

const TIME_ADD = 5;
const PRESS_DELAY = 300;
const SEEK_PLAY_RATE = 3;
let isLonePress = false;
let originRate = 1;
let originPlayState = false;

let pressTimer: NodeJS.Timeout | null = null;

const handlePressStart = () => {
	if (pressTimer) return;
	pressTimer = setTimeout(() => {
		originRate = rate.value;
		originPlayState = playing.value;
		isLonePress = true;
		playing.value = true;
		rate.value = SEEK_PLAY_RATE;
	}, PRESS_DELAY);
};

const handlePressEnd = () => {
	if (pressTimer) {
		clearTimeout(pressTimer);
		pressTimer = null;
	}

	if (isLonePress) {
		rate.value = originRate;
		playing.value = originPlayState;
		isLonePress = false;
	} else {
		seekForward();
	}
};

const seekForward = () => {
	currentTime.value = Math.min(currentTime.value + TIME_ADD, duration.value);
};

const seekBackward = () => {
	currentTime.value = Math.max(currentTime.value - TIME_ADD, 0);
};

onMounted(() => {
	// 视频区域交互
	useEventListener(videoPlayer, 'mousemove', () => {
		showControls();
		scheduleHideControls();
	});

	useEventListener(videoPlayer, 'mouseleave', scheduleHideControls);

	// 移动端点击控制
	useEventListener(videoPlayer, 'touchstart', () => {
		clearHideTimer();
		if (!controlsVisible.value) {
			showControls();
			scheduleHideControls();
		} else if (!isOverControls.value) {
			hideControls();
		}
	});

	useEventListener(videoPlayer, 'keydown', (e) => {
		if (e.code === 'ArrowRight') {
			handlePressStart();
		} else if (e.code === 'ArrowLeft') {
			seekBackward();
		}

		if (e.code == 'Space') {
			togglePlay();
		}
	});

	useEventListener(videoPlayer, 'keyup', (e) => {
		if (e.code === 'ArrowRight') {
			handlePressEnd();
		}
	});
});
</script>

<template>
	<div ref="videoPlayer" class="simple-player-container" :tabindex="0" autofocus>
		<div class="video-player-primary-area">
			<div class="video-player-wrapper" @click="() => (togglePlay(), (isOverControls = false))">
				<video ref="video" :class="{ mirror }" crossorigin="anonymous" :poster="poster" />
			</div>

			<controls-wrapper :hide="!controlsVisible && playing" @hover="(isOver) => (isOverControls = isOver)">
				<template #top>
					<progress-bar v-model:current-time="currentTime" :duration="duration" :video-ref="videoRef" />
				</template>

				<template #bottom-left>
					<play-pause :playing="playing" @toggle-play="togglePlay" />

					<time-display :current-time="currentTime" :duration="duration" />
				</template>

				<template #bottom-right>
					<rate-ctrl v-model:rate="rate" />

					<volume-ctrl v-model:volume="volume" :muted="muted" @toggle-muted="toggleMuted" />

					<settings
						:pic-in-pic="isPictureInPicture"
						:mirror="mirror"
						@toggle-mirror="toggleMirror"
						@toggle-pip="togglePictureInPicture" />

					<full-screen @toggle-full="toggleFullscreen" />
				</template>
			</controls-wrapper>
		</div>
	</div>
</template>

<style scoped lang="scss">
.simple-player-container {
	outline: none;
	.video-player-primary-area {
		position: relative;
		width: 100%;
		height: 100%;
		background: rgba(0, 0, 1, 0.1);
		cursor: pointer;

		.video-player-wrapper {
			position: relative;
			display: block;
			width: 100%;
			height: 100%;

			video {
				display: block;
				width: 100%;
				height: 100%;

				&.mirror {
					transform: scaleX(-1);
				}
			}

			&::after {
				position: absolute;
				content: '';
				inset: 0;
			}
		}
	}
}
</style>
