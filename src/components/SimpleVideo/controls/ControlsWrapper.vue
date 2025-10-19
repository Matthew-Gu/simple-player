<script setup lang="ts">
import { ref } from 'vue';
import { useEventListener } from '@vueuse/core';

defineProps<{ hide: boolean }>();

const emit = defineEmits<{
	(e: 'hover', isOver: boolean): void;
}>();

const controlsRef = ref<HTMLDivElement>();

// 处理鼠标进入/离开
const handleMouseEnter = () => {
	emit('hover', true);
};

const handleMouseLeave = () => {
	emit('hover', false);
};

useEventListener(controlsRef, 'mouseenter', handleMouseEnter, { passive: true });
useEventListener(controlsRef, 'mouseleave', handleMouseLeave, { passive: true });
</script>

<template>
	<div ref="controlsRef" class="player-controls-wrapper" :class="{ hide }">
		<div class="player-controls-top">
			<slot name="top" />
		</div>
		<div class="player-controls-bottom">
			<div class="player-controls-bottom-left">
				<slot name="bottom-left" />
			</div>
			<div class="player-controls-bottom-center">
				<slot name="bottom-center" />
			</div>
			<div class="player-controls-bottom-right">
				<slot name="bottom-right" />
			</div>
		</div>
	</div>
</template>

<style lang="scss" scoped>
.player-controls-wrapper {
	position: absolute;
	left: 0;
	bottom: 0;
	width: 100%;
	visibility: visible;
	opacity: 1;
	z-index: 10;
	background: linear-gradient(to bottom, rgba(0, 0, 0, 0), rgba(0, 0, 0, 0.6));

	.player-controls-top {
		padding: 0 12px;
		overflow: hidden;
	}

	.player-controls-bottom {
		display: flex;
		line-height: 22px;
		justify-content: space-between;
		padding: 0 12px 10px;

		.player-controls-bottom-left,
		.player-controls-bottom-center,
		.player-controls-bottom-right {
			display: flex;
		}
	}

	&.hide {
		visibility: hidden;
		opacity: 0;
		transition: visibility 0.6s, opacity 0.6s ease-out;
	}
}
</style>
