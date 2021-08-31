<template>
	<view>
		<view class="weekSelectHeader">
			<view class="currentWeek" @tap='showOrHideTab'>
				<view class="weekNumTxt">{{headerWeekTxt}}</view>
			</view>
			<view class="weekTab" v-if="showTab">
				<vgt-tab :list="weekSelectList" @onValueChange='onTabChange' :defaultChoseInd="currentWeekIndex">
				</vgt-tab>
			</view>
		</view>
		<view class="page">
			<view class="dayTime">
				<view class="month">
					<view>{{monthInfo.month}}</view>
					<view>{{monthInfo.monthEng}}</view>
				</view>
				<view class="dayTimeItem" v-for="(item , index) in dayTimeList" :key="index">
					<view>
						<view class="itemNum">{{index+1}}</view>
						<view class="startTime">{{item.start}}</view>
						<view class="endTime">{{item.end}}</view>
					</view>
				</view>
			</view>
			<view class="weekDay">
				<view v-for="(item,index) in weekDayList" class="weekDayItem" :key='index'>
					<view class="weekDayHeight">
						<view>{{item.weekDay}}</view>
						<view>{{item.date}}</view>
					</view>
					<view class="weekDayItemContainer">
						<!-- 在这里放置若干个组件 -->
						<view v-for="(scheduleItem , itemIndex) in totalList[index]" :key="itemIndex">
							<schedule-item :schedule="totalList[index][itemIndex]" @tap="onClick(index,itemIndex)"
								@longpress="onLongPress(index,itemIndex)" :height="totalList[index][itemIndex].height">
							</schedule-item>
						</view>
					</view>
				</view>
			</view>
			<!-- 操作删除和确认的两个按钮 -->
			<view class="optBtn" v-show="showOptBtn">
				<view class="confirmBtn" @tap="confirmItem()">
					<image src="../../static/operation/access.png" mode="scaleToFill"></image>
				</view>
				<view class="deleteBtn" @tap='deleteItem'>
					<image src="../../static/operation/delete.png" mode="scaleToFill"></image>
				</view>
			</view>
		</view>
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
		<!-- 点击标签事件弹出的模态框 -->
		<view class="cu-modal" :class="modalName=='clickEvent'?'show':''">
			<view class="cu-dialog">
				<view class="cu-bar bg-white justify-end">
					<view class="content">当前日程</view>
					<view class="action" @tap="hideModal">
						<text class="cuIcon-close text-red"></text>
					</view>
				</view>
				<view class="padding-xl">
					<!-- 起止时间、人物、地点、内容 -->
					<view class="date scheduleItem">
						<image src="../../static/scheduleIcon/calendar.png" mode="scaleToFill"></image>
						<view class="scheduleTxt">日期：{{itemInfo.date.substring(5)}}</view>
					</view>
					<view class="time scheduleItem">
						<image src="../../static/scheduleIcon/time.png" mode="scaleToFill"></image>
						<view class="scheduleTxt">时间：{{itemInfo.startTime}} - {{itemInfo.endTime}}</view>
					</view>
					<view class="place scheduleItem">
						<image src="../../static/scheduleIcon/place.png" mode="scaleToFill"></image>
						<view class="scheduleTxt">地点：{{itemInfo.place}}</view>
					</view>
					<view class="name scheduleItem">
						<image src="../../static/scheduleIcon/people.png" mode="scaleToFill"></image>
						<view class="scheduleTxt">人物：{{itemInfo.name}}</view>
					</view>
					<view class="content scheduleItem">
						<image src="../../static/scheduleIcon/task.png" mode=""></image>
						<view class="scheduleTxt">内容：{{itemInfo.content}}</view>
					</view>
				</view>
			</view>
		</view>
		<!-- 长按空白事件弹出的模态框 -->
		<view class="cu-modal" :class="modalName=='longpressEvent'?'show':''">
			<view class="cu-dialog">
				<view class="cu-bar bg-white justify-end">
					<view class="content" v-if="isTeacher==='true'">新建日程</view>
					<view class="content" v-else>新建预约</view>
					<view class="action" @tap="hideModal">
						<text class="cuIcon-close text-red"></text>
					</view>
				</view>
				<view class="padding-xl">
					<view class="createScheduleItem">
						<view class="createIcon">
							<image src="../../static/scheduleIcon/calendar.png" mode="scaleToFill"></image>
						</view>
						<!-- 现在只是做老师端，后续长按的人物就需要 -->
						<view class="createTxt">日期：{{itemInfo.date.substring(5)}}</view>
					</view>
					<view class="createScheduleItem">
						<view class="createIcon">
							<image src="../../static/scheduleIcon/time.png" mode="scaleToFill"></image>
						</view>
						<view class="createTxt">
							<view>时间：</view>
							<view>
								<picker mode="selector" :range="startHourRange" :value='startHourIndex'
									@change='startHourChange'>
									<view class="pickerFontStyle">{{startHourTxt}}</view>
								</picker>
							</view>
							<view class="marginBetween"> - </view>
							<view>
								<picker mode="selector" :range="endHourRange" :value='endHourIndex'
									@change='endHourChange'>
									<view class="pickerFontStyle">{{endHourTxt}}</view>
								</picker>
							</view>
						</view>
					</view>
					<view class="createScheduleItem">
						<view class="createIcon">
							<image src="../../static/scheduleIcon/people.png" mode="scaleToFill"></image>
						</view>
						<view class="createTxt" v-if="isTeacher==='false'">人物：{{studentName}}</view>
						<view class="createTxt" v-else>人物：毕菲菲</view>
					</view>
					<view class="createScheduleItem">
						<view class="createIcon">
							<image src="../../static/scheduleIcon/place.png" mode="scaleToFill"></image>
						</view>
						<view class="createTxt">地点：</view>
						<view class="createInput">
							<input type="text" v-model="createPlace">
						</view>
					</view>
					<view class="createScheduleItem">
						<view class="createIcon">
							<image src="../../static/scheduleIcon/task.png" mode="scaleToFill"></image>
						</view>
						<view class="createTxt">内容：</view>
						<view class="createInput">
							<input type="text" v-model="createContent">
						</view>
					</view>
					<view class="createScheduleItem" v-if="isTeacher==='false'">
						<view class="createIcon">
							<image src="../../static/scheduleIcon/record.png" mode="scaleToFill"></image>
						</view>
						<view class="createTxt">备注：</view>
						<view class="createInput">
							<input type="text" v-model="otherContent">
						</view>
					</view>
				</view>
				<view class="cu-bar bg-white justify-end">
					<view class="action">
						<button class="cu-btn line-blue text-blue" @tap="hideModal">取消</button>
						<button class="cu-btn bg-gradual-blue margin-left" @tap="saveCurrentInfo">确定</button>
					</view>
				</view>
			</view>
		</view>
		<!-- 点击删除按钮弹出的底部窗口 -->
		<view class="cu-modal bottom-modal" :class="modalName=='confirmDelete'?'show':''">
			<view class="cu-dialog">
				<view class="cu-bar bg-white padding-left-right">
					<button class="cu-btn bg-gradual-blue left" @tap="hideModal">取消</button>
					<button class="cu-btn bg-yellow right" @tap='confirmDeleteItem'>确定</button>
				</view>
				<view class="padding-xl confirmDeleteTxt">
					确定删除该日程吗？
				</view>

			</view>
		</view>
		<!-- 学生预约后弹出的窗口 -->
		<view class="cu-modal" :class="modalName=='appointmentAlertMsgModal'?'show':''">
			<view class="cu-dialog">
				<view class="cu-bar bg-white justify-end">
					<view class="content">温馨提示</view>
					<view class="action" @tap="hideModal">
						<text class="cuIcon-close text-red"></text>
					</view>
				</view>
				<view class="padding-xl" style="font-size: 30rpx;">
					{{appointmentAlertMsg}}
				</view>
			</view>
		</view>
	</view>
</template>

<script>
	import baseUrl from '../../url.js'
	import scheduleItem from '../../components/scheduleItem.vue'
	import moment from 'moment'
	import Vue from 'vue'
	import vgtTab from '../../components/vgt-tab/vgt-tab.vue'
	export default {
		components: {
			scheduleItem,
			vgtTab
		},
		data() {
			return {
				dayTimeList: [{
						start: '09:00',
						end: '10:00'
					},
					{
						start: '10:00',
						end: '11:00'
					},
					{
						start: '11:00',
						end: '12:00'
					},
					{
						start: '12:00',
						end: '13:00'
					},
					{
						start: '13:00',
						end: '14:00'
					},
					{
						start: '14:00',
						end: '15:00'
					},
					{
						start: '15:00',
						end: '16:00'
					},
					{
						start: '16:00',
						end: '17:00'
					},
					{
						start: '17:00',
						end: '18:00'
					},
					{
						start: '18:00',
						end: '19:00'
					},
					{
						start: '19:00',
						end: '20:00'
					},
				],
				isTeacher: uni.getStorageSync('isTeacher'),
				weekSchedule: {},
				weekAppointment: {},
				totalList: [],
				weekDate: {},
				//模态框相关data
				modalName: '',
				createPlace: '',
				createContent: '',
				itemInfo: {
					date: '',
					place: '',
					name: '',
					content: '',
					rowIndex: 0,
					colIndex: 0
				}, //点击or长按要显示的内容
				//长按弹出的按钮
				showOptBtn: false,
				animationData: {}, //控制微动画
				startHourRange: [],
				startHourTxt: '请选择',
				endHourRange: [],
				endHourTxt: '请选择',
				startHourIndex: 0,
				endHourIndex: 0,
				//学生相关
				studentName: uni.getStorageSync('name'),
				otherContent: '',
				//学生预约成功失败提示文字
				appointmentAlertMsg: '',
				//当前月份
				monthInfo: {},
				//周数选择相关
				weekSelectList: ['第一周', '第二周', '第三周', '第四周', '第五周', '第六周', '第七周', '第八周', '第九周', '第十周', '第十一周', '第十二周',
					'第十三周', '第十四周', '第十五周', '第十六周', '第十七周', '第十八周', '第十九周', '第二十周'
				],
				showTab: false,
				currentWeekIndex: 0,
				selectWeekIndex: 0,
			}
		},

		methods: {
			doInit() {
				// 初始化操作：若没登录，则进行登陆，若已经登录，则获取日程安排
				let avatarUrl = uni.getStorageSync('avatarUrl');
				let nickName = uni.getStorageSync('nickName');
				let name = uni.getStorageSync('name')
				if (nickName.length === 0 || avatarUrl.length === 0 || name.length === 0 || this.currentWeekIndex
					.length === 0) { //没有本地存储为新用户，请先登录
					this.modalName = 'DialogModal1'
				} else {
					this.onLogin()
					this.getAppointment()
				}
			},
			hideModal() {
				this.modalName = null
				this.startHourTxt = '请选择'
				this.endHourTxt = '请选择'
			},
			toLogin() {
				this.modalName = null;
				uni.navigateTo({
					url: '../login/login'
				})
			},
			onClick(i, j) {
				this.showOptBtn = false
				this.itemInfo = this.totalList[i][j];
				if (this.itemInfo.type !== 'none') {
					//点击事件，显示模态框
					this.modalName = 'clickEvent'
				}
			},
			onLongPress(i, j) {
				this.itemInfo = this.totalList[i][j];
				if (this.itemInfo.type === 'none') {
					this.startHourRange = []
					this.endHourRange = []
					this.otherContent = ''
					this.createPlace = ''
					this.createContent = ''
					this.startHourTxt = '请选择'
					this.endHourTxt = '请选择'
					this.modalName = 'longpressEvent'
					let startHour = parseInt(this.itemInfo.startTime.split(':')[0])
					for (var i = startHour; i < 20; i++) {
						let hourStr = (i === 9 ? '09:00' : i + ':00')
						this.startHourRange.push(hourStr)
					}
				} else if (this.isTeacher === 'true' && (this.itemInfo.type === 'teacher')) {
					this.showOptBtn = true
				} else if (this.isTeacher === 'false' && (this.itemInfo.type === 'student')) {
					this.showOptBtn = true
				}
			},
			getWeekSchedule() {
				let _this = this;
				uni.request({
					url: baseUrl +
						`schedule/getScheduleByDate?startDate=${this.weekDate[1]}&endDate=${this.weekDate[7]}`,
					success(res) {
						_this.weekSchedule = res.data
						_this.setTotalList()
					}
				})
			},
			getAppointment() {
				let _this = this;
				uni.request({
					url: baseUrl +
						`appointment/getAppointmentByDate?startDate=${this.weekDate[1]}&endDate=${this.weekDate[7]}`,
					success(res) {
						_this.weekAppointment = res.data
						_this.getWeekSchedule()
					}
				})
			},
			setTotalList() {
				this.totalList = []
				let tempList = []
				let list = []
				for (var i = 1; i <= 7; i++) {
					let dayList = []
					for (var j = 0; j < this.weekSchedule[i].length; j++) {
						this.weekSchedule[i][j].type = 'teacher'
						this.weekSchedule[i][j].name = '毕菲菲'
						dayList.push(this.weekSchedule[i][j])
					}
					for (var k = 0; k < this.weekAppointment[i].length; k++) {
						this.weekAppointment[i][k].type = 'student'
						dayList.push(this.weekAppointment[i][k])
					}
					list.push(dayList)
				}
				for (var i = 0; i < 7; i++) {
					let dayList = []
					for (var j = 0; j < 11; j++) {
						dayList.push({
							date: this.weekDate[i + 1],
							startTime: j === 0 ? '09:00' : (j + 9) + ':00',
							endTime: (j + 10) + ':00',
							type: 'none'
						})
					}
					tempList.push(dayList)
				}
				for (var i = 0; i < 7; i++) {
					let dayList = list[i];
					for (var j = 0; j < dayList.length; j++) {
						let item = dayList[j];
						let startIndex = parseInt(item.startTime) - 9;
						let endIndex = parseInt(item.endTime) - 9;
						for (var k = startIndex; k < endIndex; k++) {
							delete tempList[i][k]
						}
						tempList[i].splice(startIndex, 1, item)
					}
				}

				for (var i = 0; i < tempList.length; i++) {
					let dayList = []
					for (var j = 0; j < tempList[i].length; j++) {
						if (tempList[i][j] !== undefined) {
							dayList.push(tempList[i][j])
						}
					}
					this.totalList.push(dayList)
				}
				this.setHeight()
			},

			onTabChange(e) {
				this.selectWeekIndex = e.currentInd;
				this.setWeekDate()
				this.doInit()

			},
			showOrHideTab() {
				this.showTab = !this.showTab
			},
			setCurrentWeek(){
				this.currentWeekIndex = uni.getStorageSync('currentWeek').length===0? 0 : parseInt(uni.getStorageSync('currentWeek'))
				this.selectWeekIndex = this.currentWeekIndex
			},

			setWeekDate() {
				let gapWeekNum = this.selectWeekIndex - this.currentWeekIndex //选中的周数和当前周数之间的差值
				var weekOfday = moment().format('E'); //计算今天是这周第几天
				this.weekDate = {
					1: moment().subtract(weekOfday - 1 - 7*gapWeekNum, 'days').format('YYYY-MM-DD'),
					2: moment().subtract(weekOfday - 2 - 7*gapWeekNum, 'days').format('YYYY-MM-DD'),
					3: moment().subtract(weekOfday - 3 - 7*gapWeekNum, 'days').format('YYYY-MM-DD'),
					4: moment().subtract(weekOfday - 4 - 7*gapWeekNum, 'days').format('YYYY-MM-DD'),
					5: moment().subtract(weekOfday - 5 - 7*gapWeekNum, 'days').format('YYYY-MM-DD'),
					6: moment().subtract(weekOfday - 6 - 7*gapWeekNum, 'days').format('YYYY-MM-DD'),
					7: moment().subtract(weekOfday - 7 - 7*gapWeekNum, 'days').format('YYYY-MM-DD')
				}
				//月份是每周的第一天
				let currentMonthStr = moment().subtract(weekOfday - 1 - 7*gapWeekNum, 'days').format('MM')
				let currentMonth = parseInt(currentMonthStr)
				let monthEnglishStr = ''
				if (currentMonth === 1) {
					monthEnglishStr = 'Jan.'
				} else if (currentMonth === 2) {
					monthEnglishStr = 'Feb.'
				} else if (currentMonth === 3) {
					monthEnglishStr = 'Mar.'
				} else if (currentMonth === 4) {
					monthEnglishStr = 'Apr.'
				} else if (currentMonth === 5) {
					monthEnglishStr = 'May'
				} else if (currentMonth === 6) {
					monthEnglishStr = 'Jun.'
				} else if (currentMonth === 7) {
					monthEnglishStr = 'Jul.'
				} else if (currentMonth === 8) {
					monthEnglishStr = 'Aug.'
				} else if (currentMonth === 9) {
					monthEnglishStr = 'Sep.'
				} else if (currentMonth === 10) {
					monthEnglishStr = 'Oct.'
				} else if (currentMonth === 11) {
					monthEnglishStr = 'Nov.'
				} else {
					monthEnglishStr = 'Dec.'
				}
				this.monthInfo.month = currentMonth + '月'
				this.monthInfo.monthEng = monthEnglishStr
			},
			saveCurrentInfo() {
				this.modalName = ''
				let _this = this
				if (this.isTeacher === 'true') {
					uni.request({
						url: baseUrl + 'schedule/create',
						method: 'POST',
						header: {
							'content-type': 'application/x-www-form-urlencoded'
						},
						data: {
							date: this.itemInfo.date,
							startTime: this.startHourTxt,
							endTime: this.endHourTxt,
							place: this.createPlace,
							content: this.createContent
						},
						success(res) {
							_this.createPlace = ''
							_this.createContent = ''
							_this.startHourTxt = '请选择'
							_this.endHourTxt = '请选择'
							_this.doInit()
						}
					})
				} else {
					uni.request({
						url: baseUrl + 'appointment/create',
						method: 'POST',
						header: {
							'content-type': 'application/x-www-form-urlencoded'
						},
						data: {
							date: this.itemInfo.date,
							startTime: this.startHourTxt,
							endTime: this.endHourTxt,
							place: this.createPlace,
							content: this.createContent,
							skey: uni.getStorageSync('skey'),
							other: this.otherContent
						},
						success(res) {
							_this.createPlace = ''
							_this.createContent = ''
							_this.startHourTxt = '请选择'
							_this.endHourTxt = '请选择'
							_this.modalName = 'appointmentAlertMsgModal'
							if (res.data.code === -1) {
								_this.appointmentAlertMsg = res.data.msg;
							} else {
								let month = parseInt(_this.itemInfo.date.substring(5).split('-')[0])
								let day = parseInt(_this.itemInfo.date.substring(5).split('-')[1])
								_this.appointmentAlertMsg =
									`成功预约 ${month}月${day}日 ${_this.itemInfo.startTime}-${_this.itemInfo.endTime}时段，待老师同意后，您的预约将显示在此处`
								_this.doInit()
							}
						}
					})
				}
			},
			deleteItem() {
				this.modalName = 'confirmDelete'

			},

			confirmItem() { //确认日程
				this.showOptBtn = false

			},

			confirmDeleteItem() { //删除日程scheduleId
				let _this = this
				if (this.isTeacher === 'true' && (this.itemInfo.type === 'teacher')) {
					uni.request({
						url: baseUrl + 'schedule/delete',
						data: {
							scheduleId: this.itemInfo.scheduleId
						},
						method: 'POST',
						header: {
							'content-type': 'application/x-www-form-urlencoded'
						},
						success(res) {
							_this.doInit()
							_this.hideModal()
							_this.showOptBtn = false
						}
					})
				} else if (this.isTeacher === 'false' && (this.itemInfo.type === 'student')) {
					uni.request({
						url: baseUrl + 'appointment/cancel',
						data: {
							skey: uni.getStorageSync('skey'),
							appointmentId: this.itemInfo.appointmentId
						},
						method: 'POST',
						header: {
							'content-type': 'application/x-www-form-urlencoded'
						},
						success(res) {
							_this.hideModal()
							_this.showOptBtn = false
							console.log(res)
							if (res.data.code === -1) {
								_this.appointmentAlertMsg = res.data.msg
								_this.modalName = 'appointmentAlertMsgModal'
							} else {
								_this.doInit()
							}
						}
					})
				}
			},
			setHeight() {
				for (var i = 0; i < this.totalList.length; i++) {
					for (var j = 0; j < this.totalList[i].length; j++) {
						let startTime = this.totalList[i][j].startTime
						let endTime = this.totalList[i][j].endTime
						let height = 160 * (parseInt((endTime.split(":")[0])) - parseInt((startTime.split(":")[0])))
						this.totalList[i][j].height = height
						this.totalList[i][j].rowIndex = i;
						this.totalList[i][j].colIndex = j
					}
				}
			},
			startHourChange(event) {
				let index = event.detail.value
				this.startHourTxt = this.startHourRange[index]
				let startHour = parseInt(this.startHourTxt.split(':')[0])
				for (var i = startHour + 1; i <= 20; i++) {
					let endStr = i + ':00'
					this.endHourRange.push(endStr)
				}
			},
			endHourChange(event) {
				let index = event.detail.value
				this.endHourTxt = this.endHourRange[index]
			},
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
									uni.request({ //判断是不是老师
										url: baseUrl + 'user/getInformation?skey=' + uni
											.getStorageSync('skey'),
										success(res) {
											if (res.data.code === 200) {
												uni.setStorageSync('isTeacher', res.data.data
													.isTeacher);
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
		},
		onLoad() {
			this.setCurrentWeek()
			this.setWeekDate()
			this.doInit();
		},
		onTabItemTap(e) {
			this.setCurrentWeek()
			this.setWeekDate()
			this.doInit()
		},
		computed: {
			weekDayList: function() {
				let list = [{
						weekDay: '周一',
						date: ''
					},
					{
						weekDay: '周二',
						date: ''
					},
					{
						weekDay: '周三',
						date: ''
					},
					{
						weekDay: '周四',
						date: ''
					},
					{
						weekDay: '周五',
						date: ''
					},
					{
						weekDay: '周六',
						date: ''
					},
					{
						weekDay: '周日',
						date: ''
					}
				]
				if (this.weekDate[1]) {
					for (var i = 0; i < 7; i++) {
						list[i].date = this.weekDate[i + 1].substring(5)
					}
				}
				return list
			},
			headerWeekTxt: function() {
				if (this.currentWeekIndex === this.selectWeekIndex) {
					return this.weekSelectList[this.currentWeekIndex] + '（本周）'
				} else {
					return this.weekSelectList[this.selectWeekIndex] + '（非本周）'
				}
			}

		}
	}
</script>


<style scoped>
	.page {
		width: 100%;
		display: flex;
		text-align: center;
		padding: 12rpx;
	}

	.dayTime {
		width: 10%;
		display: grid;
		/* 单元格的高度 */
		grid-template-rows: 90rpx repeat(11, 160rpx);
	}

	.month {
		height: 90rpx;
	}

	.weekDayHeight {
		height: 90rpx;
	}

	.dayTimeItem {
		border: none;
		border-bottom: 1upx solid rgba(0, 0, 0, 0.1);
		display: flex;
		font-size: 26rpx;
		justify-content: center;
		font-family: 'Trebuchet MS', 'Lucida Sans Unicode', 'Lucida Grande', 'Lucida Sans', Arial, sans-serif;
	}

	.dayTimeItem>view {
		align-self: center;
		display: flex;
		flex-direction: column;
		justify-content: center;
	}

	.dayTimeItem>view>view {
		align-self: center;
	}



	.weekDayItemContainer {
		/* 若修改单元格的高度，这里也要同步修改 */
		height: 1760rpx;
		display: flex;
		flex-direction: column;
		width: 100%;
	}

	.weekDay {
		display: grid;
		width: 100%;
		grid-template-columns: repeat(7, 14.2857142857%);
	}


	.padding-xl {
		font-size: 30rpx;
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
		text-align: left;
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
		display: flex;
	}

	.createTxt>view {
		align-self: center;
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

	.confirmBtn {
		height: 70rpx;
		width: 70rpx;
		position: fixed;
		bottom: 140rpx;
		right: 60rpx;
	}

	.confirmBtn>image {
		width: 100%;
		height: 100%;
	}

	.deleteBtn {
		height: 70rpx;
		width: 70rpx;
		position: fixed;
		bottom: 60rpx;
		right: 60rpx;
	}

	.deleteBtn>image {
		width: 100%;
		height: 100%;
	}

	.confirmDeleteTxt {
		color: #ED1C24;
		font-family: 'Trebuchet MS', 'Lucida Sans Unicode', 'Lucida Grande', 'Lucida Sans', Arial, sans-serif;
	}

	.left {
		margin-left: 16rpx;
	}

	.right {
		margin-right: 16rpx;
	}

	.pickerFontStyle {
		font-family: 'Trebuchet MS', 'Lucida Sans Unicode', 'Lucida Grande', 'Lucida Sans', Arial, sans-serif;
		color: #707070;
		font-size: 30rpx;
	}

	.marginBetween {
		margin-left: 8rpx;
		margin-right: 8rpx;
	}

	.fontFamily {
		font-family: 'Trebuchet MS', 'Lucida Sans Unicode', 'Lucida Grande', 'Lucida Sans', Arial, sans-serif;
	}


	.currentWeek {
		display: flex;
		justify-content: center;
		height: 78rpx;
		background-color: #36c3bb;
		box-shadow: 0px 6rpx 6rpx rgba(0, 0, 0, 0.2);
	}

	.currentWeek>view {
		align-self: center;
		font-size: 34rpx;
		font-weight: 500;
	}


	.weekTab>>>.scroll-view_hold {
		padding: 18rpx 0rpx 18rpx 0rpx;
	}
	
	.padding-left-right{
		padding: 0rpx 32rpx;
	}
	

</style>
