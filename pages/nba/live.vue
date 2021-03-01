<template>
	<view>
		<!-- #ifdef H5 -->
		<view class="player" ref="player">
			<view id="player"></view>
		</view>
		<!-- #endif -->
		
		<!-- #ifndef H5 -->
		<!-- <video id="myVideo" src="http://ivi.bupt.edu.cn/hls/cctv5phd.m3u8" @error="videoErrorCallback" controls /> -->
		<video class="player" :src="videoUrl" @error="videoErrorCallback" autoplay :showProgress="false" />
		<!-- #endif -->
		
		<view class="match-info">
			<u-row>
				<u-col style="text-align: center; padding: 6rpx;" span="3" v-for="(item, i) in videoList" :key="i">
					<u-button @click="changeUrl(i)" size="mini" :type="index===i?'success':''">{{item.title}}</u-button>
				</u-col>
			</u-row>
		</view>
		
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

		<view class="match-info">
			<u-row>
				<u-col span="4" text-align="center">
					<view>球队</view>
				</u-col>
				<u-col span="8">
					<u-row>
						<u-col span="2">
							<view>{{periodGoals.head&&periodGoals.head[0]}}</view>
						</u-col>
						<u-col span="2">
							<view>{{periodGoals.head&&periodGoals.head[1]}}</view>
						</u-col>
						<u-col span="2">
							<view>{{periodGoals.head&&periodGoals.head[2]}}</view>
						</u-col>
						<u-col span="2">
							<view>{{periodGoals.head&&periodGoals.head[3]}}</view>
						</u-col>
						<u-col span="4" text-align="center">
							<view>{{periodGoals.head&&periodGoals.head[4]}}</view>
						</u-col>
					</u-row>
				</u-col>
			</u-row>
			<u-row>
				<u-col span="4">
					<u-row>
						<u-col span="4">
							<u-image mode="aspectFit" width="100%" height="120" :src="teamInfo.leftBadge"></u-image>
						</u-col>
						<u-col span="8">
							<text class="match-team-name">{{teamInfo.leftName}}</text>
						</u-col>
					</u-row>
				</u-col>
				<u-col span="8">
					<u-row>
						<u-col span="2">
							<view>{{periodGoals.rows&&periodGoals.rows[0][1]}}</view>
						</u-col>
						<u-col span="2">
							<view>{{periodGoals.rows&&periodGoals.rows[0][2]}}</view>
						</u-col>
						<u-col span="2">
							<view>{{periodGoals.rows&&periodGoals.rows[0][3]}}</view>
						</u-col>
						<u-col span="2">
							<view>{{periodGoals.rows&&periodGoals.rows[0][4]}}</view>
						</u-col>
						<u-col span="4" text-align="center">
							<view>{{periodGoals.rows&&periodGoals.rows[0][5]}}</view>
						</u-col>
					</u-row>
				</u-col>
			</u-row>
			<u-row>
				<u-col span="4">
					<u-row>
						<u-col span="4">
							<u-image mode="aspectFit" width="100%" height="120" :src="teamInfo.rightBadge"></u-image>
						</u-col>
						<u-col span="8">
							<text class="match-team-name">{{teamInfo.rightName}}</text>
						</u-col>
					</u-row>
				</u-col>
				<u-col span="8">
					<u-row>
						<u-col span="2">
							<view>{{periodGoals.rows&&periodGoals.rows[1][1]}}</view>
						</u-col>
						<u-col span="2">
							<view>{{periodGoals.rows&&periodGoals.rows[1][2]}}</view>
						</u-col>
						<u-col span="2">
							<view>{{periodGoals.rows&&periodGoals.rows[1][3]}}</view>
						</u-col>
						<u-col span="2">
							<view>{{periodGoals.rows&&periodGoals.rows[1][4]}}</view>
						</u-col>
						<u-col span="4" text-align="center">
							<view>{{periodGoals.rows&&periodGoals.rows[1][5]}}</view>
						</u-col>
					</u-row>
				</u-col>
			</u-row>
		</view>
	</view>
</template>

<script>
	let timer = null;
	export default {
		data() {
			return {
				index: 0,
				mid: '',
				videoList: [],
				teamInfo: [],
				periodGoals: [],
				matchType: 0,
				jwkey: ''
			}
		},
		async onLoad({id = 206937, mid }) {
			this.mid = mid;
			console.log(id);
			const db = uniCloud.database(); //代码块为cdb
			const { result } = await uniCloud.callFunction({
				name: 'get-live-url',
				data: { id }
			});
			console.log(result);
			// this.videoList = result;
			this.videoList = [{title: 'sinovision', url: 'https://jpts.sinovision.net/livestream.m3u8'}];
			await this.loadData();
		},
		async onReady() {
			console.log('onReady');
			// #ifdef H5
			
			const {result} = await uniCloud.callFunction({name: 'jwkey'});
			this.jwkey = result;
			
			// const video = document.createElement('div');
			// video.id = 'player';
			// this.$refs.player.$el.appendChild(video);
			
			console.log(this.jwkey);
			jwplayer.key = this.jwkey;
			// jwplayer.key="YJMkUiVsawnQxoJ9GvyKw/TUspiaT1iTKFrbgg==";
			// jwplayer('player');
			// console.log(jwplayer('player'));
			/* this.player = jwplayer('player').setup({
				// autostart: true,
				width: '100%',
				playsinline: true,
				aspectratio:"16:9",
				// file: 'http://ivi.bupt.edu.cn/hls/cctv5phd.m3u8'
				// file: 'http://yun-live.oss-cn-shanghai.aliyuncs.com/record/yunlive/record/yunlive/meeting_1070/2020-11-25-09-27-59_2020-11-25-09-35-52.m3u8'
			}); */
			jwplayer("player").setup({
				autostart: true,
				width: '100%',
				height: '100%',
				// playsinline: true,
				//aspectratio:"16:9",
				file: 'https://jpmedia.sinovision.net/video/auto/2021/0228/0228_191011_qfzhang_trumpmp4_1000k.mp4'
				// file: 'https://jpts.sinovision.net/livestream.m3u8',
				/* palylist: [
					{
						title: 'second',
						file: 'https://jpmedia.sinovision.net/video/auto/2021/0228/0228_191011_qfzhang_trumpmp4_1000k.mp4',
					},
					{
						title: 'first',
						file: 'http://ivi.bupt.edu.cn/hls/cctv5phd.m3u8'
					}	
				] */
				
			});
			
			/*let video = document.createElement('video');
			video.id = 'player';
			video.style = 'width: 100%; height: auto;';
			video.controls = true;
			video.setAttribute('playsinline', true) //IOS微信浏览器支持小窗内播放
			video.setAttribute('webkit-playsinline', true) //这个bai属性是ios 10中设置可以让视频在小du窗内播放，也就是不是全zhi屏播放的video标签的一个属性
			video.setAttribute('x5-video-player-type', 'h5') //安卓 声明启用同层H5播放器 可以在video上面加东西
			let source = document.createElement('source');
			source.src ='http://yun-live.oss-cn-shanghai.aliyuncs.com/record/yunlive/record/yunlive/meeting_1070/2020-11-25-09-27-59_2020-11-25-09-35-52.m3u8';
			source.type = 'application/x-mpegURL';
			video.appendChild(source);
			this.$refs.player.$el.appendChild(video);
			let that = this;
			that.player = videojs('player', {
				poster: 'https://s.yun-live.com/images/20200715/902844b0f1215ff236ad6b1bd1683555.jpg', // 视频封面图地址
				title:'12334567788999900',
				playbackRates: [0.7, 1.0, 1.5, 2.0], //播放速度
				autoDisable: true,
				preload: 'none', //auto - 当页面加载后载入整个视频 meta - 当页面加载后只载入元数据 none - 当页面加载后不载入视频
				language: 'zh-CN',
				fluid: false, // 自适应宽高
				muted: true, //  是否静音
				aspectRatio: '16:9', // 将播放器置于流畅模式，并在计算播放器的动态大小时使用该值。值应该代表一个比例 - 用冒号分隔的两个数字（例如"16:9"或"4:3"）
				controls: true, //是否拥有控制条 【默认true】,如果设为false ,那么只能通过api进行控制了。也就是说界面上不会出现任何控制按钮
				autoplay: 'muted', //如果true,浏览器准备好时开始回放。 autoplay: "muted", // //自动播放属性,muted:静音播放
				loop: true, // 导致视频一结束就重新开始。 视频播放结束后，是否循环播放
				techOrder: ["html5", "flash"], //播放顺序
				screenshot:true,
				controlBar: {
					volumePanel: { //声音样式
						inline: false // 不使用水平方式
					},
					timeDivider: true, // 时间分割线
					durationDisplay: true, // 总时间
					progressControl: true, // 进度条
					remainingTimeDisplay: true, //当前以播放时间
					fullscreenToggle: true, //全屏按钮
					pictureInPictureToggle: true, //画中画

				}
			});*/
			
			// #endif
			
		},
		async onShow() {
			console.log('onShow');
			timer = setInterval(this.loadData, 30000);
			// await this.loadData(mid);
		},
		onHide() {
			clearInterval(timer);
		},
		computed: {
			videoUrl() {
				// return 'http://ivi.bupt.edu.cn/hls/cctv5phd.m3u8';
				return this.videoList[this.index] && this.videoList[this.index]['url'] || '';
			}
		},
		methods: {
			videoErrorCallback: function(e) {
				console.log('视频错误信息:')
				console.log(e)
			},
			changeUrl: function(index) {
				this.index = index;
			},
			loadData: async function() {
				const db = uniCloud.database(); //代码块为cdb
				const {
					result
				} = await uniCloud.callFunction({
					name: 'matchstat',
					data: {
						mid: this.mid
					}
				});
				this.matchstat = result;
				this.teamInfo = result.teamInfo;
				this.periodGoals = result.periodGoals;
				this.matchType = result.matchType;
			}

		}
	}
</script>

<style>
	.player {
		width: 750rpx;
		height: 424rpx;
	}

	.button-group {
		text-align: center;
	}
	
	.match-info {
		width: 750rpx;
		padding: 24rpx;
	}
</style>
