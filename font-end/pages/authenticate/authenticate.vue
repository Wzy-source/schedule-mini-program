<template>
	<view>
		<view class="page">
			<view class="avatar">
				<image :src="avatarUrl" mode="scaleToFill"></image>
			</view>
			<view class="nickname">{{nickName}},请进行实名认证</view>
			<view class="form">
				<view class="inputStyle">
					<input placeholder="姓名" type="text" v-model="name"></input>
				</view>
				<view class="inputStyle">
					<input placeholder="学工号" type="text" v-model="studentId"></input>
				</view>
			</view>
			<view class="weekPicker">
				<view>当前周数</view>
				<picker mode="selector" :range="weekSelectList" @change="onValueChange">
					<view style="color: #5ab2b3;">{{currentWeek}}</view>
				</picker>
			</view>
			<view class="toIndexBtn">
				<button class="cu-btn round bg-gradual-blue" @tap="certification">确认</button>
			</view>
		</view>
	</view>
</template>

<script>
	//TODO 成功输入姓名和学号后，在输入框后面显示一个“成功”图标（要使用color ui另一个输入框样式）
	import baseUrl from '../../url.js';
	export default {
		data() {
			return {
				nickName: '',
				avatarUrl: '',
				name: '',
				studentId: '',
				weekSelectList: ['第一周', '第二周', '第三周', '第四周', '第五周', '第六周', '第七周', '第八周', '第九周', '第十周', '第十一周', '第十二周',
					'第十三周', '第十四周', '第十五周', '第十六周', '第十七周', '第十八周', '第十九周', '第二十周'
				],
				currentWeek: '请选择'

			}
		},
		methods: {
			certification() {
				let _this = this;
				uni.request({
					url: baseUrl + 'user/certification',
					method: 'POST',
					header: {
						'content-type': 'application/x-www-form-urlencoded'
					},
					data: {
						name: this.name,
						studentId: this.studentId,
						skey: uni.getStorageSync('skey'),
						avatar: this.avatarUrl
					},
					success(res) {
						uni.setStorageSync('name', _this.name);
						uni.setStorageSync('studentId', _this.studentId)
						if (res.data.code === 200) {
							uni.switchTab({
								url: '/pages/index/index',
								success() {
									/* 强制刷新页面的办法 */
									setTimeout(function() {
										let page = getCurrentPages().pop();
										if (page == undefined || page == null) {
											return;
										}
										page.onLoad();
									}, 200)
								}
							})

						}
					}
				})
			},
			onValueChange(e) {
				let index = e.detail.value
				this.currentWeek = this.weekSelectList[index]
				uni.setStorageSync('currentWeek',index)
			}
		},

		mounted() {
			this.nickName = uni.getStorageSync('nickName');
			this.avatarUrl = uni.getStorageSync('avatarUrl')
		}


	}
</script>

<style scoped>
	page {
		background-color: #f0f0f0;
	}

	.page {
		width: 100%;
		padding: 32rpx;
		box-sizing: border-box;
		display: flex;
		flex-direction: column;
	}

	.avatar {
		align-self: center;
		width: 100rpx;
		height: 100rpx;
		margin-top: 120rpx;
		border-radius: 32rpx;
	}

	.avatar>image {
		width: 100%;
		height: 100%;
		border-radius: 32rpx;
	}

	.nickname {
		align-self: center;
		margin-top: 32rpx;
		font-size: 36rpx;
		color: #8F8F94;
		font-family: 'Trebuchet MS', 'Lucida Sans Unicode', 'Lucida Grande', 'Lucida Sans', Arial, sans-serif;
		font-weight: 300;
	}

	.form {
		margin-top: 32rpx;
		display: flex;
		flex-direction: column;
	}

	.inputStyle {
		align-self: center;
		width: 80%;
		margin-bottom: 32rpx;
		border: none;
		border-bottom: #1296db 2rpx solid;
	}

	.inputStyle>input {
		height: 100rpx;
		padding-left: 40rpx;
		font-size: 32rpx;
	}

	.toIndexBtn {
		align-self: center;
		margin-top: 80rpx;
		width: 60%;
		height: 80rpx;
	}

	.toIndexBtn>button {
		width: 100%;
		font-size: 36rpx;
		height: 100%;
		font-weight: 300;
		font-family: 'Trebuchet MS', 'Lucida Sans Unicode', 'Lucida Grande', 'Lucida Sans', Arial, sans-serif;
		box-shadow: 0px 6rpx 6rpx rgba(0, 0, 0, 0.3);
	}

	.weekPicker {
		display: flex;
		align-self: center;
		width: 80%;
		justify-content: space-between;
		padding-left: 36rpx;
		margin-top: 42rpx;
		border: none;
		border-bottom: #1296db 2rpx solid;
		padding-bottom: 24rpx;
	}

	.weekPicker>view {
		align-self: center;
		font-size: 32rpx;
		color: #666666;
	}

	.weekPicker>picker {
		color: #8F8F94;
		font-size: 32rpx;
		align-self: center;
		font-weight: 500;
	}
</style>
