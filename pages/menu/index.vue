<template>
	<view class="container">
		
		<view class="page-body">
			<!-- 左侧列表 -->
			<scroll-view class="nav-left" scroll-y :style="'height:'+height+'px'">
				<view class="nav-left-item" :class="index==categoryActive?'active':''" v-for="(item,index) in roomList" :key="index" @tap="categoryClick(index)">{{item.name}}</view>
			</scroll-view>
			
			
			
						
			<!-- 右侧列表 -->
			<scroll-view class="nav-right" scroll-y :scroll-top="scrollTop" @scroll="scroll" :style="'height:'+height+'px'" scroll-with-animation id="rightItem" refresher-enabled="true" :refresher-triggered="triggered" :refresher-threshold="100" refresher-background="lightblue" @refresherpulling="onPulling"
				@refresherrefresh="onRefresh" @refresherrestore="onRestore" @refresherabort="onAbort">
			
				<view class="nav-right-item">
					
					<uni-section class="mb-10" :title="itemObj.name" sub-title="可以提前5天进行预约" type="line"></uni-section>
					
					
					<view v-for="(dateData1,i) in itemObj.optionalDates" :key="i">
						<view><uni-section :title="dateData1.date" type="line" padding></uni-section></view>
						<uni-card v-if="dateData1.optionalTimes.length == 0">今日不可预约</uni-card>
						<view v-if="dateData1.optionalTimes.length != 0">
							<view v-for="(timeData,j) in dateData1.optionalTimes" :key="j" @tap="doOrder(timeData,i,j)">
								<uni-card>
									可预约时间：{{timeData.startTime}}--{{timeData.endTime}}</br>
									剩余可预约数：{{timeData.number}}
								</uni-card>
							</view>
						</view>
						
					</view>
					
					
					
				</view>
			</scroll-view>
		</view>
		
		<!-- <popup :show="showPopup" @hidePopup="hidePopup" @addTocart="addTocart" :popupData="popupData"></popup> -->
		<view v-show="showPopup">
			<view class="mask" v-show="showPopup" @tap="hidePopup"></view>
			<view class="popup" v-show="showPopup">
				<view class="popup-header">
				</view>
				<view class="popup-middle">
					<view class="describe">
						<uni-section :title="itemObj.name" type="line" padding></uni-section>
						<uni-section title="日期" type="line" padding>
							<text>{{itemObj.optionalDates[selectedDateIndex].date}}</text>
						</uni-section>
						<uni-section title="时段" type="line" padding>
							{{detail.startTime}} -- {{detail.endTime}}
						</uni-section>
						
						
					</view>
				</view>
				<!-- 弹出层底部 -->
				<view class="popup-footer">
					<view class="btn-group">
						<button class="btn first-btn" @tap="hidePopup()">我再想想</button>
						
						<button class="btn three-btn" @tap="submit()">确定预约</button>
					</view>
				</view>
			</view>
		</view>
	</view>
</template>

<script>
	import {
		mapState,
		mapMutations
	} from 'vuex'
	import uniIcon from "@/components/uni-icon.vue"
	// import popup from "@/components/popup.vue"
	import localData from "./data.json"
	import URL from "../../api/api.js"
	
	export default {
		components: {
			uniIcon
		},
		data() {
			return {
				banner: [
					"https://s2.luckincoffeecdn.com/luckywebrm/images/pimg/about/contact.png",
					"https://s2.luckincoffeecdn.com/luckywebrm/images/pimg/about/model.png",
					"https://s2.luckincoffeecdn.com/luckywebrm/images/index/cooperation/part5_picture2.png",
				],
				
				
				// 左侧列表中的场馆
				roomList: [],
				roomIndex: 0,
				// 选中一个场馆后，右侧列表中对象数组，每一个对象就对应一个可选择的日期
				itemObj: {},
				// 每一个日期中有多个时间段可以选择
				selectedRoomIndex: 0,
				selectedDateIndex: 0,
				selectedTimeIndex: 0,
				
				detail: {},
				
				height: 0,
				categoryActive: 0,
				scrollTop: 0,
				scrollHeight: 0,
				itemHeight: 0,
				
				itemTop: 0,
				popType: 'middle',
				showPopup: false,
				popupData: {},
				shopList: [],
				num: 1,
				triggered: false,
				refreshData: ''
				
			}
		},
		
		computed: {
			...mapState(['cartList'])
		},
		
		onLoad: function () {
// 			this.height = Math.floor(uni.getSystemInfoSync().windowHeight - 124);
// 			// #ifdef APP-PLUS
// 			this.height = Math.floor(uni.getSystemInfoSync().windowHeight - 180);
// 			// #endif
			this.height = Math.floor(uni.getSystemInfoSync().windowHeight);
			// #ifdef APP-PLUS
			this.height = Math.floor(uni.getSystemInfoSync().windowHeight - 56);
			// #endif
			
			// this.roomList = localData.data;
			// this.itemObj = this.roomList[0];
			
			uni.request({
			    url: URL.testUrl + '/optionalTime/getAll', //仅为示例，并非真实接口地址。
			    data: {
			    },
			    header: {
			        'custom-header': 'hello' //自定义请求头信息
			    },
				method: "POST",
			    success: (res) => {
					if(res.code == 500){
						console.log("请求出错，使用静态数据");
						this.roomList = localData.data1;
						this.itemObj = this.roomList[0];
						
					}else{
						this.roomList = res.data.data;
						this.itemObj = this.roomList[0];
						console.log(res.data);
					}
			        
			        this.text = 'request success';
			    }
			});
			
			console.log(this.roomList)
			
			
			this._freshing = false;
			setTimeout(() => {
				this.triggered = true;
			}, 2000)
		},
		
		methods: {
			...mapMutations(['calcTotal']),
			
			// 左侧列表点击事件
			categoryClick(index) {
				this.categoryActive = index;
				this.itemObj = this.roomList[index];
				this.roomIndex = index;
				console.log(index)
				console.log(this.itemObj)
			},
			// 隐藏弹出层
			hidePopup() {
				this.showPopup = false;
			},
			// 点击时间段进入预约详情页
			doOrder(dateData,index1,index2) {
				// 场馆
				console.log(this.itemObj);
				this.selectedRoomIndex =  this.roomIndex;
				// 日期
				console.log(this.itemObj.optionalDates[index1]);
				this.selectedDateIndex = index1;
				// 时间段
				console.log(this.itemObj.optionalDates[index1].optionalTimes[index2]);
				this.selectedTimeIndex = index2;
				
				uni.request({
				    url: URL.testUrl + '/optionalTime/getOneById', //仅为示例，并非真实接口地址。
				    data: {
						"id": this.itemObj.optionalDates[index1].optionalTimes[index2].id
				    },
					method: "POST",
				    success: (res) => {
						this.detail = res.data.data
						console.log("111111"+this.detail)
						
				        
				    }
				});
				
				this.showPopup = true;
			},
			
			// 确定预约
			submit() {
				console.log(this.detail)
				uni.request({
				    url: URL.testUrl + '/order/submitOrder', //仅为示例，并非真实接口地址。
				    data: {
						"optionalTimeId": this.detail.id 
				    },
					header:{
						"token": uni.getStorageSync("token")
					},
					method: "POST",
				    success: (res) => {
						console.log(res)
						if(res.data.code === 200){
							console.log("预约成功")
							// 增加一个成功提示
							// 页面跳转
							var orderId = res.data.data;
							uni.navigateTo({
								url: "../order/order-detail?orderId=" + orderId
							})
							
							uni.showToast({
								title: '预约成功',
								icon: 'none'
							})
							
						}else{
							// 增加一个错误提示
							console.log(res.data.msg)
							
							uni.showToast({
								title: res.data.msg,
								icon: 'none',
								duration: 2000
							})
							
						}
				        
				    }
				});
				
				this.showPopup = false;
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
				    url: URL.testUrl + '/optionalTime/getAll', //仅为示例，并非真实接口地址。
				    data: {
				    },
				    header: {
				        'custom-header': 'hello' //自定义请求头信息
				    },
					method: "POST",
				    success: (res) => {
						if(res.code == 500){
							console.log("请求出错，使用静态数据");
							this.refreshData = localData.data1;
							
							
						}else{
							this.refreshData = res.data.data;
							
							console.log(res.data);
						}
				        
				        this.text = 'request success';
				    }
				});
				
				
			},
			onRestore() {
				this.triggered = 'restore'; // 需要重置
				console.log(URL.testUrl);
				
				this.roomList = this.refreshData;
				this.itemObj = this.roomList[0]
				

			},
			onAbort() {
				console.log("onAbort");

			}
			
			
						
		}
	}
</script>

<style>
	.container{border-top: 1px solid #f0f0f0;}
	.swiper-box{height: 124px;}
	.banner{position: relative;}
	.banner-image{width: 100%;}
	
	.page-body {display: flex;}
	.nav {display: flex;width: 100%;}
	.nav-left {width: 180upx;background-color: #f7f7f7;}
	.nav-left-item {height: 86upx;border-bottom: solid 1px #ececec;font-size: 28upx;display: flex;align-items: center;justify-content: center;color: #666;}
	.nav-left-item:last-child{border-bottom: none;}
	.active {color: #333;background-color: #fff;font-weight: 700;}
	.nav-right {width: 570upx;background-color: #fff;}
	.nav-right-item {width: 100%;font-size: 28upx;padding: 0 26upx;box-sizing: border-box;}
	.item-header{padding-top: 36upx;font-size: 24upx;font-weight: 700;line-height: 24upx;}
	.item-state,.item-en,.item-default{font-size: 20upx;line-height: 20upx;color: #999;}
	.item-state{margin-top: 10upx;}
	.item-li{padding: 22upx 0;position: relative;border-bottom: 1px solid #f1f1f1;height: 134upx;}
	.nav-right-item .item-li:last-child{border-bottom: none;}
	.item-img{width: 134upx;height: 134upx;border-radius: 8upx;}
	.item-text{display: inline-block;vertical-align: top;margin-left: 16upx;}
	.item-title{font-size: 28upx;line-height: 28upx;font-weight: 700;}
	.item-en{margin: 10upx 0 8upx 0;}
	.item-price{font-size: 28upx;line-height: 28upx;font-weight: 700;margin-top: 28upx;}
	.plus-filled{position: absolute;bottom: 16upx;right: 0;color: #81aad2;}
	.fixed{position: fixed;top: 0;left: 206upx;background: #fff;}
	
	
	.mask {position: fixed;z-index: 99;top: 0;right: 0;bottom: 0;left: 0;background-color: rgba(0, 0, 0, .3);}
	.popup {position: fixed;z-index: 999;background-color: #fff;box-shadow: 0 0 30upx rgba(0, 0, 0, .1);display: flex;flex-direction: column;
align-items: center;width: 690upx;height: 920upx;border-radius: 10upx;top: 50%;left: 50%;transform: translate(-50%, -50%);box-sizing: border-box;overflow: hidden;}
	.popup-header,.popup-middle,.popup-footer{width: 100%;}
	.popup-header{height: 250upx;position: relative;background: url(https://s2.luckincoffeecdn.com/luckywebrm/images/index/cooperation/part5_picture2.png) center;}
	.popup-title,.popup-en{color: #fff;margin-left: 20upx;vertical-align: bottom;}
	.popup-title{font-size: 34upx;line-height: 54upx;margin-top: 190upx;}
	.popup-en{font-size: 20upx;line-height: 20upx;}
	.popup-close{color: #fff;font-size: 56upx;position: absolute;top: 20upx;right: 20upx;}
	.popup-middle{height: 600upx;box-sizing: border-box;padding: 10upx 30upx 0;overflow: auto;}
	.popup-footer{height: 232upx;border-top: 1px solid #f1f1f1;}
	.popup-price{display: flex;justify-content: space-between;padding: 32upx 28upx 24upx 38upx;height: 120upx;box-sizing: border-box;border-bottom: 1px solid #f1f1f1;}
	.text-main{font-size: 32upx;line-height: 32upx;color: #222;margin-bottom: 15upx;}
	.text-small{font-size: 20upx;color: #333;line-height: 20upx;}
	.numbox{display: flex;}
	.numbox-minus,.numbox-plus{font-size: 56upx;line-height: 68upx;}
	.numbox-minus{color: #91b5d9;background-color: #fff;}
	.numbox-value{line-height: 68upx;margin: 0 16upx;color: #91b5d9;font-weight: 700;min-width: 20upx;}
	.numbox-plus{color: #fff;color: #91b5d9;}
	.btn-group{display: flex;justify-content: center;margin-top: 26upx;}
	.btn{height: 60upx;line-height: 58upx;padding: 0;margin: 0;font-size: 24upx;text-align: center;color: #fff;border-radius: 0;margin-right: 14upx;}
	.first-btn{width: 206upx;border: 1px solid #dc5a00;background-color: #e06e11;}
	.two-btn{width: 170upx;border: 1px solid #6c9ccd;background-color: #f3f3f3;color: #73a1cf;}
	.three-btn{width: 180upx;border: 1px solid #6c9ccd;background-color: #7ca7d2;}
	.btn:after{border: none;}
	.sort{display: flex;margin-top: 30upx;}
	.sort-label{width: 120upx;line-height: 52upx;}
	.sort-select{width: 120upx;height: 50upx;text-align: center;border-radius: 30upx;border: 1px solid #e3dbd3;color: #e3dbd3;font-size: 24upx;line-height: 50upx;margin-right: 15upx;}
	.sort-select.active{background-color: #e3dbd3;color: #fff;font-weight: 400;}
	.describe{border-top: 1px solid #f1f1f1;margin-top: 30upx;padding: 30upx 0 10upx;}
	.describe-title{margin-bottom: 10upx;}
	.describe-text{color: #666;font-size: 24upx;}
</style>
