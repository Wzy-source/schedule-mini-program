<template>
	<view class="page">
		<vgtTab :list="list" @onValueChange='onTabChange'></vgtTab>
		<view class="cardContainer" v-if="isTeacher==='true'">
			<view v-if="flag===0">
				<appointment-card v-for="(item , index) in pendingList" :key='index' :appointmentStatus='2'
					:appointmentInfo="item" @tap='toFeedbackPage(item.appointmentId)'></appointment-card>
				<view class="load-progress" :class="loadProgress!=0?'show':'hide'">
					<view class="load-progress-spinner text-blue"></view>
					<view class="loadTxt text-blue">加载中</view>
					<view v-if="noMore" class="moMoreTxt">没有更多了qwq</view>
				</view>
			</view>
			<view v-else>
				<appointment-card v-for="(item , index) in accomplishList" :key='index' :appointmentStatus='3'
					:appointmentInfo="item" @tap='toFeedbackPage(item.appointmentId)'></appointment-card>
				<view class="load-progress" :class="loadProgress!=0?'show':'hide'">
					<view class="load-progress-spinner text-blue"></view>
					<view class="loadTxt text-blue">加载中</view>
					<view v-if="noMore" class="moMoreTxt">没有更多了qwq</view>
				</view>
			</view>
		</view>
		<view v-else class="cardContainer">
			<view v-if="flag===0">
				<appointment-card v-for="(item , index) in studentPendingList" :key='index' :appointmentStatus='4'
					:appointmentInfo="item" @tap='toFeedbackPage(item.appointmentId)'></appointment-card>
			</view>
			<view v-else>
				<appointment-card v-for="(item , index) in studentAccomplishList" :key='index' :appointmentStatus='5'
					:appointmentInfo="item" @tap='toFeedbackPage(item.appointmentId)'></appointment-card>
			</view>
		</view>
	</view>
</template>

<script>
	import vgtTab from '../../../components/vgt-tab/vgt-tab.vue'
	import baseUrl from '../../../url.js'
	import appointmentCard from '../../../components/appointmentCard.vue'

	export default {
		components: {
			vgtTab,
			appointmentCard
		},
		data() {
			return {
				flag: 0,
				list: [
					'待填写', '已填写'
				],
				pageSize: 5,
				pendingIndex: 1,
				accomplishIndex: 1,
				pendingList: [],
				accomplishList: [],
				loadProgress: 0,
				isTeacher: '',
				noMore: false,
				studentPendingList: [],
				studentAccomplishList: []

			}
		},
		onReachBottom(e) {
			if (this.isTeacher === 'true') {
				let _this = this
				this.loadProgress = 1;
				if (this.flag === 0) {
					this.pendingIndex++;
					uni.request({
						url: baseUrl +
							`appointment/getRecordList?pageIndex=${this.pendingIndex}&pageSize=${this.pageSize}&status=0`,
						method: 'GET',
						success(res) {
							let list = res.data.recordList;
							setTimeout(() => {
								_this.loadProgress = 0;
								for (var i = 0; i < list.length; i++) {
									_this.pendingList.push(list[i])
								}
								if(list.length===0){
									_this.pendingIndex--;
								}
							}, 1000)
						}
					})
				} else if (this.flag === 1) {
					this.accomplishIndex++;
					uni.request({
						url: baseUrl +
							`appointment/getRecordList?pageIndex=${this.accomplishIndex}&pageSize=${this.pageSize}&status=1`,
						method: 'GET',
						success(res) {
							let list = res.data.recordList;
							setTimeout(() => {
								_this.loadProgress = 0;
								for (var i = 0; i < list.length; i++) {
									_this.accomplishList.push(list[i])
								}
								if(list.length===0){
									_this.accomplishIndex--;
								}
							}, 1000)
						}
					})
				}
			}
		},
		methods: {
			onTabChange(e) {
				this.flag = e.currentInd;
				if (this.isTeacher === 'true') {
					if (this.flag === 1 && this.accomplishIndex === 1) {
						let _this = this
						uni.request({
							url: baseUrl +
								`appointment/getRecordList?pageIndex=1&pageSize=${this.pageSize}&status=1`,
							success(res) {
								_this.accomplishList = res.data.recordList
							}
						})
					}
				}
			},
			initStudentList() {
				let skey = uni.getStorageSync('skey')
				let _this = this
				uni.request({
					url: baseUrl + `appointment/getStudentFeedbackByStatus?skey=${skey}&status=1`,
					success(res) {
						_this.studentAccomplishList = res.data
					}
				})
				uni.request({
					url: baseUrl + `appointment/getStudentFeedbackByStatus?skey=${skey}&status=0`,
					success(res) {
						_this.studentPendingList = res.data
					}
				})
			},
			toFeedbackPage(id) {
				uni.navigateTo({
					url: 'feedbackItem?id=' + id
				})
			},
			initPendingList() {
				let _this = this
				uni.request({
					url: baseUrl + `appointment/getRecordList?pageIndex=1&pageSize=${this.pageSize}&status=0`,
					method: "GET",
					success(res) {
						_this.pendingList = res.data.recordList;
					}
				})
			},
		},
		onLoad() {
			this.isTeacher = uni.getStorageSync('isTeacher')
			if (this.isTeacher === 'true') {
				this.initPendingList()
			} else {
				this.initStudentList()
			}
		},
	}
</script>

<style>
	.cardContainer {
		padding: 16rpx;
	}

	.moMoreTxt {
		font-size: 26rpx;
		margin-top: 32rpx;
		text-align: center;
		color: #8F8F94;

	}
</style>
