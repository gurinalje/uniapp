<template>
	<view class="">
		111
	</view>
	<view>
		<uni-easyinput prefixIcon="search" v-model="val" placeholder="测试" />
	</view>
</template>
<script setup>
	import {
		ref
	} from 'vue'
	const val = ref('')
</script>

<style>

</style>