<template>
	<view>
		<view class="content">
			<scroll-view class="tab" id="tab" :show-scrollbar="false" :scroll-x="true"
				:scroll-into-view="tabScrollInto">
				<view class="tabContent" style="display: flex; flex-direction: column;">
					<view style="display: flex;flex-direction: row;">
						<view class="tabItem" v-for="(tab,tabItemIndex) in tabList" :key="tab.id" :id="tab.id"
							:data-id="tabItemIndex" :data-current="tabItemIndex" @click="pressScrollViewItem">
							<text class="tabItemTitle" :class="tabIndex==tabItemIndex ? 'tabItemTitleActive' : ''"
								:style="{width:tabList[tabItemIndex].width + 'px'}">{{tab.title}}</text>
						</view>
					</view>
				</view>
			</scroll-view>
			<swiper class="childPageView" :current="tabIndex" :duration="250" @change="swiperChange"
				@animationfinish="swiperChangeEnd" @onAnimationEnd="swiperChangeEnd"
				:style="{height: swiperHeight+'px'}" style="padding-bottom: 16rpx;">
				<swiper-item class="childPageViewItem" v-for="(page, childPageViewItemIndex) in tabList"
					:key="childPageViewItemIndex">
					<scroll-view scroll-y="true" :style="{height: swiperHeight+'px'}">

						<view v-if="childPageViewItemIndex===0&&isTeacher==='true'" class="container">
							<appointment-card @tap='toFeedbackItem(item.appointmentId)'
								v-for="(item,index) in pendingList" :key='index'
								@acceptOrRefuseAppointment='acceptOrRefuse' appointmentStatus='0'
								:appointmentInfo='item'></appointment-card>
						</view>
						<!-- 学生0 -->
						<view v-if="childPageViewItemIndex===0&&isTeacher==='false'" class="container">
							<appointment-card @tap='toFeedbackItem(item.appointmentId)'
								v-for="(item,index) in studentPendingList" :key='index' appointmentStatus="0"
								:appointmentInfo="item"></appointment-card>
						</view>
						<!-- 教师1 -->
						<view v-if="childPageViewItemIndex===1&&isTeacher==='true'" class="container">
							<appointment-card @tap='toFeedbackItem(item.appointmentId)'
								v-for="(item,index) in acceptList" :key='index' appointmentStatus='1'
								:appointmentInfo='item'></appointment-card>
							<view class="load-progress" :class="loadProgress!=0?'show':'hide'">
								<view class="load-progress-spinner text-blue"></view>
								<view class="loadTxt text-blue">加载中</view>
							</view>
						</view>
						<!-- 学生1 -->
						<view v-if="childPageViewItemIndex===1&&isTeacher==='false'" class="container">
							<appointment-card @tap='toFeedbackItem(item.appointmentId)'
								v-for="(item,index) in studentAcceptList" :key='index' appointmentStatus='1'
								:appointmentInfo='item'></appointment-card>
						</view>
						<!-- 教师-1 -->
						<view v-if="childPageViewItemIndex===2&&isTeacher==='true'" class="container">
							<appointment-card @tap='toFeedbackItem(item.appointmentId)'
								v-for="(item,index) in refuseList" :key='index' appointmentStatus='-1'
								:appointmentInfo='item'></appointment-card>
							<view class="load-progress" :class="loadProgress!=0?'show':'hide'">
								<view class="load-progress-spinner text-blue"></view>
								<view class="loadTxt text-blue">加载中</view>
							</view>
						</view>
						<!-- 学生-1 -->
						<view v-if="childPageViewItemIndex===2&&isTeacher==='false'" class="container">
							<appointment-card @tap='toFeedbackItem(item.appointmentId)'
								v-for="(item,index) in studentRefuseList" :key='index' appointmentStatus='-1'
								:appointmentInfo='item'></appointment-card>
						</view>
					</scroll-view>
				</swiper-item>
			</swiper>
		</view>
		<!-- 提示登录的模态框 -->
		<view class="modal">
			<view class="cu-modal" :class="modalName=='DialogModal1'?'show':''">
				<view class="cu-dialog">
					<view class="cu-bar bg-white justify-end">
						<view class="content">温馨提示</view>
						<view class="action" @tap="hideModal">
							<text class="cuIcon-close text-red"></text>
						</view>
					</view>
					<view class="padding-xl">
						请进行登陆和实名认证
					</view>
					<view class="cu-bar bg-white justify-end">
						<view class="action">
							<button class="cu-btn line-blue text-blue" @tap="hideModal">取消</button>
							<button class="cu-btn bg-gradual-blue margin-left" @tap="toLogin">确定</button>
						</view>
					</view>
				</view>
			</view>
		</view>
	</view>
</template>

<script>
	import appointmentCard from '../../components/appointmentCard.vue'
	import baseUrl from '../../url.js'
	export default {
		components: {
			appointmentCard
		},
		data() {
			return {
				tabList: [{
					id: "tabItem0",
					title: '未处理',
					width: 80,
				}, {
					id: "tabItem1",
					title: '已同意',
					width: 80,
				}, {
					id: "tabItem2",
					title: '已拒绝',
					width: 80,
				}],
				modalName: '',
				tabIndex: 0,
				tabScrollInto: "",
				tabListSize: {},
				tabItemLineLeft: 0,
				tabItemLineWidth: 0,
				isPress: false,
				swiperHeight: 0,

				pendingIndex: 1,
				acceptIndex: 1,
				refuseIndex: 1,
				pageSize: 5,
				pendingList: [],
				acceptList: [],
				refuseList: [],
				pendingTotalNum: 0,
				acceptTotalNum: 0,
				refuseTotalNum: 0,
				//下拉加载相关
				loadProgress: 0,
				isTeacher: '',
				//学生相关
				studentRefuseList: [],
				studentAcceptList: [],
				studentPendingList: []

			}
		},
		
		//点击当前tabbar下的tab触发的刷新事件
		onTabItemTap(e){
			if (this.isTeacher === 'true') {
				this.getInitPage();
			} else {
				this.getStudentInitPage();
			}
		},
		onLoad() {
			let avatarUrl = uni.getStorageSync('avatarUrl');
			let nickName = uni.getStorageSync('nickName');
			let name = uni.getStorageSync('name')
			if (nickName.length === 0 || avatarUrl.length === 0 || name.length === 0) { //没有本地存储为新用户，请先登录
				this.modalName = 'DialogModal1'
			}
			let system = uni.getSystemInfoSync();
			this.swiperHeight = system.windowHeight - 40;
		},
		onReachBottom(e) {
			if (this.isTeacher === 'true') {
				let _this = this
				this.loadProgress = 1;
				if (this.tabIndex === 1) {
					this.acceptIndex++;
					uni.request({
						url: baseUrl +
							`appointment/getAppointmentByStatus?pageIndex=${this.acceptIndex}&pageSize=${this.pageSize}&status=1`,
						method: 'GET',
						success(res) {

							setTimeout(() => {
								_this.loadProgress = 0;
								let list = res.data.appointmentList;
								for (var i = 0; i < list.length; i++) {
									_this.acceptList.push(list[i])
								}
								uni.pageScrollTo({
									scrollTop: 0
								})
								if (list.length === 0) {
									_this.acceptIndex--;
									console.log('没有更多了')
								}
							}, 1000)
						}
					})
				} else if (this.tabIndex === 2) {
					this.refuseIndex++;
					uni.request({
						url: baseUrl +
							`appointment/getAppointmentByStatus?pageIndex=${this.refuseIndex}&pageSize=${this.pageSize}&status=-1`,
						method: 'GET',
						success(res) {
							setTimeout(() => {
								_this.loadProgress = 0;
								let list = res.data.appointmentList;
								for (var i = 0; i < list.length; i++) {
									_this.refuseList.push(list[i])
								}
								uni.pageScrollTo({
									scrollTop: 0
								})
								if (list.length === 0) {
									_this.refuseIndex--;
									console.log('没有更多了')
								}
							}, 1000)
						}
					})
				}
			}
		},
		onReady() {
			this.setTabItemDistance();
			this.setTabSelect(this.tabIndex);
			this.isTeacher = uni.getStorageSync('isTeacher')
			if (this.isTeacher === 'true') {
				this.getInitPage();
			} else {
				this.getStudentInitPage();
			}
		},
		methods: {
			getStudentInitPage() {
				let _this = this
				let skey = uni.getStorageSync('skey')
				uni.request({
					url: baseUrl + `appointment/getStudentAppointmentByStatus?skey=${skey}&status=-1`,
					success(res) {
						_this.studentRefuseList = res.data
					}
				})
				uni.request({
					url: baseUrl + `appointment/getStudentAppointmentByStatus?skey=${skey}&status=0`,
					success(res) {
						_this.studentPendingList = res.data
					}
				})
				uni.request({
					url: baseUrl + `appointment/getStudentAppointmentByStatus?skey=${skey}&status=1`,
					success(res) {
						_this.studentAcceptList = res.data
					}
				})

			},
			toFeedbackItem(id) {
				uni.navigateTo({
					url: '../myinfo/recordList/feedbackItem?id=' + id
				})
			},
			hideModal() {
				this.modalName = null
			},
			toLogin() {
				this.modalName = null;
				uni.navigateTo({
					url: '../login/login'
				})
			},
			getInitPage() {
				//onReady时调用，作用是获取未处理的第一页数据
				let _this = this
				uni.request({
					url: baseUrl +
						`appointment/getAppointmentByStatus?pageIndex=1&pageSize=${999}&status=0`,
					method: 'GET',
					success(res) {
						_this.pendingList = res.data.appointmentList;
						_this.pendingTotalNum = res.data.totalSize;
					}
				})
			},

			acceptOrRefuse(optNum, appointmentId, index) {
				let _this = this
				if (optNum === 1 || optNum === -1) {
					uni.request({
						url: baseUrl + 'appointment/handleAppointment',
						method: 'POST',
						header: {
							'content-type': 'application/x-www-form-urlencoded'
						},
						data: {
							appointmentId: appointmentId,
							status: optNum
						},
						success(res) {
							console.log(res)
							_this.pendingList.splice(index, 1)
						}

					})
				} else {
					console.log('未成功传值')
				}
			},
			getInitSwiperChangePage(tabIndex) {
				let _this = this
				if (tabIndex === 1 && this.acceptIndex === 1 && this.isTeacher === 'true') {
					uni.request({
						url: baseUrl +
							`appointment/getAppointmentByStatus?pageIndex=1&pageSize=${this.pageSize}&status=1`,
						method: 'GET',
						success(res) {
							_this.acceptList = res.data.appointmentList;
							_this.acceptTotalNum = res.data.totalSize;
							console.log(res.data)
						}
					})
				} else if (tabIndex === 2 && this.refuseIndex === 1 && this.isTeacher === 'true') {
					uni.request({
						url: baseUrl +
							`appointment/getAppointmentByStatus?pageIndex=1&pageSize=${this.pageSize}&status=-1`,
						method: 'GET',
						success(res) {
							_this.refuseList = res.data.appointmentList;
							_this.refuseTotalNum = res.data.totalSize;
							console.log(res.data)
						}
					})
				}
			},
			pressScrollViewItem(e) {
				let index = e.target.dataset.current || e.currentTarget.dataset.current;
				this.isPress = true;
				this.setTabSelect(index);
			},
			swiperChange(e) {
				let index = e.target.current || e.detail.current;
				this.tabIndex = index;
				this.setTabSelect(this.tabIndex);
				if (this.isTeacher === 'true') {
					this.getInitSwiperChangePage(this.tabIndex)
				}
			},
			setTabItemDistance() {
				var queryTabSize = uni.createSelectorQuery().in(this);
				for (var i = 0; i < this.tabList.length; i++) {
					queryTabSize.select('#' + this.tabList[i].id).boundingClientRect();
				}
				queryTabSize.exec(rects => {
					rects.forEach((rect) => {
						this.tabListSize[rect.dataset.id] = rect;
					})
					this.setTabItemLinePosition(this.tabListSize[this.tabIndex].left, this.tabListSize[this
							.tabIndex]
						.width);
				});

			},
			setTabItemLinePosition(left, width) {
				this.tabItemLineLeft = left;
				this.tabItemLineWidth = width;
			},
			swiperChangeEnd(e) {
				this.setTabItemLinePosition(this.tabListSize[this.tabIndex].left, this.tabListSize[this.tabIndex].width);
			},
			setTabSelect(index) {
				this.tabIndex = index;
				this.tabScrollInto = this.tabList[index].id;
			},

		},
	}
</script>

<style>
	.container {
		padding: 20rpx 12rpx;
		box-sizing: border-box;
	}


	/* 组件样式 */
	.content {
		display: flex;
		flex-direction: column;
		overflow: hidden;
	}

	.tab {
		width: 100vw;
		height: 80rpx;
		display: flex;
		flex-direction: row;
		/* TODO 要和最后的头部颜色一致 */
		background-color: #36c3bb;
		box-shadow: 0px 6rpx 6rpx rgba(0, 0, 0, 0.2);
	}


	.tab ::-webkit-scrollbar {
		display: none;
		width: 0 !important;
		height: 0 !important;
		-webkit-appearance: none;
		background: transparent;
	}

	.tabLineView {
		position: relative;
		height: 2px;
		background-color: transparent;

	}

	.tabLine {
		position: absolute;
		top: 0;
		bottom: 0;
		width: 0;
		background-color: #DD6161;
	}

	.padding-xl {
		font-size: 30rpx;
	}


	.tabLineActive {
		transition-duration: 0.3s;
		transition-property: left;
	}

	.childPageView {
		display: flex;
		flex-direction: column;
	}

	.tabItem {
		display: flex;
		margin-top: 16rpx;
		padding-left: 12rpx;

	}

	.tabItemTitle {
		color: #d2f1f0;
		font-size: 30rpx;
		display: flex;
		flex-wrap: nowrap;
		align-items: center;
		justify-content: center;
	}

	.tabItemTitleActive {
		color: #F8F8F8;
		font-weight: 500;
		font-size: 40rpx;
	}

	.childPageViewItem {
		flex: 1;
		flex-direction: column;
	}


	.container {}

	.loadTxt {
		align-self: center;
		margin-left: 16rpx;
	}
</style>
