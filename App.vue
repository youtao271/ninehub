<style lang="scss">
	@import "uview-ui/index.scss";
</style>

<script>
	export default {
		onLaunch: function() {
			const year = new Date().getFullYear();
			const month = new Date().getMonth() + 1;
			const day = new Date().getDate();
			this.globalData.today = `${year}-${month<10?'0':''}${month}-${day<10?'0':''}${day}`;

			console.log('App Launch', this.globalData);
		},
		onShow: function() {
			console.log('App Show')
		},
		onHide: function() {
			console.log('App Hide')
		}
	}
</script>
