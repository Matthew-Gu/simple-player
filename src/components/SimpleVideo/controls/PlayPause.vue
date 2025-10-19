<script setup lang="ts">
import CtrlBtn from './CtrlBtn.vue';
defineProps<{ playing: boolean }>();

const emit = defineEmits<{
	(e: 'toggle-play'): void;
}>();
</script>

<template>
	<ctrl-btn class="play-ctrl-play">
		<div class="player-svg-icon" @click="emit('toggle-play')">
			<svg
				v-show="!playing"
				xmlns="http://www.w3.org/2000/svg"
				viewBox="0 0 24 24"
				fill="currentColor"
				class="icon-tabler">
				<path stroke="none" d="M0 0h24v24H0z" fill="none" />
				<path d="M6 4v16a1 1 0 0 0 1.524 .852l13 -8a1 1 0 0 0 0 -1.704l-13 -8a1 1 0 0 0 -1.524 .852z" />
			</svg>
			<svg
				v-show="playing"
				xmlns="http://www.w3.org/2000/svg"
				viewBox="0 0 24 24"
				fill="currentColor"
				class="icon-tabler">
				<path stroke="none" d="M0 0h24v24H0z" fill="none" />
				<path d="M9 4h-2a2 2 0 0 0 -2 2v12a2 2 0 0 0 2 2h2a2 2 0 0 0 2 -2v-12a2 2 0 0 0 -2 -2z" />
				<path d="M17 4h-2a2 2 0 0 0 -2 2v12a2 2 0 0 0 2 2h2a2 2 0 0 0 2 -2v-12a2 2 0 0 0 -2 -2z" />
			</svg>
		</div>
	</ctrl-btn>
</template>
