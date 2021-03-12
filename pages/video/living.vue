<template>
	<view>
		<!-- <video id="myVideo" src="http://ivi.bupt.edu.cn/hls/cctv5phd.m3u8" @error="videoErrorCallback" controls /> -->
		<video class="player" id="player" :src="videoUrl" @error="videoErrorCallback" autoplay />
		
		<view class="match-info">
			<u-row>
				<u-col style="text-align: center; padding: 6rpx;" span="3" v-for="(item, i) in list" :key="i">
					<u-button @click="changeUrl(i)" size="mini" :type="index===i?'success':''">{{item.title}}</u-button>
				</u-col>
			</u-row>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				index: 0,
				list: []
			}
		},
		onLoad({url}) {
			const {vod_name, vod_play_url, vod_content} = getApp().globalData.videoInfo;
			const m3u8 = vod_play_url.split('$$$')[1];
			this.list = m3u8.split('#').map(item => {
				const [title, url] = item.split('$');
				return {title, url};
			});
		},
		onReady() {
			console.log('onReady');
		},
		onShow() {
			console.log('onShow');
		},
		computed: {
			videoUrl() {
				return this.list.length && this.list[this.index].url;
			}
		},
		methods: {
			videoErrorCallback: function(e) {
				console.log('视频错误信息:')
				console.log(e)
			},
			changeUrl: function(index) {
				this.index = index;
			}

		}
	}
</script>

<style>
.player {width: 750rpx;height: 424rpx;}
.match-info {
	width: 750rpx;
	padding: 24rpx 0;
	margin: 24rpx 0;
	background: #fff;
}

</style>
