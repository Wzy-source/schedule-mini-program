<template>
	<view>
		<view class="page">
			<view class="appointmentStatus">
				<view style="font-size: 40rpx;">预约详情</view>
				<view class="statusTxt" :style="{color:statusTxtAndStyle.color}">
					{{statusTxtAndStyle.txt}}
				</view>
			</view>
			<view class="timeAndPlace">
				<view class="time">
					<view class="timeIcon">
						<image src="../../../static/feedbackIcon/time.png" mode="scaleToFill"></image>
					</view>
					<view class="timeTxt">
						{{recordInfo.date}} {{recordInfo.startTime}}-{{recordInfo.endTime}}
					</view>
				</view>
				<view class="place">
					<view class="placeIcon">
						<image src='../../../static/feedbackIcon/place.png' mode="scaleToFill"></image>
					</view>
					<view class="placeTxt">
						{{recordInfo.place}}
					</view>
				</view>
			</view>
			<view class="avatarAndName">
				<view style="align-self: center;display: flex;">
					<view class="avatar">
						<image :src="recordInfo.avatar" mode="scaleToFill"></image>
					</view>
					<view class="nameAndId">
						<view>{{recordInfo.name}}</view>
						<view style="margin-top: 8rpx;">{{recordInfo.studentId}}</view>
					</view>
				</view>
				<view class="arrow">
					<image src='../../../static/operation/arrow.png' mode="scaleToFill"></image>
				</view>
			</view>
			<view class="content">
				<view class="headerFontStyle">
					<view>学生诉求</view>
					<view class="headerImg">
						<image src="../../../static/feedbackIcon/plane.png" mode="scaleToFill"></image>
					</view>
				</view>
				<view class="contentTxt">
					内容：{{recordInfo.content}}
				</view>
				<view class="otherTxt">
					其他请求：{{recordInfo.other.length===0? '无' : recordInfo.other}}
				</view>
			</view>
			<view class="teacherLayout" v-if="isTeacher==='true'">
				<view class="studentFeedback">
					<view class="headerFontStyle">
						<view>学生反馈</view>
						<view class="headerImg">
							<image src="../../../static/feedbackIcon/flower.png" mode="scaleToFill"></image>
						</view>
					</view>
					<view class="feedback">
						<view v-if="feedbackInfo.feedback===null">
							<view class="noFeedbackAlert">当前学生暂未反馈,待填写后会在此显示</view>
						</view>
						<view v-else style="padding: 16rpx;">
							<view class="hasFeedback">
								{{feedbackInfo.feedback}}
							</view>
						</view>
					</view>
				</view>
				<view class="teacherRecord">
					<view class="headerFontStyle">
						<view>教师记录</view>
						<view class="headerImg">
							<image src="../../../static/feedbackIcon/edit.png" mode="scaleToFill"></image>
						</view>
					</view>
					<view class="record">
						<view class="cu-form-group recordArea">
							<textarea maxlength="-1" v-model="recordTxt" placeholder="请在此输入本次谈话记录"
								auto-height style="border-radius: 16rpx;"></textarea>
						</view>
						
						<view class="editTime" v-if="recordInfo.createTime!==null">
							最新编辑于：{{recordInfo.createTime}}
						</view>
						<view class="margin-tb-sm text-right" style="margin-right: 40rpx;">
							<button class="cu-btn round shadow bg-gray"
								@tap='onTapBtn'>{{teacherUpdateOrEditTxt}}</button>
						</view>
					</view>
				</view>
			</view>

			<!-- 学生布局 -->
			<view v-else class="studentLayout">
				<view class="teacherRecord">
					<view class="headerFontStyle">
						<view>教师记录</view>
						<view class="headerImg">
							<image src="../../../static/feedbackIcon/edit.png" mode="scaleToFill"></image>
						</view>
					</view>
					<view class="feedback">
						<view v-if="recordInfo.record===null">
							<view class="noFeedbackAlert">当前老师暂未填写记录,待填写后会在此显示</view>
						</view>
						<view v-else style="padding: 16rpx;">
							<view class="hasFeedback">
								{{recordInfo.record}}
							</view>
						</view>
					</view>
				</view>
				<view class="studentFeedback">
					<view class="headerFontStyle">
						<view>学生反馈</view>
						<view class="headerImg">
							<image src="../../../static/feedbackIcon/flower.png" mode="scaleToFill"></image>
						</view>
					</view>
					<view class="record">
						<view class="cu-form-group recordArea">
							<textarea maxlength="-1" v-model="feedbackTxt" placeholder="请在此输入本次谈话反馈"
								auto-height style="	border-radius: 16rpx;"></textarea>
						</view>
						<view class="editTime" v-if="feedbackInfo.createTime!==null">
							最新编辑于：{{feedbackInfo.createTime}}
						</view>
						<view class="margin-tb-sm text-right" style="margin-right: 40rpx;">
							<button class="cu-btn round shadow bg-gray"
								@tap='onTapBtn'>{{studentUpdateOrEditTxt}}</button>
						</view>
					</view>
				</view>
			</view>
		</view>
		<!-- 反馈内容为空，则弹出模态框 -->
		<view class="cu-modal" :class="modalName=='Modal'?'show':''">
			<view class="cu-dialog">
				<view class="cu-bar bg-white justify-end">
					<view class="content">温馨提示</view>
					<view class="action" @tap="hideModal">
						<text class="cuIcon-close text-red"></text>
					</view>
				</view>
				<view class="padding-xl" style="font-size: 30rpx">
					{{alertTxt}}
				</view>
			</view>
		</view>
		<!-- 修改成功的提示框 -->
		<view class="cu-modal" :class="modalName=='successEditModal'?'show':''">
			<view class="cu-dialog">
				<view class="cu-bar bg-white justify-end">
					<view class="content">温馨提示</view>
					<view class="action" @tap="hideModal">
						<text class="cuIcon-close text-red"></text>
					</view>
				</view>
				<view class="padding-xl" style="font-size: 30rpx">
					{{successEditTxt}}
				</view>
			</view>
		</view>
	</view>
</template>

<script>
	import baseUrl from '../../../url.js'
	export default {
		data() {
			return {
				isTeacher: '',
				appointmentId: 0,
				recordInfo: {
					avatar: '',
					content: '',
					createTime: '',
					date: '',
					startTime: '',
					endTime: '',
					name: '',
					studentId: '',
					other: '',
					place: '',
					record: ''
				},
				feedbackInfo: {
					avatar: '',
					content: '',
					date: '',
					createTime: '',
					name: '',
					startTime: '',
					studentId: '',
					endTime: '',
					other: '',
					place: '',
					feedback: ''
				},
				recordTxt: '', //谈话记录输入框
				feedbackTxt: '', //反馈输入框
				teacherUpdateOrEditTxt: '提交',
				studentUpdateOrEditTxt: '提交',
				alertTxt: '',
				modalName: '',
				successEditTxt:''

			}
		},
		methods: {
			getRecord() {
				let _this = this
				uni.request({
					url: baseUrl + `appointment/getRecord?appointmentId=${this.appointmentId}`,
					method: 'GET',
					success(res) {
						_this.recordInfo = res.data
						_this.recordTxt = _this.recordInfo.record === null ? '' : _this.recordInfo.record
						if (_this.recordTxt !== '') {
							_this.teacherUpdateOrEditTxt = '修改'
						}
					}
				})
			},
			getFeedBack() {
				let _this = this
				uni.request({
					url: baseUrl + `appointment/getFeedback?appointmentId=${this.appointmentId}`,
					method: 'GET',
					success(res) {
						_this.feedbackInfo = res.data
						console.log(res.data)
						_this.feedbackTxt = _this.feedbackInfo.feedback === null ? '' : _this.feedbackInfo.feedback
						if (_this.feedbackTxt !== '') {
							_this.studentUpdateOrEditTxt = '修改'
						}
					}
				})
			},
			hideModal() {
				this.modalName = ''
			},
			onTapBtn() {
				let _this = this
				if (this.isTeacher === 'true') {
					if (this.recordTxt.length === 0) {
						this.modalName = 'Modal'
						this.alertTxt = '谈话记录不能为空'
					} else {
						uni.request({
							url: baseUrl + 'appointment/record',
							method: 'POST',
							header: {
								'content-type': 'application/x-www-form-urlencoded'
							},
							data: {
								appointmentId: this.appointmentId,
								record: this.recordTxt
							},
							success(res) {
								_this.successEditTxt = '谈话记录'+_this.teacherUpdateOrEditTxt +'成功'
								_this.modalName = 'successEditModal'
								_this.getRecord()
							}
						})
					}
				} else if (this.isTeacher === 'false') {
					if (this.feedbackTxt.length === 0) {
						this.modalName = 'Modal'
						this.alertTxt = '反馈内容不能为空'
					} else {
						uni.request({
							url: baseUrl + 'appointment/feedback',
							method: 'POST',
							header: {
								'content-type': 'application/x-www-form-urlencoded'
							},
							data: {
								appointmentId: this.appointmentId,
								skey: uni.getStorageSync('skey'),
								feedback: this.feedbackTxt
							},
							success(res) {
								_this.successEditTxt = '反馈'+_this.studentUpdateOrEditTxt+ '成功'
								_this.modalName = 'successEditModal'
								_this.getFeedBack()
							}
						})
					}
				}
			}
		},
		onLoad(options) {
			this.appointmentId = parseInt(options.id);
			this.isTeacher = uni.getStorageSync('isTeacher')
			this.getFeedBack();
			this.getRecord()
		},
		onUnload() {
			//回退刷新的方法
			let pages = getCurrentPages()
			let beforePage = pages[pages.length - 2]
			beforePage.onLoad()
		},
		computed: {
			statusTxtAndStyle: function() {
				//TODO 这里要分为两种情况：学生的反馈和谈话记录的createTime
				if (this.isTeacher === 'true') {
					if (this.recordInfo.createTime === null) {
						return {
							txt: '待填写',
							color: '#0081FF'
						}
					} else if (this.recordInfo.createTime !== null) {
						return {
							txt: '已填写',
							color: '#5ab2b3'
						}
					}
				} else { //学生
					if (this.feedbackInfo.createTime === null) {
						return {
							txt: '待反馈',
							color: '#0081FF'
						}
					} else if (this.feedbackInfo.createTime !== null) {
						return {
							txt: '已反馈',
							color: '#5ab2b3'
						}
					}
				}
			},
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

	.appointmentStatus {
		display: flex;
		justify-content: space-between;
	}

	.appointmentStatus>view {
		align-self: center;
	}

	.statusTxt {
		font-weight: 500;
		font-size: 32rpx;
	}


	.timeAndPlace {
		display: flex;
		justify-content: space-between;
		margin-top: 32rpx;
		color: #666666;
	}

	.time,
	.place {
		display: flex;
		align-self: center;
	}

	.timeIcon,
	.placeIcon {
		align-self: center;
		height: 52rpx;
		width: 52rpx;
	}

	.timeIcon>image,
	.placeIcon>image {
		height: 100%;
		width: 100%;
	}

	.timeTxt,
	.placeTxt {
		align-self: center;
		margin-left: 8rpx;
		font-size: 30rpx;
	}

	.avatarAndName {
		margin-top: 32rpx;
		height: 160rpx;
		padding: 20rpx 8rpx;
		display: flex;
		border: none;
		border-top: 1upx solid rgba(0, 0, 0, 0.1);
		border-bottom: 1upx solid rgba(0, 0, 0, 0.1);
		justify-content: space-between;
	}

	.avatar {
		align-self: center;
		height: 84rpx;
		width: 84rpx;
		border-radius: 50%;
	}

	.avatar>image {
		height: 100%;
		width: 100%;
		border-radius: 50%;
	}

	.nameAndId {
		align-self: center;
		margin-left: 16rpx;
		font-size: 30rpx;
		color: #666666;
	}

	.arrow {
		align-self: center;
		height: 26rpx;
		width: 26rpx;
	}

	.arrow>image {
		height: 100%;
		width: 100%;
	}

	.content,
	.studentFeedback,
	.teacherRecord {
		margin-top: 32rpx;
	}

	.headerFontStyle {
		font-weight: 500;
		font-size: 36rpx;
		display: flex;
	}

	.headerFontStyle>view {
		align-self: center;
	}

	.headerImg {
		height: 50rpx;
		width: 50rpx;
		margin-left: 2rpx;
	}

	.headerImg>image {
		height: 100%;
		width: 100%;
	}

	.contentTxt {
		margin-top: 32rpx;
		font-size: 32rpx;
		margin-left: 16rpx;
	}

	.hasFeedback {
		font-size: 30rpx;
		padding: 16rpx;
		background-color: #F8F8F8;
		min-height: 50rpx;
		border-radius: 16rpx;
	}

	.otherTxt {
		margin-top: 16rpx;
		font-size: 26rpx;
		color: #8F8F94;
		margin-left: 16rpx;
	}

	.feedback {
		margin-top: 32rpx;
	}

	.noFeedbackAlert {
		text-align: center;
		font-size: 26rpx;
		color: #8F8F94;
	}

	.record {
		position: relative;
	}


	.recordArea>textarea {
		background-color: #F8F8F8;
		padding: 16rpx;
		font-size: 30rpx;
		min-height: 200rpx;
	}

	.editTime {
		font-size: 24rpx;
		color: #8F8F94;
		position: absolute;
		left: 32rpx;
	}
</style>
