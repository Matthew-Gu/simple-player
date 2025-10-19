<script setup lang="ts">
import { ref, computed, watch } from 'vue';
import { useMouseInElement, useEventListener } from '@vueuse/core';
import CtrlBtn from './CtrlBtn.vue';

const props = defineProps<{ muted: boolean; volume: number }>();

const emit = defineEmits<{
	(e: 'toggle-muted'): void;
	(e: 'update:volume', value: number): void;
}>();

const progressRef = ref<HTMLDivElement>();
const volumeBoxRef = ref<HTMLDivElement>();
const dragging = ref(false);

const { elementY, elementHeight } = useMouseInElement(progressRef);

const progressPercent = computed(() => {
	if (!props.volume) return 0;
	return Math.round(props.volume * 100);
});

const startDragging = () => {
	dragging.value = true;
};

const stopDragging = () => {
	dragging.value = false;
};

const handleWheel = (e: WheelEvent) => {
	e.preventDefault();

	const step = 0.02;
	let newVolume = props.volume;

	if (e.deltaY < 0) {
		newVolume += step;
	} else if (e.deltaY > 0) {
		newVolume -= step;
	}

	newVolume = Math.max(0, Math.min(1, newVolume));

	const volumePercent = Math.round(newVolume * 100);
	const finalVolume = volumePercent / 100;

	emit('update:volume', finalVolume);
};

watch([dragging, elementY], () => {
	if (!dragging.value) return;
	const progress = Math.max(0, Math.min(1, 1 - elementY.value / elementHeight.value));

	const volumePercent = Math.round(progress * 100);
	const volume = volumePercent / 100;

	emit('update:volume', volume);
});

const toggleMuted = () => {
	emit('toggle-muted');
};

useEventListener(window, 'mouseup', stopDragging, { passive: true });
useEventListener(window, 'touchend', stopDragging, { passive: true });
useEventListener(window, 'touchcancel', stopDragging, { passive: true });
useEventListener(volumeBoxRef, 'wheel', handleWheel, { passive: false });
</script>

<template>
	<ctrl-btn class="player-ctrl-volume">
		<div class="player-svg-icon" @click="toggleMuted">
			<svg
				v-show="!muted"
				xmlns="http://www.w3.org/2000/svg"
				viewBox="0 0 24 24"
				fill="none"
				stroke="currentColor"
				stroke-width="2"
				stroke-linecap="round"
				stroke-linejoin="round"
				class="icon-tabler">
				<path stroke="none" d="M0 0h24v24H0z" fill="none" />
				<path d="M15 8a5 5 0 0 1 0 8" />
				<path d="M17.7 5a9 9 0 0 1 0 14" />
				<path
					d="M6 15h-2a1 1 0 0 1 -1 -1v-4a1 1 0 0 1 1 -1h2l3.5 -4.5a.8 .8 0 0 1 1.5 .5v14a.8 .8 0 0 1 -1.5 .5l-3.5 -4.5" />
			</svg>
			<svg
				v-show="muted"
				xmlns="http://www.w3.org/2000/svg"
				viewBox="0 0 24 24"
				fill="none"
				stroke="currentColor"
				stroke-width="2"
				stroke-linecap="round"
				stroke-linejoin="round"
				class="icon-tabler">
				<path stroke="none" d="M0 0h24v24H0z" fill="none" />
				<path d="M15 8a5 5 0 0 1 1.912 4.934m-1.377 2.602a5 5 0 0 1 -.535 .464" />
				<path d="M17.7 5a9 9 0 0 1 2.362 11.086m-1.676 2.299a9 9 0 0 1 -.686 .615" />
				<path
					d="M9.069 5.054l.431 -.554a.8 .8 0 0 1 1.5 .5v2m0 4v8a.8 .8 0 0 1 -1.5 .5l-3.5 -4.5h-2a1 1 0 0 1 -1 -1v-4a1 1 0 0 1 1 -1h2l1.294 -1.664" />
				<path d="M3 3l18 18" />
			</svg>
		</div>
		<div ref="volumeBoxRef" class="player-volume-box">
			<div class="volume-number">{{ progressPercent }}</div>
			<div
				ref="progressRef"
				class="volume-progress"
				@mousedown="startDragging"
				@touchstart.passive="startDragging">
				<div class="progress-schedule" :style="{ '--progress': `${progressPercent}%` }">
					<div class="progress-thumb"></div>
				</div>
			</div>
		</div>
	</ctrl-btn>
</template>

<style lang="scss" scoped>
.player-ctrl-volume {
	position: relative;

	&:hover {
		.player-volume-box {
			display: block;
		}
	}

	.player-volume-box {
		display: none;
		position: absolute;
		bottom: 32px;
		left: 50%;
		transform: translateX(-50%);
		border-radius: 2px;
		background: rgba(0, 0, 0, 0.6);
		width: 40px;
		height: 140px;
        user-select: none;

		.volume-number {
			font-size: 14px;
			line-height: 32px;
			color: #ffffff;
			text-align: center;
		}

		.volume-progress {
			position: relative;
			width: 4px;
			height: 100px;
			background: rgba(255, 255, 255, 0.3);
			cursor: pointer;
			margin: 0 auto;
			.progress-schedule {
				position: absolute;
				bottom: 0;
				width: 100%;
				max-height: 100%;
				height: var(--progress, 0%);
				background: rgba(255, 255, 255, 0.7);

				.progress-thumb {
					position: absolute;
					top: 0;
					left: 50%;
					transform: translate(-50%, -50%);
					width: 8px;
					height: 8px;
					border-radius: 50%;
					background: #ffffff;

					&:hover {
						zoom: 1.2;
					}
				}
			}
		}

		&::after {
			position: absolute;
			content: '';
			width: 36px;
			height: 10px;
			bottom: 0;
			left: 50%;
			transform: translate(-50%, 100%);
		}
	}
}
</style>
