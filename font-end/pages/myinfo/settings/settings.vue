<template>
	<view>
		<view class="page">
			<view class="headerTxt">设置</view>
			<view class="settingsContainer">
				<view class="settingsItem">
					<view class="itemKey">修改当前周数</view>
					<picker mode="selector" :range="weekSelectList" :value='selectorIndex' @change='onPickerChange'>
						<view class="pickerFontStyle">
							<view class="itemValue">
								<text>当前周数：</text>
								<text style="color: #0081FF;">{{currentWeekTxt}}</text>
							</view>
						</view>
					</picker>

				</view>
			</view>

		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				weekSelectList: ['第一周', '第二周', '第三周', '第四周', '第五周', '第六周', '第七周', '第八周', '第九周', '第十周', '第十一周', '第十二周',
					'第十三周', '第十四周', '第十五周', '第十六周', '第十七周', '第十八周', '第十九周', '第二十周'
				],
				selectorIndex: 0,
				currentWeekTxt: ''
			}
		},
		methods: {
			onPickerChange(event) {
				let index = event.detail.value
				this.currentWeekTxt = this.weekSelectList[index]
				//TODO 
			},
			setCurrentWeek() {
				//TODO 使用
				

			}
		},
		onLoad() {
			this.setCurrentWeek()
		}
	}
</script>

<style>
	page {
		background-color: #FFFFFF;
	}

	.page {
		box-sizing: border-box;
		padding: 24rpx 32rpx;
	}

	.headerTxt {
		font-weight: 500;
		font-size: 40rpx;
	}

	.settingsContainer {
		margin-top: 36rpx;
	}

	.settingsItem {
		height: 80rpx;
		display: flex;
		justify-content: space-between;
		padding-left: 16rpx;
		padding-right: 16rpx;
		border: none;
		border-bottom: 1upx solid rgba(0, 0, 0, 0.1);
		background-color: #FFFFFF;
	}

	.settingsItem>picker {
		align-self: center;
	}

	.itemKey {
		align-self: center;
		font-size: 34rpx;
	}

	.itemValue {
		font-size: 30rpx;
		color: #8F8F94;
		align-self: center;
		font-family: 'Trebuchet MS', 'Lucida Sans Unicode', 'Lucida Grande', 'Lucida Sans', Arial, sans-serif;
	}
</style>
