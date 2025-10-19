<script setup lang="ts">
import CtrlBtn from './CtrlBtn.vue';

defineProps<{ rate: number }>();

const emit = defineEmits<{
	(e: 'update:rate', value: number): void;
}>();

const rateList = [
	{ label: '3.0x', value: 3 },
	{ label: '2.0x', value: 2 },
	{ label: '1.5x', value: 1.5 },
	{ label: '1.0x', value: 1 },
	{ label: '0.5x', value: 0.5 }
];

const selectRate = (value: number) => {
	emit('update:rate', value);
};
</script>

<template>
	<ctrl-btn class="player-ctrl-rate">
		<div class="player-svg-icon">
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
				<path d="M5.636 19.364a9 9 0 1 1 12.728 0" />
				<path d="M16 9l-4 4" />
			</svg>
		</div>
		<ul class="player-rate-box">
			<li
				class="rate-menu-item"
				:class="{ active: rate == item.value }"
				v-for="item in rateList"
				:key="item.value"
				@click="selectRate(item.value)">
				{{ item.label }}
			</li>
		</ul>
	</ctrl-btn>
</template>

<style lang="scss" scoped>
.player-ctrl-rate {
	position: relative;

	&:hover {
		.player-rate-box {
			display: block;
		}
	}

	.player-rate-box {
		display: none;
		position: absolute;
		bottom: 32px;
		left: 50%;
		transform: translateX(-50%);
		border-radius: 2px;
		background: rgba(0, 0, 0, 0.6);
		width: 60px;
        user-select: none;

		.rate-menu-item {
			font-size: 14px;
			height: 28px;
			line-height: 28px;
			text-align: center;
			cursor: pointer;
			color: #ffffff;

			&:hover {
				background: rgba(255, 255, 255, 0.1);
			}

			&.active {
				color: #000000;
				background: rgba(255, 255, 255, 0.6);
			}
		}

		&::after {
			position: absolute;
			content: '';
			width: 36px;
			height: 20px;
			bottom: 0;
			left: 50%;
			transform: translate(-50%, 100%);
		}
	}
}
</style>
