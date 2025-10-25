<script setup lang="ts">
import { ref, useTemplateRef, onMounted } from 'vue';
import { useMediaControls, useFullscreen, useEventListener } from '@vueuse/core';
import PlayPause from './controls/PlayPause.vue';
import TimeDisplay from './controls/TimeDisplay.vue';
import RateCtrl from './controls/RateCtrl.vue';
import VolumeCtrl from './controls/VolumeCtrl.vue';
import PipBtn from './controls/PipBtn.vue';
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
const touchLeft = useTemplateRef('touchLeft');
const touchRight = useTemplateRef('touchRight');

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

// 短按时间增量
const TIME_ADD = 3;
// 长按倍数
const SEEK_PLAY_RATE = 3;
// 长按相关状态
const originRate = ref(1);
const originPlayState = ref(false);
let previewTimer: number | null = null;

// 桌面端方向键状态
let keyPressTimer: number | null = null;
let isPreviewingWithKey = false;
const KEY_SHORT_PRESS_DELAY = 200; // 小于这个算短按

const handleKeyShortPress = (isForward: boolean) => {
	if (isForward) {
		currentTime.value = Math.min(currentTime.value + TIME_ADD, duration.value);
	} else {
		currentTime.value = Math.max(currentTime.value - TIME_ADD, 0);
	}
};

const startKeyPreview = (isForward: boolean) => {
	isPreviewingWithKey = true;
	originRate.value = rate.value;
	originPlayState.value = playing.value;

	if (isForward) {
		playing.value = true;
		rate.value = SEEK_PLAY_RATE;
	} else {
		// 快退预览（模拟）
		playing.value = false;
		previewTimer = window.setInterval(() => {
			currentTime.value = Math.max(0, currentTime.value - TIME_ADD);
		}, 100);
	}
};

const stopKeyPreview = () => {
	if (!isPreviewingWithKey) return;
	isPreviewingWithKey = false;

	if (previewTimer) {
		clearInterval(previewTimer);
		previewTimer = null;
	}

	rate.value = originRate.value;
	playing.value = originPlayState.value;
};

onMounted(() => {
	useEventListener(videoPlayer, 'keydown', (e) => {
		if (e.code === 'ArrowRight' || e.code === 'ArrowLeft') {
			e.preventDefault();
			// 如果已经在预览，忽略重复 keydown
			if (isPreviewingWithKey) return;

			const isForward = e.code === 'ArrowRight';

			// 启动短按检测计时器
			keyPressTimer = window.setTimeout(() => {
				startKeyPreview(isForward);
			}, KEY_SHORT_PRESS_DELAY);
		} else if (e.code === 'Space') {
			e.preventDefault();
			togglePlay();
		}
	});

	useEventListener(videoPlayer, 'keyup', (e) => {
		if (e.code === 'ArrowRight' || e.code === 'ArrowLeft') {
			e.preventDefault();

			if (keyPressTimer) {
				// 在阈值内松开视为短按
				clearTimeout(keyPressTimer);
				keyPressTimer = null;

				if (!isPreviewingWithKey) {
					// 执行单次 seek
					handleKeyShortPress(e.code === 'ArrowRight');
				}
			}

			// 如果是预览中，停止预览
			if (isPreviewingWithKey) {
				stopKeyPreview();
			}
		}
	});

	// 失焦时清理
	useEventListener(window, 'blur', () => {
		if (keyPressTimer) clearTimeout(keyPressTimer);
		if (isPreviewingWithKey) stopKeyPreview();
	});

	// 移动端 touch 区域（仅长按预览
	const bindTouch = (el: Element | null, isForward: boolean) => {
		if (!el) return;
		let pressTimer: number | null = null;
		let hasMoved = false;

		const reset = () => {
			if (pressTimer) clearTimeout(pressTimer);
			pressTimer = null;
			hasMoved = false;
		};

		useEventListener(
			el,
			'touchstart',
			(e) => {
				e.preventDefault();
				reset();
				pressTimer = window.setTimeout(() => {
					if (!hasMoved) {
						// 长按开始：预览
						originRate.value = rate.value;
						originPlayState.value = playing.value;
						if (isForward) {
							playing.value = true;
							rate.value = SEEK_PLAY_RATE;
						} else {
							playing.value = false;
							previewTimer = window.setInterval(() => {
								currentTime.value = Math.max(0, currentTime.value - TIME_ADD);
							}, 100);
						}
					}
				}, 300); // 移动端长按阈值
			},
			{ passive: false }
		);

		useEventListener(el, 'touchmove', () => {
			hasMoved = true;
			reset();
		});

		useEventListener(el, 'touchend', () => {
			reset();
			rate.value = originRate.value;
			playing.value = originPlayState.value;
			if (previewTimer) {
				clearInterval(previewTimer);
				previewTimer = null;
			}
		});

		useEventListener(el, 'touchcancel', () => {
			reset();
			rate.value = originRate.value;
			playing.value = originPlayState.value;
			if (previewTimer) {
				clearInterval(previewTimer);
				previewTimer = null;
			}
		});
	};

	bindTouch(touchLeft.value, false);
	bindTouch(touchRight.value, true);

	useEventListener(videoPlayer, 'mousemove', () => {
		showControls();
		scheduleHideControls();
	});
	useEventListener(videoPlayer, 'mouseleave', scheduleHideControls);
	useEventListener(videoPlayer, 'touchstart', () => {
		clearHideTimer();
		if (!controlsVisible.value) {
			showControls();
			scheduleHideControls();
		} else if (!isOverControls.value) {
			hideControls();
		}
	});
});
</script>

<template>
	<div ref="videoPlayer" class="simple-player-container" :tabindex="0" autofocus>
		<div class="video-player-primary-area">
			<div class="video-player-wrapper" @click="() => (togglePlay(), (isOverControls = false))">
				<video ref="video" :class="{ mirror }" crossorigin="anonymous" :poster="poster" />
				<div class="video-touch-layer">
					<div ref="touchLeft" class="touch-area touch-left"></div>
					<div ref="touchMiddle" class="touch-area touch-middle"></div>
					<div ref="touchRight" class="touch-area touch-right"></div>
				</div>
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

					<settings :pic-in-pic="isPictureInPicture" :mirror="mirror" @toggle-mirror="toggleMirror" />

					<pip-btn :is-pip="isPictureInPicture" @toggle-pip="togglePictureInPicture" />

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

			.video-touch-layer {
				position: absolute;
				content: '';
				inset: 0;
				display: flex;

				.touch-area {
					flex: 1 1 0;
					touch-action: manipulation;
				}
			}
		}
	}
}
</style>
