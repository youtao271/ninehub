<template>
	<view>
		<!-- <video id="myVideo" src="http://ivi.bupt.edu.cn/hls/cctv5phd.m3u8" @error="videoErrorCallback" controls /> -->
		<video class="player" :src="videoUrl" @error="videoErrorCallback" autoplay :showProgress="false"/>
		<u-row>
			<u-col style="text-align: center; padding: 6rpx;" span="3" v-for="(item, i) in videoList" :key="i">
				<u-button @click="changeUrl(i)" size="mini" :type="index===i?'success':''">{{item.title}}</u-button>
			</u-col>
		</u-row>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				index: 0,
				videoList: []
			}
		},
		async onLoad({id=206937}) {
			console.log(id);
			const db = uniCloud.database(); //代码块为cdb
			const { result } = await uniCloud.callFunction({
				name: 'get-live-url',
				data: { id }
			});
			
			console.log(result);
			this.videoList = result;
		},
		onReady() {
			console.log('onReady');
		},
		onShow() {
			console.log('onShow');
		},
		computed: {
			videoUrl() {
				console.log(this.index);
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
			}

		}
	}
</script>

<style>
.player {width: 750rpx;height: 424rpx;}
.button-group {text-align: center;}
</style>
