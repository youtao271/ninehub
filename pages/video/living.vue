<template>
	<view>
		<!-- <video id="myVideo" src="http://ivi.bupt.edu.cn/hls/cctv5phd.m3u8" @error="videoErrorCallback" controls /> -->
		<!--<video class="player" id="player" :src="videoUrl" @error="videoErrorCallback" autoplay />-->
    <view class="player_container">
      <Player :file="src" :autoplay="true" />
    </view>
	</view>
</template>

<script>
import Player from "../../components/Player";
	export default {
		data() {
			return {
				index: 0,
				src: '',
        title: ''
			}
		},
    components: {Player},
		onLoad({url}) {
			this.src = url;
		},
		onReady() {
			console.log('onReady');
		},
		onShow() {
			console.log('onShow');
			// this.setNavigation(this.list[this.index].title)
		},
		methods: {
      setNavigation: function() {
			  const title = this.title + '_' + this.list[this.index].title
        uni.setNavigationBarTitle({ title });
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
