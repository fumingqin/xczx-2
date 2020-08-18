<template>
	<view class="contentView">
		<!-- <view class="top u-f-jsb" style="background-color: #FC4646; width: 100%; height: 180rpx;">
			<view style="color: #FFFFFF; font-size: 35rpx; margin-left: 20rpx;">{{getCtkyOrderStatus(orderInfo.state)}}</view>
			<view style="color: #FFFFFF; font-size: 30rpx; margin-right: 20rpx;">￥{{orderInfo.totalPrice}}</view>
		</view> -->
		<!-- 头部视图 -->
		<view class="head" style="margin-top: 30rpx;padding-bottom: 30rpx;">
			<view style="height: 200rpx;background-color: #3DABFC;border-top-right-radius: 20rpx;border-top-left-radius: 20rpx;color: #FFF;padding-top:20rpx ;">
				<view style="display: flex;justify-content: space-between;">
					<view style="display: flex;font-size: 30rpx;">
						<view>{{gettime(orderInfo.setOutTime)}}</view>
						<view style="margin-left: 20rpx;">发车</view>
					</view>
					<view>{{getCtkyOrderStatus(orderInfo.state)}}</view>
				</view>
				<view style="display: flex;justify-content: space-between;align-items: center;padding-top: 30rpx;">
					<view style="font-size: 36rpx;height: 40rpx;text-align: center;">
						<view>{{orderInfo.startSiteName}}</view>
						<!-- <view>{{(gettime(orderInfo.setOutTime)).substr(11,16)}}</view> -->
					</view>
					<view style="font-size: 28rpx;text-align: center;height: 40rpx;">
						<view style="display: flex;">——<view style="border: 1rpx solid #FFF;padding: 3rpx 10rpx;">班次</view>——</view>
						<view>{{getScheduleNum(orderInfo.planScheduleCode)}}</view>
					</view>
					<view style="font-size: 36rpx;height: 40rpx;">
						<view>{{orderInfo.endSiteName}}</view>
					</view>
				</view>
				<!-- <view style="margin-top: 100rpx;font-size: 36rpx;display: flex;justify-content: space-between;">
					<view>{{orderInfo.carType}}:{{orderInfo.lineName}} x{{getTicketNum(orderInfo)}}</view>
					<view>￥{{orderInfo.totalPrice}}</view>
				</view> -->

			</view>
			<!-- 发车时间 -->
			<view>
				<view class="headText"> 订单号：{{orderInfo.orderNumber}}</view>
				<view class="headText"> 乘车人数：{{getTicketNum(orderInfo)}}人</view>
				<view class="headText"> 金额：{{orderInfo.totalPrice}}元</view>
			</view>
			<view v-for="(item,index) in passageInfo" :key="index">
				<view style="border: 1rpx solid #D9D9D9;margin:0 20rpx 20rpx 20rpx;">
					<view class="headText">乘车人：{{item.userName}}</view>
					<view class="headText">身份证号码：{{userCodeNumChange(item.userCodeNum)}}</view>
					<view class="headText">保险：{{isInsured(orderInfo.insured)}}</view>
					<view class="infoCotent" style="text-align: center;">
						<!-- 二维码 -->
						<view style="justify-content: center; align-items: center;display: flex;">
							<view class="QRImage">
								<tki-qrcode v-if="isShowQrcode" :cid="qrcodeIndex+index" ref="qrcode" :val="getOneTicketNum(orderInfo.ticketNumber,index)"
								 :size="size" :unit="unit" :background="background" :pdground="pdground" :icon="icon" :iconSize="iconsize" :lv="lv"
								 :onval="onval" :loadMake="loadMake" :usingComponents="true" @result="qrR" />
								<view v-if="isShowQrcode == false" style="font-weight: 300;color: #2C2D2D;font-size: 36rpx;justify-content: center; align-items: center;">{{getQRCodeStatus(orderInfo.state)}}</view>
							</view>
						</view>
						<view v-if="isShowQrcode&&orderInfo.carType != '定制巴士'" style="color: #2C2D2D;font-size: 32rpx;font-weight: 300; padding-bottom: 10rpx;">
							取票号 {{getOneTicketNum(orderInfo.ticketNumber,index)}}
						</view>
						<view v-if="isShowQrcode" style="color: #999999;font-size: 28rpx;font-weight: 300; padding-bottom: 32rpx; ">
							出示二维码，检票上车
						</view>
					</view>
				</view>
			</view>
		</view>
	</view>
</template>

<script>
	import uQRCode from '@/pages_CTKY/components/CTKY/uni-qrcode/uqrcode.js'
	import tkiQrcode from '@/pages_CTKY/components/CTKY/tki-qrcode/tki-qrcode.vue'
	export default {
		components: {
			tkiQrcode
		},

		data() {
			return {
				orderID: '',
				role: '过发车时间将无法退票',
				orderInfo: [], //订单数据
				passageInfo: [],
				ticketNum: 0,
				seat: '无', //座位号
				qrcodeSrc: '', //二维码
				qrcodeText: 'uQRCode',
				qrcodeSize: 150,
				ticketNumber: '',
				ticktIndex: '', //车票下标
				specialCodeArray: [],
				// isShowQrcode:'',
				orderState: '', //订单状态


				qrcodeIndex: 'qrcodeIndex',
				isShowQrcode: true,
				val: '二维码', // 要生成的二维码值
				size: 300, // 二维码大小
				unit: 'upx', // 单位
				background: '#b4e9e2', // 背景色
				// foreground: '#ffffff', // 前景色
				pdground: '#32dbc6', // 角标色
				icon: '', // 二维码图标
				iconsize: 40, // 二维码图标大小
				lv: 3, // 二维码容错级别 ， 一般不用设置，默认就行
				onval: false, // val值变化时自动重新生成二维码
				loadMake: true, // 组件加载完成后自动生成二维码
				src: '' // 二维码生成后的图片地址或base64
			}
		},
		onLoad(res) {
			var that = this;
			var orderInfo = JSON.parse(res.orderInfo);
			that.orderInfo = orderInfo;
			console.log(that.orderInfo)
			that.orderState = orderInfo.state;
			this.specialCodeArray = orderInfo.CheckInfoList;
			that.stringTurnArray(orderInfo.iDNameType);
			//检票号---生成二维码
			if (orderInfo.carType == '定制巴士') {
				// for(let i = 0;i < orderInfo.passageInfo.length;i++){
				// 	this.ticketNumber = orderInfo.ticketNumber;
				// 	that.make(this.orderInfo.ticketNumber,i);
				// }
			} else { //定制巴士
				// for(let i = 0;i < orderInfo.CheckInfoList.length;i++){
				// 	this.ticketNumber = orderInfo.CheckInfoList[i].CheckCode;
				// 	that.make(this.ticketNumber,i);
				// }
			}
			//计算车票数量
			that.getTicketNum(orderInfo);
		},
		methods: {
			//-------------------------------生成二维码-------------------------------
			make(param, index) {
				if (param) {
					uQRCode.make({
						canvasId: 'ctkyQrcode' + index,
						text: param,
						size: this.qrcodeSize,
						margin: 20,
						success: res => {
							console.log('生成二维码成功', param);
							// console.log('完成')
							this.qrcodeSrc = res
						},
						complete: () => {
							// uni.hideLoading()
							// console.log('完成')
						}
					})
				}
			},
			//-------------------------------身份证-------------------------------
			userCodeNumChange: function(userCodeNum) {
				return (userCodeNum.substr(0, 6)) + '******' + (userCodeNum.substr(16, 18))
			},
			//-------------------------------判断是否有保险-------------------------------
			isInsured: function(param) {
				if (this.orderInfo.carType == '定制巴士') {
					return '无保险'
				} else if (param == 'False') {
					return '无保险'
				} else {
					return '乘车险 x1'
				}
			},
			//-------------------------------报班信息-------------------------------
			getDetailInfo(param) {
				if (!param) {
					return '该车未报班，无法获取详细信息'
				} else {
					return param
				}
			},
			//-------------------------------获取班次信息-------------------------------
			getScheduleNum: function(param) {
				if (param) {
					return param;
				} else {
					return '无';
				}
			},
			//-------------------------------获取乘车人信息-------------------------------
			stringTurnArray(param) {
				var that = this;

				let a = param.indexOf('|')
				var singleArray = [];
				if (a == -1) { //不存在'|' 只有一张车票
					var array = param.split(',');
					var passenger = {
						userName: array[1],
						userCodeNum: array[0],
					}
					that.passageInfo.push(passenger);
					console.log('只有一张票')
					// this.ticketNumber = that.orderInfo.ticketNumber;
					// that.make(this.orderInfo.ticketNumber,0);
				} else { //多人订票
					//存在'|'
					var array = param.split('|');
					for (let i = 0; i < array.length; i++) {
						singleArray = array[i].split(',');
						var passenger = {
							userName: singleArray[1],
							userCodeNum: singleArray[0],
						}
						that.passageInfo.push(passenger);
						// this.ticketNumber = that.orderInfo.ticketNumber;
						// that.make(that.getOneTicketNum(this.orderInfo.ticketNumber,i),i);
					}
				}
			},
			//-------------------------------二维码生成后的图片地址或base64-------------------------------
			qrR(res) {
				this.src = res
			},
			//-------------------------------获取取票号-------------------------------
			getOneTicketNum(ticketNum, index) {
				var that = this;
				if (!(/(^[1-9]\d*$)/.test(that.orderState))) { //如果不是数字
					if (that.orderState == '尚未支付') {
						return '尚未支付'
					} else if (that.orderState == '作废') {
						return '尚未支付'
					} else if (that.orderState == '已退票') {
						return '尚未支付'
					} else if (that.orderState == '支付正常') {
						if (ticketNum) {
							let a = ticketNum.indexOf(',')
							if (a == -1) {
								var array = ticketNum.split('-');
								let ticketHeader = array[0];
								this.seat = array[1];
								return ticketHeader;
							} else {
								var array = ticketNum.split('-');
								let ticketHeader = array[0];
								var array2 = array[1];
								var array3 = array2.split(',');
								return ticketHeader;
								// return ticketHeader + '-' + array3[index];
							}
						}
					} else if (that.orderState == '已完成') {
						if (ticketNum) {
							let a = ticketNum.indexOf(',')
							if (a == -1) {
								var array = ticketNum.split('-');
								let ticketHeader = array[0];
								this.seat = array[1];
								return ticketHeader;
							} else {
								var array = ticketNum.split('-');
								let ticketHeader = array[0];
								var array2 = array[1];
								var array3 = array2.split(',');
								// return ticketHeader + '-' + array3[index];
								return ticketHeader;
							}
						}
					}
				} else if (that.orderState == 4) {
					if (ticketNum) {
						let a = ticketNum.indexOf(',')
						if (a == -1) {
							var array = ticketNum.split('-');
							let ticketHeader = array[0];
							this.seat = array[1];
							return ticketHeader;
						} else {
							var array = ticketNum.split('-');
							let ticketHeader = array[0];
							var array2 = array[1];
							var array3 = array2.split(',');
							// return ticketHeader + '-' + array3[index];
							return ticketHeader;
						}
					}
				} else if (that.orderState == 5) {
					if (ticketNum) {
						let a = ticketNum.indexOf(',')
						if (a == -1) {
							var array = ticketNum.split('-');
							let ticketHeader = array[0];
							this.seat = array[1];
							return ticketHeader;
						} else {
							var array = ticketNum.split('-');
							let ticketHeader = array[0];
							var array2 = array[1];
							var array3 = array2.split(',');
							// return ticketHeader + '-' + array3[index];
							return ticketHeader;
						}
					}
				} else if (that.orderState == 6) {
					return '已退票'
				} else if (that.orderState == 7) {
					return '未支付'
				} else if (that.orderState == 9) {
					return '已撤销'
				} else if (that.orderState == 22) {
					return '已改签'
				}
			},
			//-------------------------------获取定制巴士取票号-------------------------------
			getSpecialOneTicketNum(res, index) {
				if (this.orderInfo.carType == '定制巴士') {
					return res[index].CheckCode
				}
			},
			//-------------------------------计算车票数量-------------------------------
			getTicketNum(param) {
				return Number(param.fullTicket) + Number(param.halfTicket) + Number(param.carryChild)
			},
			//-------------------------判断订单状态-------------------------
			getCtkyOrderStatus(param) {
				var that = this;
				if (!(/(^[1-9]\d*$)/.test(param))) { //如果不是数字
					if (param == '尚未支付') {
						that.isShowQrcode = false;
					} else if (param == '作废') {
						that.isShowQrcode = false;
					} else if (param == '已退票') {
						that.isShowQrcode = false;
					} else {
						that.isShowQrcode = true;
					}
					return param
				} else if (param == 4) {
					that.isShowQrcode = true;
					return '进行中'
				} else if (param == 5) {
					that.isShowQrcode = true;
					return '已完成'
				} else if (param == 6) {
					that.isShowQrcode = false;
					return '已退票'
				} else if (param == 7) {
					that.isShowQrcode = false;
					return '未支付'
				} else if (param == 9) {
					that.isShowQrcode = false;
					return '已撤销'
				} else if (param == 22) {
					that.isShowQrcode = true;
					return '已改签'
				}
			},
			//-------------------------判断订单状态-------------------------
			getQRCodeStatus(param) {
				if (param == 6) { //已退票
					return '已退票'
				} else if (param == 7) { //未支付
					return '未支付'
				} else if (param == 9) { //已撤销
					return '已撤销'
				} else if (param == '已退票') { //已退票-----定制巴士
					return '已退票'
				} else if (param == '尚未支付') { //作废
					return '尚未支付'
				} else if (param == '作废') { //作废
					return '已作废'
				}
			},
			//--------------------时间转换-----------------
			gettime: function(param) {
				let array = param.split(":");
				var a = array[0] + ":" + array[1];
				return a;
			}
		}
	}
</script>

<style>
	/* flex布局 */
	.u-f,
	.u-f-ac,
	.u-f-jsb,
	.u-f-ajc {
		display: flex;
	}

	.u-f-ac,
	.u-f-jsb,
	.u-f-ajc {
		align-items: center;
	}

	.u-f-ajc {
		justify-content: center;
	}

	.u-f-jsb {
		justify-content: space-between;
	}

	.paddingTLR {
		padding-top: 10rpx;
		padding-left: 20rpx;
		padding-right: 20rpx;
	}

	/* 内容 */
	page,
	.contentView {
		background: #F5F9FC;
		display: block;
	}

	/* 头部视图 */
	.head {
		background: #FFFFFF;
		border-radius: 20rpx;
		margin: 20rpx;
		/* margin-top: -40rpx; */
		/* padding: 20rpx 0; */
	}

	/* 起始站/价格 */
	.head>view:first-child {
		padding: 12rpx 20rpx;
		font-size: 30rpx;
		color: #2C2D2D;
		font-weight: bold;
	}

	/* 发车时间 */
	.headText {
		padding: 10rpx 20rpx;
		font-size: 30rpx;
		color: #333;
		font-weight: 300;
	}

	/* 滚动区域 */
	.scrollBox {
		height: 100%;
	}

	.infoCotent {
		border-radius: 20rpx;
		background: #FFFFFF;
		margin: 0 20rpx;
		margin-bottom: 20rpx;
		padding: 20rpx 0;
	}

	/* 乘客信息 */
	.passageInfo {}

	/* 标题 */
	.title {
		text-align: left;
		color: #666666;
		display: block;
		padding-top: 20rpx;
		padding-bottom: 20rpx;
		font-size: 28rpx;
		font-weight: 300;
	}

	.title view {
		margin-bottom: 20rpx;
		margin-left: 20rpx;
	}

	.detailInfo {
		text-align: left;
		color: #2C2D2D;
		display: block;
		padding-top: 20rpx;
		padding-bottom: 20rpx;
		font-size: 28rpx;
		font-weight: 500;
	}

	.detailInfo view {
		margin-bottom: 20rpx;
		margin-left: 60rpx;
	}

	.QRImage {
		display: flex;
		width: 100%;
		align-items: center;
		justify-content: center;
		margin-bottom: 20rpx;
		width: 300rpx;
		height: 300rpx;
	}
</style>
