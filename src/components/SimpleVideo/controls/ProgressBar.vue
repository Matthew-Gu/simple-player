<script setup lang="ts">
import { ref, computed, watch, onMounted, onUnmounted } from 'vue';
import { useMouseInElement, useEventListener, useDebounceFn } from '@vueuse/core';

const props = defineProps<{
	currentTime: number;
	duration: number;
	videoRef: HTMLVideoElement | null;
}>();

const emit = defineEmits<{
	(e: 'update:currentTime', value: number): void;
}>();

const progressRef = ref<HTMLDivElement>();
const previewRef = ref<HTMLDivElement>();
const dragging = ref(false);
const previewX = ref(0);
const previewTime = ref(0);
const previewImg = ref('');
const isVideoReady = ref(false);

const hiddenVideo = document.createElement('video');
hiddenVideo.crossOrigin = 'anonymous';
hiddenVideo.muted = true;
hiddenVideo.playsInline = true;
hiddenVideo.style.cssText = `
	width: 1px;
	height: 1px;
	position: absolute;
	left: -9999px;
	top: -9999px;
	pointer-events: none;
    opacity: 0;
`;

const previewCanvas = document.createElement('canvas');
previewCanvas.width = 160;
previewCanvas.height = 90;
const previewCtx = previewCanvas.getContext('2d')!;

const previewCache = new Map<number, string>();

onMounted(() => {
	document.body.appendChild(hiddenVideo);
});

onUnmounted(() => {
	if (hiddenVideo.parentNode) {
		hiddenVideo.parentNode.removeChild(hiddenVideo);
	}
	previewCache.clear();
});

const { elementX, elementWidth, isOutside } = useMouseInElement(progressRef);

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
	if (!props.duration || elementWidth.value <= 0) return;
	if (!dragging.value) return;
	const progress = Math.max(0, Math.min(1, elementX.value / elementWidth.value));
	const time = progress * props.duration;
	emit('update:currentTime', time);
});

const debouncedGeneratePreview = useDebounceFn((time: number) => {
	captureFrame(time);
}, 50);

watch(elementX, (x) => {
	if (!props.duration || elementWidth.value <= 0 || isOutside.value) return;
	const progress = Math.max(0, Math.min(1, x / elementWidth.value));
	const time = progress * props.duration;
	previewTime.value = time;
	previewX.value = x;

	const cached = previewCache.get(Math.floor(time * 10) / 10);
	if (cached) {
		previewImg.value = cached;
	} else {
		ensureHiddenVideoReady()
			.then(() => {
				debouncedGeneratePreview(time);
			})
			.catch(() => {
				previewImg.value = '';
			});
	}
});

const previewLeft = computed(() => {
	if (!progressRef.value || !previewRef.value) return '0px';
	const halfPreviewWidth = previewRef.value.offsetWidth / 2;
	let left = elementX.value - halfPreviewWidth;
	if (left < 0) left = 0;
	if (left + previewRef.value.offsetWidth > elementWidth.value)
		left = elementWidth.value - previewRef.value.offsetWidth;
	return `${left}px`;
});

useEventListener(window, 'mouseup', stopDragging, { passive: true });
useEventListener(window, 'touchend', stopDragging, { passive: true });
useEventListener(window, 'touchcancel', stopDragging, { passive: true });

const ensureHiddenVideoReady = async (): Promise<void> => {
	if (isVideoReady.value) return;

	const mainVideo = props.videoRef;
	if (!mainVideo || !mainVideo.currentSrc) {
		throw new Error('Main video not ready');
	}

	if (hiddenVideo.src !== mainVideo.currentSrc) {
		isVideoReady.value = false;
		previewCache.clear();
		hiddenVideo.src = mainVideo.currentSrc;
		hiddenVideo.load();
	}

	await new Promise<void>((resolve, reject) => {
		const onReady = () => {
			isVideoReady.value = true;
			hiddenVideo.removeEventListener('loadedmetadata', onReady);
			hiddenVideo.removeEventListener('error', onError);
			resolve();
		};
		const onError = () => {
			hiddenVideo.removeEventListener('loadedmetadata', onReady);
			hiddenVideo.removeEventListener('error', onError);
			reject(new Error('Failed to load hidden video'));
		};
		hiddenVideo.addEventListener('loadedmetadata', onReady, { once: true });
		hiddenVideo.addEventListener('error', onError, { once: true });
	});
};

const captureFrame = async (time: number) => {
	try {
		if (!isVideoReady.value) return;

		const cacheKey = Math.floor(time * 10) / 10;
		if (previewCache.has(cacheKey)) {
			previewImg.value = previewCache.get(cacheKey)!;
			return;
		}

		// Seek 到指定时间
		await new Promise<void>((resolve) => {
			const onSeeked = () => {
				hiddenVideo.removeEventListener('seeked', onSeeked);
				resolve();
			};
			hiddenVideo.addEventListener('seeked', onSeeked, { once: true });
			hiddenVideo.currentTime = Math.min(time, hiddenVideo.duration - 0.01);
		});

		if (hiddenVideo.readyState < 2) return;

		previewCtx.drawImage(hiddenVideo, 0, 0, 160, 90);
		const imgData = previewCanvas.toDataURL('image/jpeg', 0.75);
		previewCache.set(cacheKey, imgData);
		previewImg.value = imgData;
	} catch (err) {
		console.warn('captureFrame failed:', err);
		previewImg.value = '';
	}
};

const formatSeconds = (seconds: number = 0) => {
	const dur = props.duration ?? 0;
	const secs = Math.max(0, Math.floor(seconds));
	const showHours = dur >= 3600;
	const pad = (v: number) => v.toString().padStart(2, '0');
	const h = Math.floor(secs / 3600);
	const m = Math.floor((secs % 3600) / 60);
	const s = secs % 60;
	return showHours ? `${pad(h)}:${pad(m)}:${pad(s)}` : `${pad(m)}:${pad(s)}`;
};
</script>

<template>
	<div
		ref="progressRef"
		class="player-controls-progress-wrapper"
		@mousedown="startDragging"
		@touchstart.passive="startDragging">
		<div class="player-controls-progress">
			<div class="player-controls-progress-schedule" :style="{ '--progress': `${progressPercent}%` }">
				<div class="player-controls-progress-thumb"></div>
			</div>

			<transition name="fade">
				<div
					v-if="!isOutside"
					ref="previewRef"
					class="player-controls-progress-preview"
					:style="{ left: previewLeft }">
					<img v-if="previewImg" :src="previewImg" alt="Preview" />
					<span>{{ formatSeconds(previewTime) }}</span>
				</div>
			</transition>
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

.player-controls-progress-preview {
	position: absolute;
	width: 320px;
	bottom: 100%;
	transform: translateY(-8px);
	background: rgba(0, 0, 0, 0.6);
	border-radius: 4px;
	overflow: hidden;
	box-shadow: 0 2px 6px rgba(0, 0, 0, 0.3);
	pointer-events: none;
	display: flex;
	flex-direction: column;
	align-items: center;
}

.player-controls-progress-preview img {
	display: block;
	width: 320px;
	height: 180px;
	object-fit: cover;
	border-radius: 4px;
	box-shadow: 0 2px 8px rgba(0, 0, 0, 0.4);
}

.player-controls-progress-preview span {
	padding: 4px 0;
	font-size: 14px;
	color: #fff;
	text-shadow: 0 0 2px rgba(0, 0, 0, 0.8);
}

.fade-enter-active,
.fade-leave-active {
	transition: opacity 0.2s;
}
.fade-enter-from,
.fade-leave-to {
	opacity: 0;
}
</style>
