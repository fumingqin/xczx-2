<template>
	<view class="Ly_background">
		<view>
			<!-- 标题、发布时间、点击量、分享 -->
			<view class="clicks">
				<text class="title">{{scSpotContent[0].ticketTitle}}</text>
				<text class="time">开放时间：{{scSpotContent[0].ticketOpenUp}} </text>
			</view>
		</view>
		<!-- 顶部轮播图（可点击进入相册并放大） -->
		
			<view class="swi-item">
				<image style="margin-left:80upx;":src="piclist[0]" mode="aspectFill" @click="goImgList" />
				<!-- <view class="view">1张图片</view> -->
			</view>
		
		<!-- 文章内容 -->
		<view class="Zj_background">
			<view class="tweetsTitle">介绍</view>
			<view class="tweetscontent">暂无简介</view>
		</view>
		<!-- 门票滑块 -->
		<!-- 模块命名：Tk -->
		<view :hidden="admissionTicketStatus == '该景区暂无门票产品信息！'">
		<view class="Tk_scrollview">
			<view class="tweetsTitle2">门票</view>
			
			<view class="Tk_item"  v-for="(item,index) in admissionTicket" :key="index" >
				<view class="Tk_bacg">
					<!-- <text class="Tk_text1">{{item.admissionTicketName}}</text> -->
					<view class="Tk_text3">¥{{item.ticketAdultPrice}}元</view>
					<view class="Tk_text2">包含：{{item.ticketContain}}</view>
					<!-- <view style="display: block;text-overflow: ellipsis; white-space: nowrap; overflow: hidden; width: 400upx;">
						<text class="Tk_text2" v-for="(item2,index2) in item.ticketComment" :key="index2" >{{item2}}&nbsp;|&nbsp;</text>
					</view> -->
					
				</view>
				<view class="Tk_butter" @click="godetail(item)">立即预订</view>
			</view>
		</view>
		
		</view>
	</view>
</template>
<script>
	import $lyfw from '@/common/LYFW/LyfwFmq.js' //旅游服务
	export default {
		data() {
			return {
				piclist: [], //图片内容
				scSpotContent: [{
					ticketId: '',
					ticketTitle: '',
					ticketOpenUp: '',
					ticketScenicContent: '',
				}], //景区内容
				admissionTicket: [{
					admissionTicketID: '',
					admissionTicketName: '',
					ticketContain: '',
					ticketComment_s1: '',
					ticketComment_s2: '',
					ticketComment_s3: '',
					ticketAdultPrice: 240,
					ticketChildPrice: '', 
					ticketAdultPrice: '',
					companyId : '',
					executeScheduleId : '',
				}], //门票内容
				admissionTicketStatus:'',//判断是否展示
			}
		},
		onLoad(options) {
			this.lyfwData(options.ticketId);
		},
		onNavigationBarButtonTap: function() {
			this.share();
		},
		onShareAppMessage(res) {
			if (res.from === 'menu') { // 来自页面内分享按钮
				return {
					title: '来自' + this.userInfo.nickname + '的分享',
					imageUrl: this.piclist[0].ticketImage,
					success: function() {
						uni.showToast({
							title: '分享成功',
							duration: 3000
						})
					},
					fail: function() {
						uni.showToast({
							title: '分享失败',
							duration: 3000
						})
					}
				}
			}
		},
		methods: {
			//读取静态数据json.js 
			lyfwData: function(e) {
				// 请求景区图片
				uni.request({
					url:$lyfw.Interface.spt_GetticketDetailByticketId.value,
					method:$lyfw.Interface.spt_GetticketDetailByticketId.method,
					data:{
						ticketId : e,
					},
					header: {'content-type': 'application/json'},
					success: (res) => {
						console.log(res)
						this.piclist = res.data.data[0].ticketImage;
						console.log(this.piclist);
						this.scSpotContent = res.data.data;
						this.admissionTicket = res.data.data;
						this.admissionTicketStatus = res.data.msg;

					}
				})

			},

			//保存图片至本地并打开新页面
			goImgList() {
				uni.setStorageSync('imagePiclist', this.piclist);
				uni.navigateTo({
					url: '../currency/imglist2',
					
				})
			},
			//路由整合
			godetail: function(e) {
				uni.setStorage({
					key:'ticketInformation',
					data:e,
					success() {
						uni.navigateTo({
							url: 'orderAdd'
						})
					}
				})
				
			},

			//分享
			share() {
				uni.share({
					provider: "weixin",
					scene: "WXSceneSession",
					type: 0,
					href: "http://www.baidu.com",
					title: "来自" + this.userInfo.nickname + "的分享",
					summary: this.scSpotContent[0].ticketTitle,
					imageUrl: this.piclist[0].ticketImage,
					success: function() {
						uni.showToast({
							title: '分享成功',
							duration: 3000
						})
					},
					fail: function() {
						uni.showToast({
							title: '分享失败',
							duration: 3000
						})
					}
				});
			},

			// 统一跳转接口
			navTo(url) {
				uni.navigateTo({
					url
				})
			},

		}
	}
</script>

<style lang="scss">
	//页面背景
	.Ly_background {
		width: 100%;
		height: 100%;
		background: #fff;

		//顶部轮播区样式
		.swi {
			width: 100%;
			height: 440upx;

			.swi-item {
				width: 100%;
				height: 100%;
				overflow: hidden;
			}

			image {
				width: 100%;
				height: 100%;
			}
		}

		//相册下标
		.view {
			background-color: rgba(0, 0, 0, 0.7);
			color: #fff;
			display: flex;
			align-items: center;
			justify-content: center;
			padding: 0 32upx;
			height: 48upx;
			position: absolute;
			bottom: 24upx;
			right: 24upx;
			font-size: 30upx;
			border-radius: 24upx;
		}

		//时间、点击量、标题
		.clicks {
			display: flex;
			flex-direction: column;
			margin-top: 10upx;

			//标题
			.title {
				font-size: 40upx;
				font-weight: bold;
				color: #333333;
				padding-top: 32upx;
				padding: 10upx 0upx;
			}

			//时间
			.time {
				font-size: 32upx;
				color: #333333;
				font-weight: 500;
				// padding-left: 20upx;
				margin: 16upx 0;
			}
		}

		//Y轴滚动视图
		.Tk_scrollview {
			width: 100%;
			margin: 0 auto;
			background: #fff;

			.Tk_item {
				// background: #f9f9f9;
				margin: 10upx 0upx;
				// border-radius: 16upx;
				// display: flex;
				// box-shadow: 0px 1px 2px 0.1px #aaa;
				.Tk_text3 {
					// position: absolute;
					// float: right;
					// right: 1upx;
					top: 0upx;
					font-size: 28upx;
					color: #FF6600;
				}
				.Tk_bacg {
					position: relative;
					margin: 10upx 0upx;

					.Tk_text1 {
						font-size: 34upx;
						// margin-left: 12upx;
						display: flex;
						font-weight: bold;
						text-overflow: ellipsis; //文章超出宽度隐藏并用...表示
						white-space: nowrap;
						overflow: hidden;
						width: 480upx; //内容宽度
						
					}

					.Tk_text2 {
						font-size: 34upx;
						font-weight: 500;
						margin-top: 20upx;
						// margin-left: 12upx;
					}

					

					
				}

			}
			
		}
		.Tk_butter {
			// position: absolute;
			width: 510upx;
			height: 80upx;
			// bottom: -4upx;
			// right: -158upx;
			margin-top: 24upx;
			margin-left: 120upx;
			background: #FF6600;
			border-radius: 12upx;
			color: #FFFFFF;
			font-size: 36upx;
			font-weight: 500;
			text-align: center;
			line-height: 78upx;
		}

		//推文标题
		.tweetsTitle {
			font-size: 38upx;
			font-weight: bold;
			color: #333333;
			padding: 24upx 32upx;
			padding-top: 40upx;
		}

		.tweetsTitle2 {
			font-size: 38upx;
			font-weight: bold;
			color: #333333;
			padding: 10upx 0upx;
			padding-top: 20upx;
		}

		//推文内容
		.tweetscontent {
			display: flex;
			position: relative;
			letter-spacing: 4upx;
			line-height: 48upx; 
			color: #333333;
			padding: 32upx 32upx;
			padding-top: 8upx;
			font-size: 33upx;
		}
	}
</style>
