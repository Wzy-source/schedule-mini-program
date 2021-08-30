<template>
	<view class="page">
		<!-- content              - status -->
		<!-- time -->
		<!-- place -->
		<!-- avatar name id -->
		<!-- 确认/拒绝 -->
		<view class="contentAndStatus">
			<view class="content">
				{{info.content}}
			</view>
			<!-- 根据状态显示不同颜色 -->
			<view class="status" :style="{color:statusColorAndTxt.color}">
				{{statusColorAndTxt.txt}}
			</view>
		</view>
		<view class="time">
			{{info.date}} {{info.startTime}}-{{info.endTime}}
		</view>
		<view class="place">
			{{info.place}}
		</view>
		<view class="idCard">
			<view class="avatar">
				<image :src="info.avatar" mode="scaleToFill"></image>
			</view>
			<view class="nameAndId">
				<view class="nameAndIdTxt">{{info.name}} {{info.studentId}}</view>
				<view class="nameAndIdArrow">
					<image src="../static/operation/arrow.png" mode="scaleToFill"></image>
				</view>
			</view>
		</view>
		<view v-if="status===0&&isTeacher==='true'">
			<view class='solid-top paddingTop'>
				<view style="color: #E54D42;" @tap='onTap(-1)' @tap.stop='onTap'>拒绝</view>
				<view style="color: #E6E6E6; font-size: 40rpx;" class="borderLeft"></view>
				<view style="color: #5ab2b3;" @tap='onTap(1)' @tap.stop='onTap'>同意</view>
			</view>
		</view>
	</view>
</template>

<script>
	export default {
		props: {
			appointmentInfo: {
				type: Object,
				default: function() {
					return {
						appointmentId: 0,
						name: '',
						studentId: '',
						date: '',
						startTime: '',
						endTime: '',
						place: '',
						content: '',
						avatar: ''
					}
				}
			},
			appointmentStatus: {
				type: Number,
				default: 0
			},
			index: {
				type: Number,
				default: 0
			}
		},
		computed: {
			info: function() {
				return this.appointmentInfo
			},
			status: function() {
				return parseInt(this.appointmentStatus)
			},
			statusColorAndTxt: function() {
				if (this.status === 1) {
					return {
						color: '#5ab2b3',
						txt: '已同意'
					}
				} else if (this.status === 0) {
					return {
						color: '#0081FF',
						txt: '待处理'
					}
				} else if (this.status === -1) {
					return {
						color: '#E54D42',
						txt: '已拒绝'
					}
				} else if (this.status === 2) { //待填写
					return {
						color: '#0081FF',
						txt: '待填写'
					}
				} else if (this.status === 3) { //已填写
					return {
						color: '#5ab2b3',
						txt: '已填写',
					}
				} else if (this.status === 4) { //待反馈
					return {
						color: '#0081FF',
						txt: '待反馈'
					}
				} else if (this.status === 5) {
					return {
						color: '#5ab2b3',
						txt: '已反馈',
					}
				}
			}
		},
		methods: {
			onTap(optNum) {
				this.$emit('acceptOrRefuseAppointment', optNum, this.info.appointmentId, this.index)
			},
		},
		data() {
			return {
				isTeacher:uni.getStorageSync('isTeacher')
			}
		},
	}
</script>

<style>
	.page {
		box-sizing: border-box;
		padding: 24rpx 28rpx 20rpx 28rpx;
		border-radius: 16rpx;
		background-color: #FFFFFF;
		margin-bottom: 16rpx;
	}

	.contentAndStatus {
		display: flex;
		justify-content: space-between;
	}

	.content {
		align-self: center;
		font-size: 40rpx;
	}

	.time {
		font-size: 28rpx;
		color: #666666;
		margin-top: 16rpx;
	}

	.place {
		box-sizing: border-box;
		padding: 4rpx 6rpx;
		background-color: #F5F5F5;
		color: #666666;
		width: fit-content;
		border-radius: 8rpx;
		font-size: 26rpx;
		margin-top: 16rpx;
	}

	.idCard {
		display: flex;
		height: 100rpx;
		margin-top: 8rpx;
		margin-bottom: 4rpx;
	}

	.status {
		align-self: center;
		font-size: 30rpx;
	}

	.avatar {
		align-self: center;
		height: 64rpx;
		min-width: 64rpx;
		border-radius: 50%;
	}

	.avatar>image {
		height: 100%;
		width: 100%;
		border-radius: 50%;
	}

	.nameAndId {
		align-self: center;
		margin-left: 32rpx;
		font-size: 28rpx;
		display: flex;
		justify-content: space-between;
		width: 100%;
	}

	.nameAndIdTxt,
	.nameAndIdArrow {
		align-self: center;
	}

	.nameAndIdArrow {
		height: 26rpx;
		width: 26rpx;
	}

	.nameAndIdArrow>image {
		height: 100%;
		width: 100%;
	}

	.paddingTop {
		padding-top: 18rpx;
		display: flex;
		justify-content: space-around;
		font-size: 36rpx;
	}
	
	.borderLeft{
		border: none;
		border-left: 1upx solid rgba(0,0,0,0.1);
	}
</style>
