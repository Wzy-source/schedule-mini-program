<template>
	<view class="page">
		<view v-if="!isLogin" class="loginPanel" @tap="toLogin">
			<view style="align-self: center; display: flex;">
				<view class="avatar">
					<image src="../../static/boy.png" mode="scaleToFill"></image>
				</view>
				<view class="loginTxt">请先登录/注册</view>
			</view>
			<view class="toLoginArrow">
				<image src="../../static/operation/arrow.png" mode="scaleToFill"></image>
			</view>
		</view>

		<view v-else class="avatarAndName">
			<view style="align-self: center; display: flex;">
				<view class="avatar">
					<image :src="avatarUrl" mode="scaleToFill"></image>
				</view>
				<view class="nameTxt">{{name}}</view>
			</view>
			<view class="toLoginArrow">
				<image src="../../static/operation/arrow.png" mode="scaleToFill"></image>
			</view>
		</view>
		
		<view v-if="isLogin">
			<view class="myRecordList" @tap="toRecordList">
				<view style="display: flex; align-self: center;">
					<view class="imgStyle">
						<image src="../../static/myInfoIcon/appointment.png" mode="scaleToFill"></image>
					</view>
					<view class="fontStyle">
						查看/填写记录
					</view>
				</view>
				<view class="arrowStyle">
					<image src="../../static/operation/arrow.png" mode="scaleToFill"></image>
				</view>
			</view>
			<view class="recordBox">
				<view class="myRecordList" @tap="showNoticeItems">
					<view style="display: flex; align-self: center;">
						<view class="imgStyle">
							<image src="../../static/myInfoIcon/notice.png" mode="scaleToFill"></image>
						</view>
						<view class="fontStyle">
							通知管理
						</view>
					</view>
					<view class="arrowStyle rotateStyle" :class="showNotice===true ? 'showNoticeArrowStyle' : ''">
						<image src="../../static/operation/arrow.png" mode="scaleToFill"></image>
					</view>
				</view>
				<!-- 下面是noticeItem的 container -->
				<view class="noticeInfo" :class="showNotice===true? 'showNoticeContentStyle':'hideNoticeContentStyle'">

					<view class="noticeNumAndBtn" v-if="isTeacher==='true'">
						<view class="noticeNum">
							<text>学生预约通知</text> 
							<text style="color: #666666;">(剩余次数:</text>
							<text style="color: #E54D42;">{{studentAppointmentNoticeNum}}</text>
							<text style="color: #666666;">次)</text>
						</view>
						<view><button class="cu-btn round lines-blue shadow noticeBtn"
								@tap='studentAppointmentNotice'>订阅一次</button>
						</view>
					</view>
					<view class="noticeNumAndBtn" v-if="isTeacher==='true'">
						<view class="noticeNum">
							<text>预约取消通知</text> 
							<text style="color: #666666;">(剩余次数:</text>
							<text style="color: #E54D42;">{{studentCancelAppointmentNoticeNum}}</text>
							<text style="color: #666666;">次)</text>
						</view>
						<view><button class="cu-btn round lines-blue shadow noticeBtn"
								@tap='studentCancelAppointmentNotice'>订阅一次</button>
						</view>
					</view>
					<!-- 预约受理通知 -->
					<view class="noticeNumAndBtn" v-if="isTeacher==='false'">
						<view class="noticeNum">
							<text>预约受理通知</text> 
							<text style="color: #666666;">(剩余次数:</text>
							<text style="color: #E54D42;">{{teacherDealAppointmentNoticeNum}}</text>
							<text style="color: #666666;">次)</text>
						</view>
						<view><button class="cu-btn round lines-blue shadow noticeBtn"
								@tap='teacherDealAppointmentNotice'>订阅一次</button>
						</view>
					</view>
					<!-- 这个置于底部 -->
					<view class="noticeAlertTxt" v-show="showNoticeAlertTxt">
						<view>订阅规则说明：</view>
						<view>小程序订阅规定：您订阅一次，我们有权向您发送一条消息通知</view>
					</view>
				</view>
			</view>
		</view>
	</view>
</template>

<script>
	import baseUrl from '../../url.js'
	export default {
		data() {
			return {
				avatarUrl: uni.getStorageSync('avatarUrl'),
				name: uni.getStorageSync('name'),
				noticeNum: 0,
				showNotice: false,
				showNoticeAlertTxt: false,
				//预约剩余次数相关
				studentAppointmentNoticeNum: 0, //学生预约给老师的通知
				studentCancelAppointmentNoticeNum: 0, //学生取消预约给老师的通知
				teacherDealAppointmentNoticeNum:0,//老师处理预约给学生通知
				isTeacher:''

			}
		},
		methods: {
			toLogin() {
				uni.navigateTo({
					url: '../login/login'
				})
			},
			toRecordList() {
				uni.navigateTo({
					url: 'recordList/feedbackList'
				})
			},
			showNoticeItems() {
				this.showNotice = !this.showNotice
				if (this.showNoticeAlertTxt === false) {
					setTimeout(() => {
						this.showNoticeAlertTxt = true
					}, 250)
				} else {
					this.showNoticeAlertTxt = false
				}
			},
			/* 增加学生预约通知 */
			studentAppointmentNotice() {
				let _this = this
				wx.requestSubscribeMessage({
					tmplIds: ['D9rT-8zRcF-YRyS2vNQQHc4IzXXvrMH9qxhC8eBiwRw'],
					success(res) {
						if (res['D9rT-8zRcF-YRyS2vNQQHc4IzXXvrMH9qxhC8eBiwRw'] === 'accept') {
							uni.request({
								url: baseUrl + `permission/increaseAppointmentCount`,
								method: 'POST',
								header: {
									'content-type': 'application/x-www-form-urlencoded'
								},
								data: {
									skey: uni.getStorageSync('skey')
								},
								success(res) {
									_this.studentAppointmentNoticeNum = res.data.data
								}
							})
						}
					}
				})
			},
			studentCancelAppointmentNotice() {
				let _this = this
				wx.requestSubscribeMessage({
					tmplIds: ['-mGz-2oSlZIrm-nfK6V2jj7sUSq3v1cEn6x8x-aSvyo'],
					success(res) {
						if (res['-mGz-2oSlZIrm-nfK6V2jj7sUSq3v1cEn6x8x-aSvyo'] === 'accept') {
							uni.request({
								url: baseUrl + `permission/increaseCancelCount`,
								method: 'POST',
								header: {
									'content-type': 'application/x-www-form-urlencoded'
								},
								data: {
									skey: uni.getStorageSync('skey')
								},
								success(res) {
									_this.studentCancelAppointmentNoticeNum = res.data.data
								}
							})
						}
					}
				})
			},
			teacherDealAppointmentNotice(){
				let _this = this
				wx.requestSubscribeMessage({
					tmplIds: ['s508HTgz9M7ZfePv6VCj-Ji54iUbXQmGUUZw_EXrkGk'],
					success(res) {
						if (res['s508HTgz9M7ZfePv6VCj-Ji54iUbXQmGUUZw_EXrkGk'] === 'accept') {
							uni.request({
								url: baseUrl + `permission/increaseStatusCount`,
								method: 'POST',
								header: {
									'content-type': 'application/x-www-form-urlencoded'
								},
								data: {
									skey: uni.getStorageSync('skey')
								},
								success(res) {
									_this.teacherDealAppointmentNoticeNum = res.data.data
								}
							})
						}
					}
				})
				
			}
		},
		computed: {
			isLogin: function() {
				if (this.avatarUrl.length === 0 || this.name.length === 0) {
					return false
				} else {
					return true
				}
			}
		},
		onLoad() {
			this.isTeacher = uni.getStorageSync('isTeacher')
			let _this = this
			let skey = uni.getStorageSync('skey')
			uni.request({
				url:baseUrl + `permission/getPermission?skey=${skey}`,
				method:'GET',
				success(res) {
					_this.teacherDealAppointmentNoticeNum = res.data.sendStatusCount
					_this.studentAppointmentNoticeNum = res.data.sendAppointmentCount
					_this.studentCancelAppointmentNoticeNum = res.data.sendCancelCount
				}
			})

		}
	}
</script>

<style>
	.page {
		box-sizing: border-box;
		padding-left: 16rpx;
		padding-right: 16rpx;
		padding-top: 20rpx;
	}

	.loginPanel,
	.avatarAndName {
		padding-left: 16rpx;
		padding-right: 16rpx;
		border-radius: 16rpx;
		display: flex;
		justify-content: space-between;
		height: 180rpx;
		background-color: #FFFFFF;
	}

	.avatar {
		align-self: center;
		height: 110rpx;
		width: 110rpx;
		border-radius: 50%;
	}

	.avatar>image {
		height: 100%;
		width: 100%;
		border-radius: 50%;
	}

	.loginTxt {
		align-self: center;
		font-weight: 500;
		font-family: 'Trebuchet MS', 'Lucida Sans Unicode', 'Lucida Grande', 'Lucida Sans', Arial, sans-serif;
		font-size: 38rpx;
		margin-left: 24rpx;
	}

	.nameTxt {
		align-self: center;
		font-family: 'Trebuchet MS', 'Lucida Sans Unicode', 'Lucida Grande', 'Lucida Sans', Arial, sans-serif;
		font-size: 30rpx;
		margin-left: 24rpx;
	}

	.toLoginArrow {
		align-self: center;
		height: 32rpx;
		width: 32rpx;
	}

	.toLoginArrow>image {
		height: 100%;
		width: 100%;
	}

	.myRecordList {
		height: 90rpx;
		display: flex;
		justify-content: space-between;
		padding-left: 16rpx;
		padding-right: 16rpx;
		border-radius: 16rpx;
		margin-top: 16rpx;
		background-color: #FFFFFF;
	}

	.fontStyle {
		align-self: center;
		font-size: 32rpx;
		margin-left: 16rpx;
	}

	.imgStyle {
		align-self: center;
		height: 40rpx;
		width: 40rpx;
	}

	.imgStyle>image {
		height: 100%;
		width: 100%;
	}

	.arrowStyle {
		height: 24rpx;
		width: 24rpx;
		align-self: center;
	}

	.arrowStyle>image {
		height: 100%;
		width: 100%;
	}

	.noticeInfo {
		position: relative;
		transition: height 0.4s linear;
		overflow: hidden;
		padding-left: 72rpx;
		padding-right: 32rpx;

	}


	.showNoticeContentStyle {
		height: 450rpx;
	}

	.hideNoticeContentStyle {
		height: 0rpx;
	}

	.noticeNumAndBtn {
		display: flex;
		justify-content: space-between;
		border: none;
		border-bottom: 1upx rgba(0, 0, 0, 0.1) solid;
		height: 110rpx;
	}

	.noticeNumAndBtn>view {
		align-self: center;
	}

	.recordBox {
		background-color: #FFFFFF;
		border-radius: 16rpx;
	}

	.noticeNum {
		font-size: 30rpx;
	}
	

	.noticeBtn {
		width: 130rpx;
		height: 50rpx;
		font-size: 24rpx;
		padding: 2rpx;
	}

	.noticeAlertTxt {
		text-align: left;
		position: absolute;
		bottom: 32rpx;
	}

	.noticeAlertTxt :nth-child(1) {
		font-size: 24rpx;
		color: #8F8F94;
	}

	.noticeAlertTxt :nth-child(2) {
		font-size: 24rpx;
		color: #8F8F94;
		margin-top: 8rpx;
	}


	.showNoticeArrowStyle {
		transform: rotate(90deg);
	}

	.rotateStyle {
		transition: transform 0.2s linear;
	}
</style>
