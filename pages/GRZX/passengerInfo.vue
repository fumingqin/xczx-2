<!-- 乡村振兴  第二版 -->
<template>
	<view class="content">	
		<view class="passengerList">
			<view class="boxClass" v-for="(item, index) in passengerList" :key="index" @click="choosePassenger(item)">  <!--非个人中心页面进入 -->
				<image src="../../static/GRZX/newIcon/name.png" class="nameIcon"></image>
				<image src="../../static/GRZX/newIcon/type.png" class="typeIcon"></image>
				<image src="../../static/GRZX/newIcon/num.png" class="numIcon"></image>
				<view class="nameClass">{{item.userName}}</view>
				<view class="typeClass">{{item.userType}}</view>
				<view class="codeNumClass fontStyle">{{item.userCodeNum}}</view>
				<view class="phoneNumClass fontStyle">{{item.userPhoneNum}}</view>
				<image v-if="item.userDefault==true" src="../../static/GRZX/newIcon/default.png" class="defaultIcon"></image>
				<view>
					<image v-if="item.hiddenIndex == 0"  class="checkClass" src="../../static/GRZX/newIcon/unchecked.png"></image>
					<image v-if="item.hiddenIndex == 1"  class="checkClass" src="../../static/GRZX/newIcon/checked.png"></image>
				</view>
			</view>
		</view>	
		<view class="btnBox"> 
			<button @click="addPassenger" class="btnAdd1">添加乘客</button>
			<!-- <button @click="definite" class="btnDefinite">确定</button> -->
		</view>
		<view class="returnBox">
			<image class="returnClass" src="../../static/GRZX/btnReturn.png" @click="returnPages"></image>
			<view class="titleClass" @click="returnPages">上一页</view>
			<view class="addClass" @click="definite">确定</view>
		</view>
	</view>
</template>

<script>
	export default {
		data(){
			return{
				limt:'', //限制的人数
				passengerList:[], //乘客列表
			}
		},
		onLoad(options){
			//传参
			//limitNum参数为你限制添加乘车人的数量（大于等于1）
			this.limit=options.limitNum;
			uni.getStorage({
				key:'userInfo',
				fail() {
					uni.showToast({
						icon:'none',
						title:'暂未登录,请登录后查看'
					})
					uni.stopPullDownRefresh();
					//#ifdef APP-PLUS
					setTimeout(function(){
						uni.navigateTo({	
							url  : '/pages/GRZX/userLogin'
						}) 
					},500);
					//#endif
					//#ifdef MP-WEIXIN
					setTimeout(function(){
						uni.navigateTo({
							url:'/pages/Home/wxAuthorize',
						})
					},1500);
					// #endif
				}
			})
		},
		onPullDownRefresh:function(){
		  this.loadData();
		},
		onShow() {
			this.loadData();
		},
		methods:{
			//--------------------------------加载数据------------------------------
			loadData(){
				var that=this;
				var array=[];
				var list=[];
				uni.getStorage({
					key:"passengerList",
					success(res2) {
						for(var j=0;j<res2.data.length;j++){
							list.push(res2.data[j].passengerId);
						}
					}
				})
				uni.getStorage({
					key:'userInfo',
					success(res){
						if(res.data.userId==""||res.data.userId==null){
							uni.showToast({
								icon:'none',
								title:'暂未登录,请先登录'
							})
						}else{
							uni.request({
								url:that.$GrzxInter.Interface.userInfoList.value,
								data:{
									userId:res.data.userId
								},
								method:that.$GrzxInter.Interface.userInfoList.method,
								success(res1) {
									console.log(res1,'乘车人列表')
									for(var i=0;i<res1.data.data.length;i++){
										if(res1.data.data[i].userSex==0){
											res1.data.data[i].userSex="男";
										}else{
											res1.data.data[i].userSex="女";
										}
										var data1=res1.data.data[i];
										data1.hiddenIndex=0;
										for(var q=0;q<list.length;q++){
											if(data1.passengerId==list[q]){
												data1.hiddenIndex=1;
											}
										}
										array.push(data1);
										uni.stopPullDownRefresh();
									}
									var defaultList=[];
									for(var n=0;n<array.length;n++){
										if(array[n].hiddenIndex==1){
											defaultList.unshift(array[n]);
										}else{
											defaultList.push(array[n]);
										}
									}
									that.passengerList=defaultList;
								}
							})
						}
						
					}
				})
			},
			
			//--------------------------------添加乘客------------------------------
			addPassenger(){
				var that=this;
				uni.getStorage({
					key:'userInfo',
					success() {
						uni.navigateTo({
							url:that.$GrzxInter.Route.addPassenger.url+'?type=add'
						})
					},
					fail() {
						uni.showToast({
							icon:'none',
							title:'暂未登录,无法添加乘客'
						})
					}
				})
			},
			
			//--------------------------------返回上一页------------------------------
			returnPages(){
				uni.navigateBack();
			},
			
			//--------------------------------编辑乘车人信息------------------------------
			editPassenger(e){   
				uni.setStorage({
					key:'editPassenger',
					data:e
				})
				uni.navigateTo({
					url:'/pages/GRZX/addPassenger?type=edit'
				})
			},
			
			//--------------------------------选择乘车人------------------------------
			choosePassenger(e){  
				var list=this.passengerList;
				var count=0;
				for(var i=0;i<list.length;i++){
					if(list[i].hiddenIndex==1){
						count++;
					}
				}
				if(e.hiddenIndex==1){
					e.hiddenIndex=0;
				}else if(count>(this.limit-1) && this.submitType==2){
					uni.showToast({
						title: '乘客最多只能添加'+this.limit+'名',
						icon:"none"
					});
				}else{
					e.hiddenIndex=1;
				}		
			},
			
			//--------------------------------提交选中的乘客------------------------------
			definite(){
				var data=this.passengerList;
				var array=[];
				for(var i=0;i<data.length;i++){
					if(data[i].hiddenIndex==1){
						array.push(data[i]);
					}
				}
				console.log(array,"111");
				if(array.length==0){
					uni.showToast({
						title: '请选择乘客',
						icon:"none"
					})
				}else{
					console.log(array,"222");
					uni.setStorageSync('passengerList',array);
					uni.navigateBack();	
				}			
			}
		}
	}
</script>

<style lang="scss">
	page{
		background-color: #F6F8FC;
	}
	.content {
		width: 100%;
	}
	.checkClass{	//勾选
		position: absolute;
		left: 89.26%;
		top:70upx;
		width: 50upx;
		height:50upx ;
	}
	.returnClass{ //返回按钮
		width: 22upx;
		/* #ifdef H5 */
		margin-top: 30upx;
		/* #endif */
		/* #ifndef H5 */
		margin-top: 103upx;
		/* #endif */
		height: 40upx;
		margin-left: 3.47%;
	}
	.btnRight{ //进入编辑的箭头
		width:60upx;
		height: 60upx; 
		position: absolute;
		left: 90%;
		top:80upx;
	}
	.titleClass{
		margin-left: 20upx;
		font-size: 38upx;
		/*font-weight: bold; */
		/* #ifdef H5 */
		margin-top: 20upx;
		/* #endif */
		/* #ifndef H5 */
		margin-top: 95upx;
		/* #endif */
	}
	.passengerList{ //列表样式
		width: 100%;
		/* #ifdef H5 */
		margin-top: 110upx;
		margin-bottom: 30upx;
		padding-bottom: 160upx;
		/* #endif */
		/* #ifndef H5 */
		margin-top: 180upx;
		margin-bottom: 160upx;
		/* #endif */
	}
	.btnAdd1{
		background-color: #0000ff;
		color: #FFFFFF;
		border-radius: 12upx;
		width: 45%;
		height: 90upx;
		line-height: 90upx;
		margin-top: 30upx;
	}
	.btnDefinite{
		background-color: #ff0000;
		color: #FFFFFF;
		border-radius: 12upx;
		width: 45%;
		height: 90upx;
		line-height: 90upx;
		margin-top: 30upx;
	}
	.btnAdd2{ //添加按钮
		width: 92%;
		height: 90upx;
		line-height: 90upx;
		border-radius: 12upx;
		background-color: #FC4646;
		margin-top: 30upx;
	}
	.boxClass{
		background-color: #FFFFFF;
		width: 94%;
		margin-top: 20upx;
		margin-left: 3%;
		height: 200upx;
		font-size:28upx;
		color: #666666;
		position: relative;
		border-radius:20upx ;
	}
	.nameClass{
		font-size: 36upx;
		color: #2C2D2D;
		position: absolute;
		left: 15%;
		top:33upx;
		width: 18%;
		font-weight: bold;
		display: block;
		text-overflow: ellipsis;
		white-space: nowrap;
		overflow: hidden;
	}
	.sexClass{
		position: absolute;
		left: 25%;
		top:47upx;
		font-size: 24upx;
		color: #2C2D2D;
	}
	.fontStyle{
		color: #666666;
		font-size: 30upx;
	}
	.typeClass{
		font-size: 30upx;
		color: #2C2D2D;
		line-height: 57upx;
		position: absolute;
		left: 16%;
		top:115upx;
	}
	.codeClass{
		position: absolute;
		left: 4%;
		top:108upx;
	}
	.codeNumClass{
		position: absolute;
		left: 36%;
		top:123upx;
	}
	.phoneClass{
		position: absolute;
		left: 4%;
		top:163upx;
	}
	.phoneNumClass{
		position: absolute;
		left: 42%;
		top:40upx;
	}
	.returnBox{
		width: 100%;
		/* #ifdef H5 */
		height: 90upx;
		/* #endif */
		/* #ifndef H5 */
		height: 160upx;
		/* #endif */
		position: fixed;
		top: 0upx;
		display: flex;
		flex-direction: row;
		background-color: #F6F8FC;
		z-index: 9999;
	}
	.btnBox{ //按钮位置样式
		position: fixed;
		display: flex;
		flex-direction: row;
		bottom: 0upx;
		width: 100%;
		height: 158upx;
		background-color: #F6F8FC;
		/* border: 1px solid #4CD964; */
	}
	.redBox{
		position: absolute;
		left:32%;
		top: 34upx;
		display: flex;
	}
	.fontClass{ //本人,紧急联系人,待审核,审核通过,审核未通过
		height: 40upx;
		line-height: 40upx;
		font-size: 24upx;
		color: #0000ff;
		margin-left: 20upx;
		border: 1upx solid #0000ff;
		border-radius: 10upx;
		text-align: center;
	}
	.nameIcon{
		width: 55upx;
		height: 60upx;
		position: absolute;
		top: 25upx;
		left: 5%;
	}
	.codeIcon{
		width: 60upx;
		height: 60upx;
		position: absolute;
		top: 55upx;
		left: 5%;
	}
	.numIcon{
		width: 50upx;
		height: 53upx;
		position: absolute;
		top: 30upx;
		left: 34%;
	}
	.typeIcon{
		width:55upx;
		height: 55upx;
		position: absolute;
		top: 115upx;
		left: 5%;
	}
	.defaultIcon{
		width:55upx;
		height: 55upx;
		position: absolute;
		top: 0upx;
		left: 93%;
	}
	.addClass{	//添加
		position: absolute;
		/* #ifdef H5 */
		top: 20upx;
		/* #endif */
		/* #ifndef H5 */
		top: 90upx;
		/* #endif */
		/* #ifdef MP-WEIXIN */
		left: 60%;
		/* #endif */
		/* #ifndef MP-WEIXIN */
		left: 85%;
		/* #endif */
		color: #232323;
		font-size: 38upx;
	} 
</style>
