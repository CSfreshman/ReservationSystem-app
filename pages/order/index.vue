<template>
	<view class="content">
		<!-- 顶部选择栏 -->
		<view class="tab-ul">
			<view class="pr tab">
				<view class="tab-li" :class="[activeIndex == index ? 'active' : '']" v-for="(item,index) in tabUl" :key="index" @tap="tabChange(index)">
					<text :id="item.id">{{item.name}}</text>
				</view>
				<view class="line" :style="{width: lineWidth,left: lineLeft}"></view>
			</view>
        </view>
		
		<view class="tab-content">
			<!-- <scroll-view v-show="order == ''" :scroll-top="scrollTop" @scroll="scroll" :style="'height:'+height+'px'"  refresher-enabled="true" :refresher-triggered="triggered" :refresher-threshold="100" refresher-background="lightblue" @refresherpulling="onPulling"
				@refresherrefresh="onRefresh" @refresherrestore="onRestore" @refresherabort="onAbort">
				<image src="../../static/nodata.png" class="nodata-img" mode="aspectFit"></image>
				<view class="state">您还没有订单哦</view>
				<button class="nodata-btn" hover-class="btn-hover" @tap="goMenu">去预约</button>
			</scroll-view> -->
			<view class="nodata" v-show="allOrder[activeIndex] == ''">
				<image src="../../static/nodata.png" class="nodata-img" mode="aspectFit"></image>
				<view class="state">您还没有订单哦</view>
				<button class="nodata-btn" hover-class="btn-hover" @tap="goMenu">去预约</button>
				<button class="fresh-btn" hover-class="btn-hover" @tap="refresh()">点击刷新</button>
			</view>
			
			<view class="page-body" v-show="allOrder[activeIndex] != ''">
				
				<scroll-view scroll-y :scroll-top="scrollTop" @scroll="scroll" :style="'height:'+height+'px'"  refresher-enabled="true" :refresher-triggered="triggered" :refresher-threshold="100" refresher-background="lightblue" @refresherpulling="onPulling"
					@refresherrefresh="onRefresh" @refresherrestore="onRestore" @refresherabort="onAbort">
					
					<view v-for="(orderItem,i) in allOrder[activeIndex]" :key="i">
						<uni-card @click="orderDetail(orderItem)">
							<uni-section class="mb-10" :title="orderItem.venueDesc" :sub-title="orderItem.date+'- - -'+orderItem.startTime+'--'+orderItem.endTime">
								<template v-slot:right>
									<uni-tag v-show="orderItem.state == 1" inverted="true" text="已完成" type="success" />
									<uni-tag v-show="orderItem.state == 2" inverted="true" text="待出行" type="primary" />
									<uni-tag v-show="orderItem.state == 3" inverted="true" text="已取消" type="warning" />
								</template>
							</uni-section>
						</uni-card>
						
					</view>
				</scroll-view>
			</view>
		</view>
	</view>
</template>

<script>
	import orderData from "./orderData.json"
	import URL from "../../api/api.js"
	
	export default {
		data() {
			return {
				tabUl: [{
					name: "全部",
					id: "all"
				},{
					name: "已完成",
					id: "completed"
				},{
					name: "待出行",
					id: "wait"
				},{
					name: "已取消",
					id: "cancel"
				}],
				order: "",
				allOrder: "",
				completed: "",
				activeIndex: 0,
				lineWidth: "56upx",
				lineLeft: "89upx",
				scrollTop: "0",
				height: "100upx",
				triggered: false,
				refreshData: ''
			}
		},
		onLoad() {
			
			uni.request({
				url: URL.testUrl + "/order/getAll1",
				method: "POST",
				data:{},
				header:{
					"token": uni.getStorageSync("token")
				},
				success: (res) => {
					console.log(res)
					if(res.data.code === 200){
						console.log("查询成功")
						this.allOrder = res.data.data
						this.order = this.allOrder[this.activeIndex]
					}else{
						// 增加一个错误提示
						console.log(res.data.msg)
						uni.showToast({
							title: res.data.msg,
							icon: 'none'
						})
					}
				}
			});
			
			
			this.height = Math.floor(uni.getSystemInfoSync().windowHeight-50);
			
			this._freshing = false;
			setTimeout(() => {
				this.triggered = true;
			}, 2000)
			
		},
		methods: {
			async tabChange(index) {
				this.activeIndex = index;
				if(index == 0){
					console.log("选择0")

				}else if(index == 1){
					console.log("选择1")

				}else if(index == 2){
					console.log("选择2")

				}else if(index == 3){
					console.log("选择3")

				}
				
				this.order = this.allOrder[this.activeIndex]

				var w = await this.getElSize(this.tabUl[index].id);
				this.lineLeft = w.left + "px";
				this.lineWidth = w.width + "px";
			},
			getElSize(id) { //得到元素的size
				return new Promise((res, rej) => {
					uni.createSelectorQuery().select("#" + id).fields({
						size: true,
						scrollOffset: true,
						rect: true
					}, (data) => {
						res(data);
					}).exec();
				})
			},
			goMenu(){
				uni.switchTab({
					url: '/pages/menu/index'
				})
			},
			orderDetail(orderItem){
				uni.navigateTo({
					url: "../order/order-detail?orderId=" + orderItem.id
				})
			},
			scroll(){},
			
			onPulling(e) {
				console.log("onpulling", e);
			
			},
			onRefresh() {
				if (this._freshing) return;
				this._freshing = true;
				setTimeout(() => {
					this.triggered = false;
					this._freshing = false;
				}, 2000)
				
				uni.request({
					url: URL.testUrl + "/order/getAll1",
					method: "POST",
					data: {
						"state" : this.activeIndex
					},
					header:{
						"token": uni.getStorageSync("token")
					},
					success: (res) => {
						console.log(res)
						if(res.data.code === 200){
							console.log("查询成功")
							this.refreshData = res.data.data
						}else{
							// 增加一个错误提示
							console.log(res.data.msg)
							uni.showToast({
								title: res.data.msg,
								icon: 'none'
							})
						}
					}
				});
				
			},
			
			// 获得指定状态的所有订单的方法
			getOrder(state){
				uni.request({
					url: URL.testUrl + "/order/getAll1",
					method: "POST",
					data: {
						"state": state
					},
					header:{
						"token": uni.getStorageSync("token")
					},
					success: (res) => {
						console.log(res)
						if(res.data.code === 200){
							console.log("查询成功")
							this.allOrder = res.data.data
						}else{
							// 增加一个错误提示
							console.log(res.data.msg)
							uni.showToast({
								title: res.data.msg,
								icon: 'none'
							})
						}
					}
				});
			},
			
			onRestore() {
				this.triggered = 'restore'; // 需要重置
				console.log(URL.testUrl);
				
				this.allOrder = this.refreshData
				this.order = this.allOrder[this.activeIndex]
				
			},
			onAbort() {
				console.log("onAbort");
			
			},
			refresh(){
				this.getOrder(0)
			}
		}
	}
</script>

<style>
	.content{height: 100%;text-align: center;}
	.tab{display: flex;}
	.tab-ul{width: 100%;height: 86upx;border-top: 2upx solid #e7e7e7;background-color: #fff;color: #444;position: fixed;top: 0;left: 0;}
	// #ifdef H5
	.tab-ul{top: 44px;}
	// #endif
	.line{width: 56upx;height: 6upx;background-color: #7da7d2;position: absolute;bottom: 0;left: 89upx;transition: .2s;}
	.tab-li{flex-grow: 1;line-height: 86upx;}
	.tab-li.active{color: #74a1cf;}
	.tab-content{height: 100%;padding-top: 86upx;box-sizing: border-box;}
	.state{margin: 108upx 0 88upx 0;color: #999;}
	.nodata{padding-top: 230upx;}
	.nodata-img{width: 160upx;height: 144upx;}
	.nodata-btn{border: 1px solid #6999cb;width: 250upx;height: 66upx;color: #6999cb;line-height: 66upx;font-size: 28upx;background: #fff;}
	.fresh-btn{border: 1px solid #6999cb;width: 250upx;height: 66upx;color: #6999cb;line-height: 66upx;font-size: 28upx;background: #fff; margin-top: 50upx;}
	.btn-hover{background: #fff;}
	.nodata-btn:after{border: none;}
</style>
