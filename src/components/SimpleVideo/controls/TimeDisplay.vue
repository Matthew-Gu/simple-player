<script setup lang="ts">
import CtrlBtn from './CtrlBtn.vue';
const props = defineProps<{
	currentTime: number;
	duration: number;
}>();

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
	<ctrl-btn class="player-ctrl-time">
		<div class="player-ctrl-time-label">
			<span class="player-ctrl-time-current">{{ formatSeconds(currentTime) }}</span>
			<span class="player-ctrl-time-divide">/</span>
			<span class="player-ctrl-time-duration">{{ formatSeconds(duration) }}</span>
		</div>
	</ctrl-btn>
</template>

<style lang="scss" scoped>
.player-ctrl-time {
	min-width: 90px;
}
</style>
