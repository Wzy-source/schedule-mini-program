<template>
	<view class="content">
		<scroll-view class="tab" id="tab" :show-scrollbar="false" :scroll-x="true" :scroll-into-view="tabScrollInto">
			<view class="tabContent" style="display: flex; flex-direction: column;">
				<view style="display: flex;flex-direction: row;">
					<view class="tabItem" v-for="(tab,tabItemIndex) in tabList" :key="tab.id" :id="tab.id" :data-id="tabItemIndex"
						:data-current="tabItemIndex" @click="pressScrollViewItem">
						<text class="tabItemTitle"
							:class="tabIndex==tabItemIndex ? 'tabItemTitleActive' : ''"
							:style="{width:tabList[tabItemIndex].width + 'px'}"
							>{{tab.title}}</text>
					</view>
				</view>
			</view>
		</scroll-view>
		<swiper class="childPageView" :current="tabIndex" :duration="250" @change="swiperChange"
			@animationfinish="swiperChangeEnd" @onAnimationEnd="swiperChangeEnd" :style="{height: swiperHeight+'px'}">
			<swiper-item class="childPageViewItem" v-for="(page, childPageViewItemIndex) in tabList" :key="childPageViewItemIndex">
				<scroll-view  scroll-y="true" :style="{height: swiperHeight+'px'}">
						<!-- 在这里填写内容 -->
				</scroll-view>
			</swiper-item>
		</swiper>
	</view>
</template>

<script>
	import cell1 from './cell1.vue';

	export default {
		components: {
			cell1
		},
		data() {
			return {
				tabList: [{
					id: "tabItem0",
					title: '未处理',
					width:80,
				}, {
					id: "tabItem1",
					title: '已同意',
					width:80,
				}, {
					id: "tabItem2",
					title: '已拒绝',
					width:80,
				}],
				tabIndex: 0,
				tabScrollInto: "",
				tabListSize: {},
				tabItemLineLeft: 0,
				tabItemLineWidth: 0,
				isPress: false,
				swiperHeight: 0,
			}
		},
		onLoad() {

			let system = uni.getSystemInfoSync();
			this.swiperHeight = system.windowHeight - 44;
			console.log(this.swiperHeight);
		},
		onReady() {

			this.setTabItemDistance();
			this.setTabSelect(this.tabIndex);
		},
		methods: {
			pressScrollViewItem(e) {
				let index = e.target.dataset.current || e.currentTarget.dataset.current;
				this.isPress = true;
				this.setTabSelect(index);
			},
			swiperChange(e) {
				let index = e.target.current || e.detail.current;

				this.tabIndex = index;
				this.setTabSelect(this.tabIndex);
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
					console.log(this.tabListSize);
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
				console.log(this.tabScrollInto);

			},

		}
	}
</script>

<style>
	.content {
		/* flex: 1; */
		display: flex;
		flex-direction: column;
		overflow: hidden;
		background-color: #ffffff;
	}

	.tab {
		width: 100vw;
		height: 64px;
		display: flex;
		flex-direction: row;
		background-color:#36c3bb ;
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
		margin-top: 40rpx;
		
	}

	.tabItemTitle {
		color: #d2f1f0;
		font-size: 30rpx;
		height: 84rpx;
		display: flex;
		flex-wrap: nowrap;
		align-items: center;
		justify-content: center;
	}

	.tabItemTitleActive {
		color: #F8F8F8;
		font-weight: 500;
		font-size:40rpx ;
	}

	.childPageViewItem {
		flex: 1;
		flex-direction: column;
	}
</style>
