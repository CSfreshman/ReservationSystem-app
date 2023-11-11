<template>
	<view>
		<view>订单号 = {{orderId}}</view>
		
		<uni-section title="订单状态" type="line" padding="">
			<template>
				<uni-tag v-show="orderDetail.state == 1" inverted="true" text="已完成" type="success" />
				<uni-tag v-show="orderDetail.state == 2" inverted="true" text="待出行" type="primary" />
				<uni-tag v-show="orderDetail.state == 3" inverted="true" text="已取消" type="warning" />
			</template>
		</uni-section>
		<uni-section title="场馆" type="line" padding="">
			{{orderDetail.venueDesc}}
		</uni-section>
		<uni-section title="日期" type="line" padding="">
			{{orderDetail.date}}
		</uni-section>
		<uni-section title="时段" type="line" padding="">
			{{orderDetail.startTime}} -- {{orderDetail.endTime}}
		</uni-section>
		<uni-section title="取消原因" v-show="orderDetail.state == 3" type="line" padding="">
			{{orderDetail.cancelReason}}
		</uni-section>
		<uni-card title="操作" v-show="orderDetail.state == 2">
			<view class="btn-group">
				<button class="btn first-btn" @click="cancelOrder()">取消订单</button>
			</view>
		</uni-card>
		<uni-card title="订单二维码">
			<image :src="orderDetail.qrCodeUrl"></image>
		</uni-card>
		
		
		
	</view>
</template>

<script>
	import URL from "../../api/api.js"
	export default {
		data() {
			return {
				orderId: '',
				orderDetail: ""
			}
			
		},
		onLoad(option) {
			console.log(option)
			this.orderId = option.orderId;
			console.log(this.orderId)
			uni.request({
			    url: URL.testUrl + '/order/getById', //仅为示例，并非真实接口地址。
			    data: {
					"id": this.orderId 
			    },
				header:{
					"token": uni.getStorageSync("token")
				},
				method: "POST",
			    success: (res) => {
					console.log(res)
					if(res.data.code === 200){
						console.log("查询成功")
						this.orderDetail = res.data.data
					}else{
						// 增加一个错误提示
						console.log(res.data.msg)
					}
			        
			    }
			});
		},
		
		methods: {
			cancelOrder(){
				console.log("要取消的订单：" + this.orderId);
				uni.request({
					url: URL.testUrl + "/order/cancelOrder",
					method: "POST",
					data:{
						"id" : this.orderId
					},
					header:{
						"token": uni.getStorageSync("token")
					},
					success: (res) => {
						console.log(res)
						if(res.data.code === 200){
							console.log("操作成功")
							uni.showToast({
								title: "操作成功",
								icon: null
							})
							uni.request({
							    url: URL.testUrl + '/order/getById', //仅为示例，并非真实接口地址。
							    data: {
									"id": this.orderId 
							    },
								header:{
									"token": uni.getStorageSync("token")
								},
								method: "POST",
							    success: (res) => {
									console.log(res)
									if(res.data.code === 200){
										console.log("查询成功")
										this.orderDetail = res.data.data
									}else{
										// 增加一个错误提示
										console.log(res.data.msg)
									}
							        
							    }
							});
						}else{
							// 增加一个错误提示
							console.log(res.data.msg)
							uni.showToast({
								title: res.data.msg,
								icon: null
							})
						}
					}
				})
			}
		}
	}
</script>

<style>
</style>