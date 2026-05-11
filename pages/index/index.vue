<template>
	<view class="home">
		<!-- #ifdef MP-WEIXIN ||H5 -->
		<search-solt>
			<template #leftbar>
				<view class="left-bar">
					<view class="iconfont icon-xiaoxi"></view>
				</view>
			</template>
			<template #centerbar>
				<view class="center-bar flex-1">
					<view class="center-bar flex-1" @click="getSearch()">
						请输入搜索内容
					</view>
				</view>
			</template>
			<template #rightbar>
				<!-- #ifdef MP -->
				<view class="right-bar" @click="getCode">
					<view class="iconfont icon-saoma">
					</view>
				</view>
				<!-- #endif -->
				<!-- #ifdef H5 -->
				<view class="right-bar">
					<view class="h-login">
						登录
					</view>
				</view>
				<!-- #endif -->
			</template>
		</search-solt>
		<!-- #endif -->
		<view class="a-address">
			{{address}}
		</view>
		<scroll-view scroll-x="true" :scroll-into-view="scrollinto" scroll-with-animation>
			<view class="d-flex">
				<view v-for="(item,i) in navbars" :key="i" :id="'tab'+i" class="flex-shrink mx-2"
					:class="currentIndex==i?'active':''" @click="changeTabs(i)">
					{{item.name}}
				</view>
			</view>
		</scroll-view>
		<swiper :style="'height:'+scrollH+'px'" :current="currentIndex" @change="changeIndex">
			<!-- 这里是最外层的循环，这里是左右滑动切换的部分 -->
			<swiper-item v-for="(news,index) in newsItems" :key="index">
				<!-- 这里就是判断 newsItems里面有没有数据的地方-->
				<template v-if="news.list.length>0">
					<view class="swiper-item">
						<!-- <scroll-view scroll-y="true" :style="'height:'+scrollH+'px'">
							<view v-for="i in 100" :key="i">{{i}}</view>
						</scroll-view> -->
						<scroll-view scroll-y="true" :style="'height:'+scrollH+'px'"
							@scrolltolower="scrollToLower(index)">
							<!-- block标签不会在屏幕上显示任何实体 适合配合循环、判断 主要就是将几行代码捆绑一起执行-->
							<block v-for="(item,n) in news.list" :key="n">
								<!-- 这里循环是切换具体内容的部分 例如首页的轮播图-->
								<template v-if="item.type=='swiper'">
									<swiper-banner :swipers="item.data"></swiper-banner>
								</template>
								<!-- 金刚区 -->
								<template v-if="item.type=='indexnavs'">
									<fast-nav :pagges="pagges"></fast-nav>
								</template>
								<!--三图区  -->
								<template v-else-if="item.type=='threeAdv'">
									<three-adv :news="item.data"></three-adv>
								</template>
								<!-- 图文区 -->
								<template v-else-if="item.type=='oneAdv'">
									<one-Adv :title="item.data.title" :cover="item.data.cover"></one-Adv>
								</template>
								<!-- 商品列表 -->
								<template v-else-if="item.type=='list'">
									<common-list :list="item.data"></common-list>
								</template>
							</block>
							<view class="l-loadmore text-center my-3">
								{{news.loadmore}}
							</view>
						</scroll-view>
					</view>
				</template>
				<template v-else-if="news.forward=='before'">
					<view class="text-center text-muted my-2">
						<image src="/static/loading.gif" mode="" style="width: 80rpx;height: 80rpx;"></image>
					</view>
				</template>
				<!-- 这里是占位用的数据 -->
				<template v-else>
					<view class="text-center text-muted my-2">
						暂无数据
					</view>
				</template>
			</swiper-item>
		</swiper>
		<sk-index v-if="isShow"></sk-index>
	</view>
</template>

<script setup>
	import {
		computed,
		onMounted,
		ref
	} from 'vue';
	import {
		onLoad,
		onNavigationBarSearchInputClicked,
		onNavigationBarButtonTap
	} from '@dcloudio/uni-app'
	import {
		getHome,
		getPage
	} from '../../api/index'
	var QQMapWX = require('../../static/qqmap-wx-jssdk.min.js');
	let title = ref('Hello')
	let navbars = ref([])
	let scrollinto = ref('')
	let currentIndex = ref(0)
	let newsItems = ref([])
	let scrollH = ref(0)
	let qqmapsdk = ""
	let address = ref("")
	let isShow = ref(true)
	const getSearch = () => {
		uni.navigateTo({
			url: "/pages/search/search"
		})
	}

	const gethome = () => {
		getHome().then(res => {
			// console.log(res)
			// 记得这里要.value,才能拿到响应式数组，在模板中自动解包不用加，这是vue3的规定，所以也可以使用reactive定义数组对象，不加value
			navbars.value = res.data.category
			for (var i = 0; i < navbars.value.length; i++) {
				// 注意这里要占位，就是循环几次我就有几个空的list数组，确保Tab和swiper数组长度一致
				let forward = i == 0 ? 'after' : 'before'
				let obj = {
					list: [],
					loadmore: '加载更多…',
					forward: forward
				}
				if (i == 0) {
					obj.list = res.data.data
				}

				newsItems.value.push(obj)
				// console.log('1222', newsItems.value);
			}
			isShow.value = false
		})
	}
	const getpage = (callback) => {
		let id = navbars.value[currentIndex.value].id
		let obj = newsItems.value[currentIndex.value]
		let page = Math.ceil(obj.list.length / 5) + 1
		obj.forward = 'before'
		getPage(id, page).then(res => {
			let data = res.data
			obj.list = [...obj.list, ...data]
			obj.loadmore = data.length < 5 ? '没有更多了…' : '加载更多…'
			obj.forward = 'after'
		})
		if (typeof callback == 'function') {
			callback()
		}
	}
	const changeTabs = (index) => {
		currentIndex.value = index;
		scrollinto.value = 'tab' + index
		if (currentIndex.value < 5) {
			scrollinto.value = 'tab0'
		} else {
			scrollinto.value = 'tab' + (index - 5)
		}
	}
	const changeIndex = (e) => {
		console.log('eeeee', e);
		currentIndex.value = e.detail.current
		let obj = newsItems.value[currentIndex.value]
		if (obj.forward == 'before') {
			getpage()
		} else {
			return;
		}

		if (currentIndex.value < 5) {
			scrollinto.value = 'tab0'
		} else {
			scrollinto.value = 'tab' + (e.detail.current - 5)
		}
	}
	const pagges = computed(() => {
		let pagge = []
		let index = currentIndex.value
		if (newsItems.value[index].list) {
			// console.log( newsItems.value[index].list);
			let obj = newsItems.value[index].list
			obj.forEach(v => {
				if (v.type == 'indexnavs') {
					let indexnav = Object.values(v.data)
					// console.log('indexnav', indexnav);
					indexnav.forEach((item, index) => {
						let page = Math.floor(index / 8)
						if (!pagge[page]) {
							pagge[page] = []
						}
						pagge[page].push(item)
					})
				}
			})
		}
		console.log('pagge', pagge);
		return pagge;
	})


	onLoad(() => {
		qqmapsdk = new QQMapWX({
			key: "OMPBZ-CSDCB-GNSUV-N4XD7-MLOCF-DSBHT"
		})
	})
	// 手机扫码
	const getCode = () => {
		uni.scanCode({
			success: function(res) {
				console.log('条码类型：' + res.scanType);
				console.log('条码内容：' + res.result);
			}
		})
	}

	onMounted(() => {
		gethome()
		uni.getSystemInfo({
			success: (res) => {
				scrollH.value = (res.windowHeight - uni.upx2px(264))
				console.log('qqqq', uni.upx2px(264));
			}
		})
		// 定位
		uni.getLocation({
			type: 'gcj02',
			success: (res) => {
				// console.log('当前位置的经度：' + res.longitude);
				// console.log('当前位置的纬度：' + res.latitude);
				qqmapsdk.reverseGeocoder({
					location: {
						latitude: res.latitude,
						longitude: res.longitude
					},
					success: (res2) => {
						console.log('res2', res2);
						address.value = res2.result.address
					},
					fail: (error) => {
						console.log('error', error);
					}
				})
			},
			fail: (err) => {
				console.log('err', err);
			}
		});
	})
	// 上拉加载
	const scrollToLower = (index) => {
		let obj = newsItems.value[index]
		obj.loadmore = '加载中…'
		//加载中…  没有更多了…
		if (obj.loadmore != '加载中…') {
			return;
		} else {
			setTimeout(() => {
				getpage(() => {
					uni.showToast({
						title: '加载成功'
					})
				})
			}, 2000)
		}

	}
	onNavigationBarSearchInputClicked(() => {
		uni.navigateTo({
			url: "/pages/search/search"
		})
	});
	onNavigationBarButtonTap((e) => {
		console.log(e)
		if (e.index == 0) {
			console.log("获取消息")
		}
	})
</script>

<style lang="scss">
	.left-bar,
	.right-bar {
		width: 44rpx;
		height: 44rpx;
		// background-color: green;
		text-align: center;
		line-height: 44rpx;
	}

	.center-bar {
		height: 60rpx;
		background-color: #f4f4f4;
		text-indent: 20rpx;
		line-height: 60rpx;
		color: #ccc;
	}

	.h-login {
		font-size: 25rpx;
		white-space: nowrap;
	}

	.a-address {
		width: 100%;
		height: 88rpx;
		line-height: 88rpx;
		background-color: rgba(255, 0, 0, 0.1);
	}

	.fastnav-img {
		width: 88rpx;
		height: 88rpx;
	}
</style>