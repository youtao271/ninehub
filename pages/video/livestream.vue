<template>
	<view class="wrap">
		<u-waterfall v-model="currentList" ref="uWaterfall">
			<template v-slot:left="{ leftList }">
				<view class="demo-warter" v-for="(item, index) in leftList" :key="index">
					<navigator :url="'/pages/video/plate?id='+item.address.split('.')[0]">
						<!-- 微信小程序需要hx2.8.11版本才支持在template中引入其他组件，比如下方的u-lazy-load组件 -->
						<u-lazy-load threshold="-450" border-radius="10" :image="item.xinimg" :index="index"></u-lazy-load>
						<view class="demo-title">{{ item.title }}</view>
					</navigator>
				</view>
			</template>
			<template v-slot:right="{ rightList }">
				<view class="demo-warter" v-for="(item, index) in rightList" :key="index">
					<navigator :url="'/pages/video/plate?id='+item.address.split('.')[0]">
						<u-lazy-load threshold="-450" border-radius="10" :image="item.xinimg" :index="index"></u-lazy-load>
						<view class="demo-title">{{ item.title }}</view>
					</navigator>
				</view>
			</template>
		</u-waterfall>
		<u-loadmore bg-color="rgb(240, 240, 240)" :status="loadStatus" @loadmore="parseData"></u-loadmore>
	</view>
</template>

<script>
export default {
	data() {
		return {
			page: 0,
			pageSize: 10,
			loadStatus: 'loadmore',
			plateList: [],
			currentList: []
		};
	},
	async onLoad() {
		await this.loadData();
		this.parseData(this.page);
	},
	onReachBottom() {
		this.loadStatus = 'loading';
		// 模拟数据加载
		this.parseData();
	},
	methods: {
		loadData: async function() {
      const {tcbCloud} = getApp().globalData;
			const { result } = await tcbCloud.callFunction({
				name: 'livestream'
			});
			this.plateList = result;
		},
		parseData: function() {
			const end = (this.page + 1) * this.pageSize;
			this.currentList = this.plateList.slice(0, end);
			this.page++;
			console.log(this.currentList);
		},
		remove(id) {
			this.$refs.uWaterfall.remove(id);
		},
		clear() {
			this.$refs.uWaterfall.clear();
		}
	}
};
</script>

<style>
/* page不能写带scope的style标签中，否则无效 */
page {
	background-color: rgb(240, 240, 240);
}
</style>

<style lang="scss" scoped>
.demo-warter {
	border-radius: 8px;
	margin: 5px;
	background-color: #ffffff;
	padding: 8px;
	position: relative;
}

.u-close {
	position: absolute;
	top: 32rpx;
	right: 32rpx;
}

.demo-img-wrap {
}

.demo-image {
	width: 100%;
	border-radius: 4px;
}

.demo-title {
	font-size: 30rpx;
	margin-top: 5px;
	color: $u-main-color;
	word-break: break-all;
}

.demo-tag {
	display: flex;
	margin-top: 5px;
}

.demo-tag-owner {
	background-color: $u-type-error;
	color: #ffffff;
	display: flex;
	align-items: center;
	padding: 4rpx 14rpx;
	border-radius: 50rpx;
	font-size: 20rpx;
	line-height: 1;
}

.demo-tag-text {
	border: 1px solid $u-type-primary;
	color: $u-type-primary;
	margin-left: 10px;
	border-radius: 50rpx;
	line-height: 1;
	padding: 4rpx 14rpx;
	display: flex;
	align-items: center;
	border-radius: 50rpx;
	font-size: 20rpx;
}

.demo-price {
	font-size: 30rpx;
	color: $u-type-error;
	margin-top: 5px;
}

.demo-shop {
	font-size: 22rpx;
	color: $u-tips-color;
	margin-top: 5px;
}
</style>
