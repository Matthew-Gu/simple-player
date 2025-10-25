<template>
	<div class="player-ctrl-btn" role="button">
		<slot />
	</div>
</template>

<style lang="scss" scoped>
.player-ctrl-btn {
	fill: #ffffff;
	color: #ffffff;
	width: 32px;
	height: 22px;
	font-size: 12px;
	line-height: 22px;
	text-align: center;
}

:deep(.player-svg-icon) {
	cursor: pointer;
	width: 100%;
	height: 22px;
	vertical-align: middle;
}
</style>
