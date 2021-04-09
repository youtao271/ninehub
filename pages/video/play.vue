<template>
	<view>
		<!-- <video id="myVideo" src="http://ivi.bupt.edu.cn/hls/cctv5phd.m3u8" @error="videoErrorCallback" controls /> -->
		<!--<video class="player" id="player" :src="videoUrl" @error="videoErrorCallback" autoplay />-->
    <view class="player_container">
      <Player :file="videoUrl" :autoplay="true" :playnext="playnext" />
    </view>

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
import Player from "../../components/Player";
	export default {
		data() {
			return {
				index: 0,
				list: [],
        title: ''
			}
		},
    components: {Player},
		onLoad({url}) {
			const {vod_name, vod_play_url, vod_content} = getApp().globalData.videoInfo;
			const m3u8 = vod_play_url.split('$$$')[1];
			this.title = vod_name;
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
			this.setNavigation(this.list[this.index].title)
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
        this.setNavigation()
      },
      setNavigation: function() {
			  const title = this.title + '_' + this.list[this.index].title
        uni.setNavigationBarTitle({ title });
			},
      playnext: function () {
			  this.index++;
			  this.setNavigation();
      }
		}
	}
</script>

<style lang="less">
.player_container {
  width: unit(750, rpx);
  height: unit(424, rpx);
}
.match-info {
	width: unit(750, rpx);
	padding: unit(24, rpx) 0;
	margin: unit(24, rpx) 0;
	background: #fff;
}

</style>
