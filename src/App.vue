<script setup lang="ts">
import { ref } from 'vue';
import SimplePlayer from './components/SimpleVideo/index.vue';
import VideoSelector from './components/SimpleVideo/selector.vue';

const video = ref<{ name: string; url: string } | null>(null);

const handleSelect = (value: any) => {
	video.value = value;
};
</script>

<template>
	<div class="main-container">
		<h1>Simple Player</h1>
		<simple-player v-if="video" :src="video.url" />
		<video-selector v-else @select-video="handleSelect" />
	</div>
</template>

<style scoped lang="scss">
.main-container {
	width: 1200px;
	max-width: 100%;
	margin: 0 auto;
    padding: 0 20px;

	h1 {
		padding: 20px 0;
		color: #ffffff;
	}
}
</style>
