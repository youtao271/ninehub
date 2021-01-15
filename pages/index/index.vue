<template>
	<view>
		<u-card v-for="(item, index) in matches" :title="item.away+'vs'+item.home" :key="index">
			<view class="" slot="body">
				<view class="u-body-item u-flex u-border-bottom u-col-between u-p-t-0">
					<image :src="item.awayImg" mode="aspectFill"></image>
				</view>
				<view class="u-body-item u-flex u-row-between u-p-b-0">
					<image :src="item.homeImg" mode="aspectFill"></image>
				</view>
			</view>
			<view class="" slot="foot">
				<u-icon name="chat-fill" size="34" color="" label="比分"></u-icon>
			</view>
		</u-card>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				title: 'Hello',
				matches: []
			}
		},
		async onLoad() {
			const db = uniCloud.database(); //代码块为cdb
			// 使用uni-clientDB
			// const {result: {data}} = await db.collection('nba_match_list').get();
			const {
				result: {
					data
				}
			} = await db.collection('nba_match_list').where('startTime >= 1610611200000').get();
			this.matches = data;

			/* db.collection('nba_match_list')
			  .where({
			    mid: "100000:55077563" //传统MongoDB写法，不是jql写法。实际开发中推荐使用jql写法
			  }).get()
			  .then((res)=>{
			    // res 为数据库查询结果
				console.log(res);
			  }).catch((err)=>{
			    console.log(err.code); // 打印错误码
			    console.log(err.message); // 打印错误内容
			  }) */
		},
		methods: {

		}
	}
</script>

<style>
	.content {
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
	}

	.logo {
		height: 200rpx;
		width: 200rpx;
		margin-top: 200rpx;
		margin-left: auto;
		margin-right: auto;
		margin-bottom: 50rpx;
	}

	.text-area {
		display: flex;
		justify-content: center;
	}

	.title {
		font-size: 36rpx;
		color: #8f8f94;
	}
</style>
