<template>
	<view>
		<view class="nav-bar d-flex a-center px">
			<slot name="leftbar"></slot>
			<slot name="centerbar"></slot>
			<slot name="rightbar"></slot>
		</view>
	</view>
</template>

<script setup>

</script>

<style lang="scss">
	.nav-bar {
		width: 100%;
		height: 80rpx;
		// background-color: #f60;
		box-sizing: border-box;
	}
</style>