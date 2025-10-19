<script setup lang="ts">
import { ref, computed, watch } from 'vue';
import { useMouseInElement, useEventListener } from '@vueuse/core';

const props = defineProps<{
	currentTime: number;
	duration: number;
}>();

const emit = defineEmits<{
	(e: 'update:currentTime', value: number): void;
}>();

const progressRef = ref<HTMLDivElement>();
const dragging = ref(false);

const { elementX, elementWidth } = useMouseInElement(progressRef);

const progressPercent = computed(() => {
	if (!props.duration) return 0;
	return (props.currentTime / props.duration) * 100;
});

const startDragging = () => {
	dragging.value = true;
};

const stopDragging = () => {
	dragging.value = false;
};

watch([dragging, elementX], () => {
	if (!props.duration) return;
	if (!dragging.value) return;
	const progress = Math.max(0, Math.min(1, elementX.value / elementWidth.value));
	const time = progress * props.duration;
	emit('update:currentTime', time);
});

useEventListener(window, 'mouseup', stopDragging, { passive: true });
useEventListener(window, 'touchend', stopDragging, { passive: true });
useEventListener(window, 'touchcancel', stopDragging, { passive: true });
</script>

<template>
	<div class="player-controls-progress-wrapper">
		<div
			ref="progressRef"
			class="player-controls-progress"
			@mousedown="startDragging"
			@touchstart.passive="startDragging">
			<div class="player-controls-progress-schedule" :style="{ '--progress': `${progressPercent}%` }">
				<div class="player-controls-progress-thumb"></div>
			</div>
		</div>
	</div>
</template>

<style lang="scss" scoped>
.player-controls-progress-wrapper {
	padding: 6px 0;
	.player-controls-progress {
		position: relative;
		width: 100%;
		height: 4px;
		background: rgba(255, 255, 255, 0.3);
		cursor: pointer;

		.player-controls-progress-schedule {
			position: absolute;
			width: var(--progress, 0%);
			max-width: 100%;
			height: 100%;
			background: rgba(255, 255, 255, 0.7);
			.player-controls-progress-thumb {
				position: absolute;
				top: 50%;
				right: 0;
				transform: translate(50%, -50%);
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
}
</style>
