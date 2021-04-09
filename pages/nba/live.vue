<template>
	<view>
		<u-sticky>
			<!-- #ifdef H5 -->
			<view class="player_container"></view>
			<!-- #endif -->
			
			<!-- #ifndef H5 -->
			<video class="player" id="player" :src="videoUrl" @error="videoErrorCallback" autoplay :showProgress="false" />
			<!-- #endif -->
		</u-sticky>
		
		<!-- 直播线路列表 -->
		<view class="match-info">
			<u-row>
				<u-col class="url-item" span="3" v-for="(item, i) in videoList" :key="i">
					<u-button @click="changeUrl(i)" size="mini" :type="index===i?'success':''">{{item.title}}</u-button>
				</u-col>
			</u-row>
		</view>
		
		<!-- 比赛双方信息 -->
		<view class="match-info">
			<u-row>
				<u-col span="3" text-align="center">
					<u-image mode="aspectFit" width="100%" height="120" :src="teamInfo.leftBadge" />
					<text class="match-team-name">{{teamInfo.leftName}}</text>
				</u-col>
				<u-col span="6">
					<u-row>
						<u-col text-align="center" span="12">{{`NBA${matchType==2?'常规赛':'季后赛'}`}}</u-col>
					</u-row>
					<u-row>
						<u-col text-align="center" span="4">
							<text class="color-red">{{teamInfo.leftGoal}}</text>
						</u-col>
						<u-col text-align="center" span="4">
							<u-tag type="success" text="live"/>
						</u-col>
						<u-col text-align="center" span="4">
							<text class="color-red">{{teamInfo.rightGoal}}</text>
						</u-col>
					</u-row>
					<u-row>
						<u-col text-align="center" span="12">{{teamInfo.quarterDesc}}</u-col>
					</u-row>
				</u-col>
				<u-col span="3" text-align="center">
					<u-image mode="aspectFit" width="100%" height="120" :src="teamInfo.rightBadge" />
					<text class="match-team-name">{{teamInfo.rightName}}</text>
				</u-col>
			</u-row>
		</view>

		<!-- 各节得分详情 -->
		<view class="match-info">
			<u-row class="quarter-item table-title">
				<u-col span="4">
					<text class="quarter-item-title">球队</text>
				</u-col>
				<u-col span="8">
					<u-row>
						<u-col span="2" v-for="(item, i) in (periodGoals.head||[])" :key="i">
							<view>{{item}}</view>
						</u-col>
					</u-row>
				</u-col>
			</u-row>
			<u-row class="quarter-item" v-for="(row, index) in (periodGoals.rows||[])" :key="index">
				<u-col span="4">
					<u-row>
						<u-col span="4">
							<u-image mode="aspectFit" width="100%" height="52" :src="index ? teamInfo.rightBadge : teamInfo.leftBadge"></u-image>
						</u-col>
						<u-col span="8">
							<text class="match-team-name">{{index ? teamInfo.rightName : teamInfo.leftName}}</text>
						</u-col>
					</u-row>
				</u-col>
				<u-col span="8">
					<u-row>
						<u-col span="2" v-for="(item, col) in row" v-if="col > 0" :key="col">
							<view>{{item}}</view>
						</u-col>
					</u-row>
				</u-col>
			</u-row>
		</view>

		<!-- 技术统计 -->
		<view class="match-info" v-for="(team, key) in playerStats" :key="key">
      <view class="stat-fixed">
        <view class="stat-head">
          <view class="stat-row">
            <text class="stat-col player-head-name">{{ getStatHeadFixed(key)[0] }}</text>
            <text class="stat-col player-time">{{ getStatHeadFixed(key)[1] }}</text>
            <text class="stat-col player-score">{{ getStatHeadFixed(key)[2] }}</text>
          </view>
        </view>
        <view class="stat-body">
          <view class="stat-row" v-for="(item, i) in getStatBody(key)" :key="i">
            <text class="stat-col player-num u-line-1">{{ getStatBodyFixed(item)[0] }}</text>
            <text class="stat-col player-body-name u-line-1">{{ getStatBodyFixed(item)[1] }}</text>
            <text class="stat-col player-time u-line-1">{{ getStatBodyFixed(item)[2] }}</text>
            <text class="stat-col player-score u-line-1">{{ getStatBodyFixed(item)[3] }}</text>
          </view>
        </view>
      </view>

      <scroll-view class="stat-scroll" scroll-x="true">
        <view class="stat-head">
          <view class="stat-row">
            <text class="stat-col player-stat-item" v-for="(item, i) in getStatHeadScroll(key)" :key="i">{{ item }}</text>
          </view>
        </view>
        <view class="stat-head">
          <view class="stat-row" v-for="(row, r) in getStatBody(key)" :key="r">
            <text class="stat-col player-stat-item" v-for="(item, i) in getStatBodyScroll(row)" :key="i">{{ item }}</text>
          </view>
        </view>
      </scroll-view>

      <view class="clearfix"></view>
		</view>
	</view>
</template>

<script>
	let timer = null;
	export default {
		data() {
			return {
				index: 0,
				videoList: [],
				teamInfo: [],
				periodGoals: [],
				playerStats: [],
				leftPlayers: [],
				rightPlayers: [],
				matchType: 0,
				video: 'https://jpmedia.sinovision.net/video/2021/2/16/TDA_FEB_1000k.mp4',
				// video3: 'https://cdnmedia1.sinovision.net/247A45/live/livestream.m3u8',
        video3: 'http://ivi.bupt.edu.cn/hls/cctv3hd.m3u8'
      }
		},
		async onLoad({id = 206937, mid }) {
			this.id = id;
			this.mid = mid;
			await this.loadData();
		},
		async onReady() {
			const {tcbCloud} = getApp().globalData;
			console.log('onReady');
			const db = tcbCloud.database(); //代码块为cdb
			const { result: {url, title} } = await tcbCloud.callFunction({
				name: 'get-live-url',
				data: { id: this.id }
			});
			this.videoList = url;
			this.setNavigation(title);
			
			// #ifndef H5
			this.player = uni.createVideoContext('player');
			// #endif

      // #ifdef H5
      await this.createPlayer();
      // #endif

		},
		onShow() {
			console.log('onShow');
			timer = setInterval(this.loadData, 30000);
			this.player && this.player.play();
		},
		onHide() {
			clearInterval(timer);
			this.player.pause();
		},
		onUnload() {
			clearInterval(timer);
		},
		computed: {
			videoUrl() {
			  // return this.video3;
				return this.videoList[this.index] && this.videoList[this.index]['url'] || '';
			}
		},
		methods: {
		  async createPlayer() {
        if(!jwplayer.key) jwplayer.key = await this.getJwkey();
		    const player = document.createElement('div');
		    player.id = player.class = 'player';
		    document.querySelector('.player_container').appendChild(player);
        this.player = jwplayer(player).setup({
          file: this.videoUrl,
          autostart: true,
          width: '100%',
          height: 'auto',
          aspectratio: '16:9'
        });
      },
      getJwkey: async () => {
        const {jwkey, tcbCloud} = getApp().globalData;
        if (jwkey) return jwkey;
        const {result} = await tcbCloud.callFunction({
          name: 'jwkey'
        });
        getApp().globalData.jwkey = result;
        return result;
      },
			videoErrorCallback: function(e) {
				console.log('视频错误信息:')
				console.log(e)
			},
			changeUrl: function(index) {
				this.index = index;
        console.log(this.videoUrl);
				// #ifdef H5
				this.player.load([{ file: this.videoUrl }]).play();
				// #endif
			},
			parseData: function(data, flag=0) {
				if(!data.length)	return [];
				return data.slice(flag+1);
			},
			loadData: async function() {
				const {tcbCloud} = getApp().globalData;
				const db = tcbCloud.database(); //代码块为cdb
				const {
					result: {teamInfo, periodGoals, maxPlayers, teamStats, playerStats, matchType}
				} = await tcbCloud.callFunction({
					name: 'matchstat',
					data: {
						mid: this.mid
					}
				});
				console.log(playerStats, this.mid);
				// this.matchstat = result;
				this.teamInfo = teamInfo;
				this.periodGoals = periodGoals;
				this.playerStats = playerStats;
				this.leftPlayers = playerStats.left;
				this.rightPlayers = playerStats.right;
				this.matchType = matchType;
			},
			setNavigation: function(title) {
				uni.setNavigationBarTitle({ title });
			},
      parseName: function (num, name) {
        num = (num.length < 2 ? '0' : '') + num;
        return num + ' ' + name;
      },
      getStatHeadFixed(index) {
        return this.playerStats[index][0]['head'].filter((item, i) => i < 3);
      },
      getStatHeadScroll(index) {
        return this.playerStats[index][0]['head'].filter((item, i) => i > 2);
      },
      getStatBody(index) {
        return this.playerStats[index].filter((item, i) => i > 0);
      },
      getStatBodyFixed(data) {
        return data['row'].filter((item, i) => i < 4);
      },
      getStatBodyScroll(data) {
        return data['row'].filter((item, i) => i > 3);
      }

    }
	}
</script>

<style lang="less">
	page {
		background: #f4f5f7;
	}
	.clearfix {
    clear: both;
  }
	.player_container {
		width: unit(750, rpx);
		height: unit(424, rpx);
	}

	.player {
		width: 100%;
		height: 100%;
	}

	.button-group {
		text-align: center;
	}
	
	.match-info {
		width: unit(750, rpx);
		padding: unit(24, rpx) 0;
		margin: unit(24, rpx) 0;
		background: #fff;
	}
	.table-title {
		color: #969ba3;
	}
	.quarter-item {
		height: unit(80, rpx);
	}
	.quarter-item-title {
		padding-left: unit(24, rpx);
	}
  
  .url-item {
    text-align: center; 
    padding: unit(6, rpx);
  }

  // 技术统计
  .stat-fixed {
    float: left;
    width: unit(360, rpx);
    border-right: 1px solid #afafaf;
  }
  .stat-scroll {
    float: left;
    width: unit(390, rpx);
    white-space: nowrap;
  }
  .stat-head {

  }
  .stat-body {

  }
  .stat-row {
    height: unit(60, rpx);
    line-height: unit(60, rpx);
  }
  .stat-col {
    display: inline-block;
    text-align: center;
    font-size: unit(24, rpx);
    color: rgba(0, 0, 0, .8);
    padding: unit(8, rpx);
  }
  .player-num {
    width: unit(42, rpx);
    text-align: left;
    padding-right: 0;
  }
  .player-head-name {
    width: unit(231, rpx);
    text-align: left;
  }
  .player-body-name {
    width: unit(189, rpx);
    text-align: left;
  }
  .player-time {
    width: unit(64, rpx);
  }
  .player-score {
    width: unit(64, rpx);
  }
  .player-stat-item {
    width: unit(70, rpx);
  }
</style>
