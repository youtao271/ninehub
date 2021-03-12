<template>
	<view>
		<view class="status_bar" />
		<u-tabs :list="tabs" :current="tabIndex" @change="changeTab" />
		<u-search 
			placeholder="请输入关键词" 
			v-model="keyword" 
			margin="12rpx 20rpx 4rpx"
			bg-color="#ffffff"
			:action-style="{color: '#409eff'}"
			@custom="search" 
			@search="search"
		/>
		<u-loading :show="loaded" size="40" />
		<u-waterfall v-model="list" ref="uWaterfall" add-time="10">
			<template v-slot:left="{leftList}">
				<view class="item" v-for="(item, index) in leftList" :key="index" @click="play(item.vod_id)">
					<u-tag v-if="item.vod_remarks" class="tag-banner" :text="item.vod_remarks" type="success" mode="dark" />
					<u-lazy-load border-radius="10" :image="pic" :index="index"></u-lazy-load>
					<!-- <u-lazy-load border-radius="10" :image="item.vod_pic" :index="index"></u-lazy-load> -->
					<view class="item-title">
						{{item.vod_name}}
					</view>
				</view>
			</template>
			<template v-slot:right="{rightList}">
				<view class="item" v-for="(item, index) in rightList" :key="index" @click="play(item.vod_id)">
					<u-tag v-if="item.vod_remarks" class="tag-banner" :text="item.vod_remarks" type="success" mode="dark" />
					<u-lazy-load border-radius="10" :image="pic" :index="index"></u-lazy-load>
					<!-- <u-lazy-load border-radius="10" :image="item.vod_pic" :index="index"></u-lazy-load> -->
					<view class="item-title">
						{{item.vod_name}}
					</view>
				</view>
			</template>
		</u-waterfall>
		<u-loadmore v-if="!loaded" :status="status" :icon-type="iconType" :load-text="loadText" />
	</view>
</template>

<script>
	let timer = null;
	export default {
		data() {
			return {
				pic: 'https://cdnvideo.sinovision.net/attachments/video/20210310/2012/1615425160-8176.jpg',
				loaded: true,
				status: 'loadmore',
				iconType: 'circle',
				loadText: {
					loadmore: '上拉或点击加载更多',
					loading: '努力加载中',
					nomore: '我是有底线的'
				},
				keyword: '',
				tid: '',
				page: 1,
				list: [],
				tabs: [
					{name: '最新'},
					{name: '电影'},
					{name: '电视剧'},
					{name: '动漫'},
					{name: '综艺'}
				],
				tabIndex: 0,
				aliCloud: null
			}
		},
		async onLoad() {
			await this.getData();
			this.loaded = false;
		},
		async onShow() {
			console.log('show');
		},
		onHide() {

		},
		async onPullDownRefresh() {
			this.page = 1;
			this.$refs.uWaterfall.clear();
			await this.getData();
			uni.stopPullDownRefresh();
		},
		async onReachBottom() {
			if(this.status !== 'nomore'){
				this.status = 'loading';
				await this.loading();
			}
		},
		methods: {
			getData: async function() {
				const {aliCloud, tcbCloud} = getApp().globalData;
				const { result } = await tcbCloud.callFunction({
					name: 'video',
					data: {
						// type: 'list',
						wd: this.keyword,
						pg: this.page,
						t: this.tid,
						h: 24,
						// ids: '69346,48602'
					}
				});
				// this.$refs.uWaterfall.clear();
				this.status = result.length ? 'loadmore' : 'nomore';
				this.list = this.page===1 ? [...result] : [...this.list, ...result];
				console.log(this.page, this.list.length);
			},
			loading: async function() {
				this.page++;
				await this.getData();
			},
			play: function(id) {
				getApp().globalData.videoInfo = this.list.find(item => item.vod_id===id);
				uni.navigateTo({
					url: `/pages/video/living`
				})
			},
			search: async function(val) {
				this.keyword = val;
				this.page = 1;
				this.$refs.uWaterfall.clear();
				await this.getData();
			},
			changeTab: function(index) {
				this.tabIndex = index;
			}
		}
	}
</script>

<style>
	page {
		background-color: rgb(240, 240, 240);
	}
	.status_bar {
		height: var(--status-bar-height);
		width: 100%;
	}
	
</style>
<style lang="scss" scoped>
	.item {
		border-radius: 16rpx;
		margin: 10rpx;
		background-color: #ffffff;
		padding: 16rpx;
		position: relative;
	}
	.item-title {
		width: 100%;
		font-size: 30rpx;
		margin-top: 10rpx;
		color: $u-main-color;
	}
	.tag-banner {
		position: absolute;
		top: 24rpx;
		right: 24rpx;
		padding: 10rpx 16rpx;
		z-index: 99;
	}
	.tag-update {
		
	}
</style>
