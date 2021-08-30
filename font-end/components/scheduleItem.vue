<template>
	<view :style="{height:itemHeight+'rpx' }" @longpress="onLongPress">
		<view class="page" :style="{border:borderSize+'rpx solid #8F8F94'}" @tap="onClick">
			<view class="container" v-if="scheduleInfo.type==='teacher'" style="background-color:#88DEF9 ;">
				<!-- 这里要分老师和学生两种情况，先用老师 -->
				<view class="info">
					<view class="icon">
						<image src="../static/people/teacher.png" mode="scaleToFill"></image>
					</view>
					<view class="txt">毕菲菲</view>
					<view class="icon">
						<image src="../static/people/location.png" mode="scaleToFill"></image>
					</view>
					<view class="txt">{{scheduleInfo.place}}</view>
				</view>
			</view>

			<view class="container" v-if="scheduleInfo.type==='student'" style="background-color: #F0AD4E;">
				<!-- 这里要分老师和学生两种情况，先用老师 -->
				<view class="info">
					<view class="icon">
						<image src="../static/people/mystudent.png" mode="scaleToFill"></image>
					</view>
					<view class="txt">{{scheduleInfo.name}}</view>
					<view class="icon">
						<image src="../static/people/location.png" mode="scaleToFill"></image>
					</view>
					<view class="txt">{{scheduleInfo.place}}</view>
				</view>
			</view>
		</view>
	</view>
</template>

<script>
	//TOOD 1.学生预约的背景颜色代表预约状态（未开始，正进行，已结束）2.区分老师和学生在点击、长按后的


	import baseUrl from '../url.js'
	export default {
		props: {
			schedule: {
				type: Object,
				default: function() {
					return {
						type: '',
						place: '',
						name: '',
						content: '',
						startTime: '',
						endTime: '',
						other: ''
					}
				}
			},
			height: 0
		},
		data() {
			return {
				borderSize: 0,
			}
		},
		methods: {
			onClick() {
				this.borderSize = 2;
				setTimeout(() => {
					this.borderSize = 0
				}, 1000)
			},
			onLongPress() {
				this.borderSize = 1;
				setTimeout(() => {
					this.borderSize = 0
				}, 1000)
			},
		},
		/* 使用prop来定义计算属性，就是响应式的了 */
		computed: {
			scheduleInfo: function() {
				return this.schedule
			},
			itemHeight: function() {
				return this.height
			},
		},
	}
</script>

<style>
	.page {
		width: 100%;
		box-sizing: border-box;
		padding: 2rpx;
		height: 100%;
	}

	.container {
		border-radius: 16rpx;
		height: 100%;
		box-shadow: 0px 4rpx 4rpx rgba(0, 0, 0, 0.2);
		overflow: hidden;
		box-sizing: border-box;
		padding: 2rpx;

	}

	.info {
		display: flex;
		flex-direction: column;
	}


	.icon {
		width: 32rpx;
		height: 32rpx;
		align-self: center;
		margin-top: 8rpx;
	}

	.icon>image {
		width: 100%;
		height: 100%;
	}

	.txt {
		align-self: center;
		color: #707070;
		font-weight: 400;
		font-family: 'Trebuchet MS', 'Lucida Sans Unicode', 'Lucida Grande', 'Lucida Sans', Arial, sans-serif;
		font-size: 24rpx;
	}

	.scheduleItem {
		display: flex;
		margin-bottom: 16rpx;
	}

	.scheduleItem>image {
		width: 40rpx;
		height: 40rpx;
	}

	.scheduleTxt {
		margin-left: 16rpx;
		font-family: 'Trebuchet MS', 'Lucida Sans Unicode', 'Lucida Grande', 'Lucida Sans', Arial, sans-serif;
		font-size: 30rpx;
	}

	.createScheduleItem {
		display: flex;
		margin-bottom: 16rpx;
	}

	.createIcon {
		width: 40rpx;
		height: 40rpx;
	}

	.createIcon>image {
		height: 100%;
		width: 100%;
	}

	.createTxt {
		margin-left: 16rpx;
		font-family: 'Trebuchet MS', 'Lucida Sans Unicode', 'Lucida Grande', 'Lucida Sans', Arial, sans-serif;
		font-size: 30rpx;
	}

	.createInput {
		width: 60%;
	}

	.createInput>input {
		width: 100%;
		border: none;
		border-bottom: #1296DB solid 2rpx;
		text-align: left;
		padding-left: 8rpx;
		font-family: 'Trebuchet MS', 'Lucida Sans Unicode', 'Lucida Grande', 'Lucida Sans', Arial, sans-serif;
	}

	.arrow {
		display: flex;
		flex-direction: column;
		justify-content: space-between;
		height: 100%;
		width: 100%;
	}

	.upArrow {
		width: 40rpx;
		height: 40rpx;
		align-self: center;
	}

	.downArrow {
		width: 40rpx;
		height: 40rpx;
		align-self: center;
	}

	.upArrow>image {
		width: 100%;
		height: 100%;

	}

	.downArrow>image {
		width: 100%;
		height: 100%;
	}
</style>
