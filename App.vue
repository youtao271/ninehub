<style lang="scss">
	@import "uview-ui/index.scss";
</style>

<script>
	export default {
		globalData: {  
			videoInfo: null,
			aliCloud: null,
			tcbCloud: null
		},
		onLaunch: function() {
			console.log('App Launch');
			getApp().globalData.aliCloud = uniCloud.init({
			  provider: 'aliyun',
			  spaceId: '8a838cb8-94a2-4e7f-a5cd-4b379370ad6e',
			  clientSecret: 'Y2AJZFwG/7KUhB6CHC6cfQ=='
			});
			getApp().globalData.tcbCloud = uniCloud.init({
			  provider: 'tencent',
			  spaceId: 'tcb-lq2bufocn2xnmhi96b1d9-9a5734'
			});
		},
		onShow: function() {
			console.log('App Show')
		},
		onHide: function() {
			console.log('App Hide')
		}
	}
</script>
