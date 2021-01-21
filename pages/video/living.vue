<template>
	<view>
		<!-- <video id="myVideo" src="http://ivi.bupt.edu.cn/hls/cctv5phd.m3u8" @error="videoErrorCallback" controls /> -->
		<video class="player" :src="url" @error="videoErrorCallback" autoplay :showProgress="false"/>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				index: 0,
				url: 'https://tx-playback.video.yximgs.com/mediacloud/gzone/game_video/hls/fCFBRFnV0_ehAhyGU1smaZAbzL7LmhseaTnI8m9IKEo.m3u8'
			}
		},
		onLoad({url}) {
			console.log(url);
			this.url = url;
		},
		onReady() {
			console.log('onReady');
		},
		onShow() {
			console.log('onShow');
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
.button-group {text-align: center;}
</style>
