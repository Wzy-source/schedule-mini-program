<template>
	<view class="page">
		<view class="logo">
			<image src="https://se-2-bighomework.oss-cn-hangzhou.aliyuncs.com/images/logo.png?OSSAccessKeyId=LTAI5t9BkdFHRepa7WyHJkpZ&Expires=3777691577&Signature=lLIs8wcjJM90hJ0BVDttm6UV6bU%3D" mode="scaleToFill"></image>
		</view>
		<view class="name">菲姐陪你唠</view>
		<view class="btn">
			<button class="cu-btn round bg-gradual-blue" @tap="getUserProfile">微信登陆</button>
		</view>
	</view>
</template>

<script>
	import baseUrl from '../../url.js'
	export default {
		data() {
			return {
				code: ''
			}
		},
		methods: {
			onLogin() {
				uni.login({
					provider: 'weixin',
					method: 'POST',
					success(loginRes) {
						uni.request({
							url: baseUrl + 'user/login',
							method: 'POST',
							header: {
								'content-type': 'application/x-www-form-urlencoded'
							},
							data: {
								code: loginRes.code
							},
							success(res) {
								if (res.data.code === 200) {
									uni.setStorageSync('skey', res.data.skey)
									uni.request({//判断是不是老师
										url:baseUrl+'user/getInformation?skey='+uni.getStorageSync('skey'),
										success(res) {
											if(res.data.code===200){
												uni.setStorageSync('isTeacher',res.data.data.isTeacher);
											}
										}
									})
								} else {
									console.log('login失败')
								}
							},
							fail(err) {
								console.log('接口调用失败')
							}
						})
					}
				})
			},


			getUserProfile() {
				uni.getUserProfile({
					desc: '业务需要',
					success(res) {
						uni.setStorageSync('avatarUrl', res.userInfo.avatarUrl);
						uni.setStorageSync('nickName', res.userInfo.nickName);
						uni.navigateTo({
							url: '../authenticate/authenticate'
						})
					}
				})
			}
		},
		mounted() {
			this.onLogin()
		}
	}
</script>

<style>
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

	.logo {
		align-self: center;
		width: 120rpx;
		height: 120rpx;
		border-radius: 32rpx;
		margin-top: 160rpx;
	}
	
	.logo > image{
		width: 100%;
		height: 100%;
		border-radius: 32rpx;
	}

	.name {
		align-self: center;
		font-size: 28rpx;
		color: #8F8F94;
		margin-top: 24rpx;
		font-family: 'Trebuchet MS', 'Lucida Sans Unicode', 'Lucida Grande', 'Lucida Sans', Arial, sans-serif;
	}

	.btn {
		align-self: center;
		margin-top: 100rpx;
		width: 70%;
		height: 80rpx;
	}

	.btn>button {
		width: 100%;
		font-size: 36rpx;
		height: 100%;
		font-weight: 300;
		font-family: 'Trebuchet MS', 'Lucida Sans Unicode', 'Lucida Grande', 'Lucida Sans', Arial, sans-serif;
		box-shadow: 0px 6rpx 6rpx rgba(0, 0, 0, 0.3);
	}
</style>
