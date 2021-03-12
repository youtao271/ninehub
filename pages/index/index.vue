<template>
	<view>
		<u-card v-for="(item, index) in matches" :key="index" :show-head="false">
			<view class="match-body" slot="body">
				<!-- <navigator :url="'/pages/play/play?id='+item.id">				 -->
					<view class="match-title">{{`${getTime(item.startTime)} NBA${item.matchType==2?'常规赛':'季后赛'}`}}</view>
					<u-row>
						<u-col span="3">
							<view class="match-team">
								<u-image mode="aspectFit" width="100%" height="120" :src="item.awayImg"></u-image>
								<text class="match-team-name">{{item.away}}</text>
							</view>
						</u-col>
						<u-col span="6">
							<view class="match-team-stat">
								<view v-if="item.matchPeriod==3">
									<view class="match-team-status">
										<u-button disabled type="warning" shape="circle">比赛延期</u-button>
									</view>
								</view>
								<view v-else>
									<view class="match-team-goal">{{`${item.awayGoal} : ${item.homeGoal}`}}</view>
									<view v-if="item.matchPeriod === '3'" class="match-team-status">
										<u-button disabled shape="circle">比赛结束</u-button>
									</view>
									<view v-else class="match-team-status">
										<u-button type="primary" shape="circle" @click="live(item.matchPeriod, item.id, item.cid, item._id)">视频直播</u-button>
									</view>
								</view>
							</view>
						</u-col>
						<u-col span="3">
							<view class="match-team">
								<u-image mode="aspectFit" width="100%" height="120" :src="item.homeImg"></u-image>
								<text class="match-team-name">{{item.home}}</text>
							</view>
						</u-col>
					</u-row>				
				<!-- </navigator> -->
			</view>
		</u-card>
	</view>
</template>

<script>
	let timer = null;
	export default {
		data() {
			return {
				title: 'Hello',
				matches: [],
				date: ''
			}
		},
		async onLoad() {
			this.setDate();
			await this.getData();
		},
		onShow() {
			timer = setInterval(this.getData, 30000);
			console.log(this.date);
			// this.setDate(-1);
		},
		onHide() {
			clearInterval(timer);
		},
		async onPullDownRefresh() {
			await this.getData();
			uni.stopPullDownRefresh();
		},
		computed: {
			finished: function() {
				if(!this.matches.length)	return false;
				return this.matches.every(item => item.matchPeriod==='3' || item.matchPeriod==='2' )
			}
		},
		methods: {
			setDate: function(flag=0) {
				const date = flag ? new Date(this.date) : new Date();
				date.setDate(date.getDate() + flag);

				const year = date.getFullYear();
				const month = date.getMonth() + 1;
				const day = date.getDate();
				this.date = `${year}-${month<10?'0':''}${month}-${day<10?'0':''}${day}`;
			},
			getData: async function() {
				console.log(this.finished);
				this.finished && clearInterval(timer);
				const db = uniCloud.database();
				const { result } = await db.collection('nba_match_list').where('date == "'+this.date+'" && cid == "100000"')
				.orderBy('_id').get();
				console.log(result.data);
				this.matches = result.data;
			},
			getTime: date => {
				return date.split(' ')[1].substr(0, 5);
			},
			live: (status, id, cid, _id) => {
				console.log(status, cid, _id);
				// if(status === '2' || status === '3')	return;
				uni.navigateTo({
					url: `/pages/nba/live?id=${id}&mid=${cid}:${_id}`
				})
			}
		}
	}
</script>

<style>
	.match-body {font-family: "Helvetica Neue",Helvetica,STHeiTi Light,sans-serif;}
	.match-title {
		color: #9E9E9E;
		font-size: 28rpx;
		text-align: center;
		width: 100%;
	}
	.match-team {text-align: center;color: #373A41;font-size: 32rpx;}
	
	.match-team-name {margin-top: 12rpx;}
	
	.match-team-stat {text-align: center; padding: 0 20rpx;}
	
	.match-team-goal {font-size: 40rpx; padding: 16rpx 0;}
</style>
