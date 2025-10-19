<script setup lang="ts">
import CtrlBtn from './CtrlBtn.vue';
const emit = defineEmits<{
	(e: 'toggle-full'): void;
}>();
</script>

<template>
	<ctrl-btn class="player-ctrl-full">
		<div class="player-svg-icon" @click="emit('toggle-full')">
			<svg
				xmlns="http://www.w3.org/2000/svg"
				viewBox="0 0 24 24"
				fill="none"
				stroke="currentColor"
				stroke-width="2"
				stroke-linecap="round"
				stroke-linejoin="round"
				class="icon-tabler">
				<path stroke="none" d="M0 0h24v24H0z" fill="none" />
				<path d="M4 8v-2a2 2 0 0 1 2 -2h2" />
				<path d="M4 16v2a2 2 0 0 0 2 2h2" />
				<path d="M16 4h2a2 2 0 0 1 2 2v2" />
				<path d="M16 20h2a2 2 0 0 0 2 -2v-2" />
			</svg>
		</div>
	</ctrl-btn>
</template>
