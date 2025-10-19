<script setup lang="ts">
import { ref } from 'vue';
import { useFileDialog, useDropZone } from '@vueuse/core';

interface MyVideo {
	name: string;
	url: string;
}

const emit = defineEmits<{
	(e: 'select-video', video: MyVideo): void;
}>();

const currentVideo = ref<MyVideo | null>(null);

const cleanupObjectUrl = () => {
	if (currentVideo.value?.url) {
		URL.revokeObjectURL(currentVideo.value.url);
		currentVideo.value = null;
	}
};

// 处理文件选择（点击上传）
const { open: openFilePicker, onChange: onFileChange } = useFileDialog({
	accept: 'video/*',
	multiple: false
});

onFileChange((files: FileList | null) => {
	cleanupObjectUrl(); // 清理之前的 URL
	if (!files?.length) return;

	const file = files[0]!;
	const url = URL.createObjectURL(file);
	currentVideo.value = {
		name: file.name,
		url
	};

	emit('select-video', currentVideo.value);
});

const dropZoneRef = ref<HTMLElement | null>(null);

const onDrop = (files: File[] | null) => {
	cleanupObjectUrl();
	if (!files?.length) return;

	const file = files[0]!;
	if (!file!.type.startsWith('video/')) {
		console.warn('Only video files are allowed.');
		return;
	}

	const url = URL.createObjectURL(file);
	currentVideo.value = {
		name: file.name,
		url
	};

	emit('select-video', currentVideo.value);
};

const { isOverDropZone } = useDropZone(dropZoneRef, {
	onDrop
});

const handleSelectClick = () => {
	openFilePicker();
};
</script>

<template>
	<div ref="dropZoneRef" :class="{ 'drag-active': isOverDropZone }" class="drop-zone" @click="handleSelectClick">
		<p>拖拽视频文件到此处，或点击选择</p>
	</div>
</template>

<style lang="scss" scoped>
.drop-zone {
	border: 2px dashed #ccc;
	border-radius: 8px;
	padding: 40px;
    font-size: 20px;
    color: #ffffff;
	text-align: center;
	cursor: pointer;
	transition: border-color 0.3s;
}

.drop-zone.drag-active {
	border-color: #409eff;
	background-color: rgba(255,255,255,.2);
}
</style>
