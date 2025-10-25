<script setup lang="ts">
import CtrlBtn from './CtrlBtn.vue';

defineProps<{ mirror: boolean; picInPic: boolean }>();

const emit = defineEmits<{
	(e: 'toggle-mirror'): void;
}>();

const toggleMirror = () => {
	emit('toggle-mirror');
};
</script>

<template>
	<ctrl-btn class="player-ctrl-setting">
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
				<path
					d="M10.325 4.317c.426 -1.756 2.924 -1.756 3.35 0a1.724 1.724 0 0 0 2.573 1.066c1.543 -.94 3.31 .826 2.37 2.37a1.724 1.724 0 0 0 1.065 2.572c1.756 .426 1.756 2.924 0 3.35a1.724 1.724 0 0 0 -1.066 2.573c.94 1.543 -.826 3.31 -2.37 2.37a1.724 1.724 0 0 0 -2.572 1.065c-.426 1.756 -2.924 1.756 -3.35 0a1.724 1.724 0 0 0 -2.573 -1.066c-1.543 .94 -3.31 -.826 -2.37 -2.37a1.724 1.724 0 0 0 -1.065 -2.572c-1.756 -.426 -1.756 -2.924 0 -3.35a1.724 1.724 0 0 0 1.066 -2.573c-.94 -1.543 .826 -3.31 2.37 -2.37c1 .608 2.296 .07 2.572 -1.065z" />
				<path d="M9 12a3 3 0 1 0 6 0a3 3 0 0 0 -6 0" />
			</svg>
		</div>
		<div class="player-setting-box">
			<div class="setting-menu-item" @click="toggleMirror">
				<div class="menu-label">视频镜像</div>
				<div class="menu-value">
					<div class="switch" :class="{ active: mirror }">
						<div class="slider"></div>
					</div>
				</div>
			</div>
		</div>
	</ctrl-btn>
</template>

<style lang="scss" scoped>
.player-ctrl-setting {
	position: relative;

	&:hover {
		.player-setting-box {
			display: block;
		}
	}

	.player-setting-box {
		display: none;
		position: absolute;
		bottom: 32px;
		left: 50%;
		transform: translateX(-50%);
		border-radius: 2px;
		background: rgba(0, 0, 0, 0.6);
		user-select: none;

		.setting-menu-item {
			height: 28px;
			font-size: 14px;
			line-height: 24px;
			text-align: center;
			color: #ffffff;
			display: flex;
			align-items: center;
			justify-content: space-between;
			padding: 0 6px;

			.menu-label {
				width: 64px;
			}
		}

		&::after {
			position: absolute;
			content: '';
			width: 50%;
			height: 20px;
			bottom: 0;
			left: 50%;
			transform: translate(-50%, 100%);
		}
	}
}

.switch {
	--button-w: 28px;
	--button-h: 16px;
	--slider-r: 7px;
	--slider-s: 4px;
	--slider-bg: #ffffff;
	--color-active: #66bb6a;
	--color-inactive: #cccccc;
	--duration: 0.3s;

	display: block;
	width: var(--button-w);
	height: var(--button-h);

	.slider {
		position: relative;
		width: 100%;
		height: 100%;
		border-radius: calc(var(--button-h) / 2);
		background-color: var(--color-inactive);
		transition: all var(--duration);
		cursor: pointer;

		&:before {
			position: absolute;
			top: calc((var(--button-h) - var(--slider-r) * 2) / 2);
			left: calc((var(--button-h) - var(--slider-r) * 2) / 2);
			transform: translateX(0%);
			content: '';
			display: block;
			width: calc(var(--slider-r) * 2);
			height: calc(var(--slider-r) * 2);
			border-radius: var(--slider-r);
			background-color: var(--slider-bg);
			transition: all var(--duration);
		}

		&:active:before {
			width: calc(var(--slider-r) * 2 + var(--slider-s));
		}
	}

	&.active {
		.slider {
			background-color: var(--color-active);

			&:before {
				left: calc(100% - var(--button-h) / 2 + var(--slider-r));
				transform: translateX(-100%);
			}
		}
	}
}
</style>
