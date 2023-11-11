<template>
	<view class="content">
		<view class="banner">
			<swiper :autoplay="autoplay" :indicator-dots="indicatorDots" :interval="interval" :circular="circular" class="swiper-box">
				<swiper-item v-for="(item,index) in banner" :key="index">
					<image :src="item" class="banner-image"></image>
				</swiper-item>
			</swiper>
			<!-- <view class="scan" @tap="scanCode">
				<uni-icon type="scan"></uni-icon>
			</view> -->
		</view>
		<!-- <view class="store">
			<view class="store-text">金泰开阳大厦店</view>
			<view class="store-distance">距您761m</view>
			<view class="store-off">
				<view class="store-btn" :class="[storeOff ? 'active' : '']" @tap="store()">自提</view>
				<view class="store-btn" :class="[!storeOff ? 'active' : '']" @tap="store()">外卖</view>
			</view>
		</view> -->
		<view class="list">
			<view class="list-cell" v-for="(item,index) in listObj" :key="index" @tap="go(item.url)">
				<view class="list-text">{{item.liText}} <view class="list-hint" v-show="item.hint">{{item.hint}}</view></view>
				<view class="list-en">{{item.liEn}}</view>
				<view class="list-right">
					<image :src="item.imgurl" class="list-img" mode="aspectFit"></image>
				</view>
			</view>
		</view>
	</view>
</template>

<script>
	import uniIcon from "@/components/uni-icon.vue"
	export default {
		components: {
			uniIcon
		},
		data() {
			return {
				banner: [
					// 乒乓室内
					"https://gulimall-imagewzw.oss-cn-hangzhou.aliyuncs.com/%E4%B9%92%E4%B9%93%E7%90%83%E5%AE%A4%E5%86%85.jpg",
					// 篮球室内
					"https://gulimall-imagewzw.oss-cn-hangzhou.aliyuncs.com/%E7%AF%AE%E7%90%83%E5%AE%A4%E5%86%85.jpg",
					// 羽毛球室内
					"https://gulimall-imagewzw.oss-cn-hangzhou.aliyuncs.com/%E7%BE%BD%E6%AF%9B%E7%90%83%E5%AE%A4%E5%86%85.jpg",
				],
				indicatorDots: true,
				autoplay: true,
				circular: true,
				interval: 2000,
				duration: 500.,
				storeOff: true,
				listObj: [
					{
						"liText": "立即预约",
						"liEn": "ORDER NOW",
						"imgurl": "../../static/index/list/预约icon.png",
						"url": "/pages/menu/index"
					},
					{
						"liText": "室内篮球馆介绍",
						"liEn": "Indoor Basketball",
						"imgurl": "../../static/index/list/篮球icon.png",
						//"url": "/pages/mine/wallet/index"
					},
					{
						"liText": "室内乒乓球馆介绍",
						"liEn": "Indoor Table Tennis",
						"imgurl": "../../static/index/list/乒乓球icon.png",
						//"url": "/pages/home/presented"
					},
					{
						"liText": "室内羽毛球馆介绍",
						"liEn": "Indoor Badminton",
						"imgurl": "../../static/index/list/羽毛球icon.png",
						"url": ""
					}
				]
			}
		},
		onLoad() {

		},
		methods: {
			store() {
				this.storeOff = !this.storeOff;
			},
			scanCode: function () {
				var that = this
				uni.scanCode({
					success: function (res) {
						that.result = res.result
					},
					fail: function (res) {}
				})
			},
			go(url){
				if(url == '/pages/menu/index'){
					uni.switchTab({
						url: url
					})
				}else{
					uni.navigateTo({
						url: url
					})
				}
			}
		}
	}
</script>

<style>
	.swiper-box{height: 421upx;}
	.banner{position: relative;}
	.banner-image{width: 100%;}
	.scan{position: absolute;top: 48upx;right: 30upx;width: 80upx;height: 80upx;color: #fff;background: rgba(0,0,0,0.6);line-height: 74upx;text-align: center;border-radius: 50%;padding-top: 10upx;box-sizing: border-box;}
	.store,.list{background: #fff;}
	.store{height: 136upx;padding: 36upx 40upx 0 40upx;box-sizing: border-box;position: relative;}
	.store-off{position: absolute;top: 50%;right: 40upx;width: 182upx;height: 66upx;line-height: 66upx;text-align: center;border: 1px solid #5d92c7;border-radius: 40upx;color: #5d92c7;margin-top: -33upx;padding: 1px;font-size: 24upx;}
	.store-btn{display: inline-block;width: 50%;line-height: 66upx;}
	.store-btn.active{color: #fff;background-color: #7ca7d2;border-radius: 40upx;}
	.store-text,.list-text{line-height: 28upx;font-size: 32upx;}
	.store-distance,.list-en{font-size: 18upx;color: #5e5e5e;line-height: 18upx;margin-top: 23upx;}
	.list{padding: 0 40upx;}
	.list-cell{height: 118upx;border-top: 1px solid #efefef;position: relative;}
	.list-text{margin-top: 27upx;display: inline-block;}
	.list-right{position: absolute;top: 50%;right: 50upx;width: 76upx;height: 76upx;line-height: 76upx;text-align: center;border-radius: 50%;border: 1px solid #c6baad;margin-top: -38upx;color: #784f2d;}
	.list-en{font-size: 16upx;}
	.list-img{width: 48upx;height: 48upx;margin-top: 12upx;}
	.list-hint{font-size: 16upx;color: #fff;background-color: #e06e11;padding: 0 4upx;border-radius: 3px;line-height: 24upx;display: inline-block;vertical-align: top;}
</style>
