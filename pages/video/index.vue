<template>
  <view>
    <view class="status_bar" />
    <u-tabs :list="tabs" :current="tabIndex" @change="changeTab" />
    <u-search
        class="search-form"
        placeholder="请输入关键词"
        v-model="keyword"
        bg-color="#ffffff"
        :action-style="{color: '#409eff'}"
        @custom="search"
        @search="search"
    />
    <u-loading :show="loaded" size="40" />
    <u-waterfall v-model="list" ref="uWaterfall" add-time="10">
      <template v-slot:left="{leftList}">
        <view class="item" v-for="(item, index) in leftList" :key="index" @click="play(item.vod_id)">
          <u-tag v-if="item.vod_remarks" class="tag-banner" :text="item.vod_remarks" type="success" mode="dark" />
          <!--<u-lazy-load border-radius="10" :image="pic" :index="index"></u-lazy-load>-->
           <u-lazy-load border-radius="10" :image="item.vod_pic" :index="index"></u-lazy-load>
          <view class="item-title">{{item.vod_name}}</view>
        </view>
      </template>
      <template v-slot:right="{rightList}">
        <view class="item" v-for="(item, index) in rightList" :key="index" @click="play(item.vod_id)">
          <u-tag v-if="item.vod_remarks" class="tag-banner" :text="item.vod_remarks" type="success" mode="dark" />
          <!--<u-lazy-load border-radius="10" :image="pic" :index="index"></u-lazy-load>-->
           <u-lazy-load border-radius="10" :image="item.vod_pic" :index="index"></u-lazy-load>
          <view class="item-title">{{item.vod_name}}</view>
        </view>
      </template>
    </u-waterfall>
    <u-loadmore v-if="!loaded" :status="status" :icon-type="iconType" :load-text="loadText" />
  </view>
</template>

<script>
let timer = null;
export default {
  data() {
    return {
      pic: 'https://cdnvideo.sinovision.net/attachments/video/20210310/2012/1615425160-8176.jpg',
      loaded: true,
      status: 'loadmore',
      iconType: 'circle',
      loadText: {
        loadmore: '上拉或点击加载更多',
        loading: '努力加载中',
        nomore: '我是有底线的'
      },
      keyword: '',
      tid: '',
      page: 1,
      list: [],
      tabs: [
        {name: '最新'},
        {name: '电影', tid: 2},
        {name: '电视剧', tid: 3},
        {name: '动漫', tid: 4},
        {name: '综艺', tid: 5},
        {name: '采集'}
      ],
      tabIndex: 0
    }
  },
  async onLoad() {
    await this.getData();
    this.loaded = false;
  },
  async onShow() {
    console.log('show');
  },
  onHide() {

  },
  async onPullDownRefresh() {
    this.page = 1;
    this.$refs.uWaterfall.clear();
    await this.getData();
    uni.stopPullDownRefresh();
  },
  async onReachBottom() {
    if(this.status !== 'nomore'){
      this.status = 'loading';
      await this.loading();
    }
  },
  methods: {
    getData: async function() {
      const {tcbCloud} = getApp().globalData;
      const { result } = await tcbCloud.callFunction({
        name: 'video',
        data: {
          // type: 'list',
          wd: this.keyword,
          pg: this.page,
          t: this.tid,
          h: this.keyword ? '' : 24,
          // ids: '69346,48602'
        }
      });
      this.status = result.length ? 'loadmore' : 'nomore';
      this.list = this.page===1 ? [...result] : [...this.list, ...result];
      console.log(this.page, this.list.length);
    },
    loading: async function() {
      this.page++;
      await this.getData();
    },
    play: function(id) {
      getApp().globalData.videoInfo = this.list.find(item => item.vod_id===id);
      uni.navigateTo({
        url: `/pages/video/play`
      })
    },
    search: async function(val) {
      this.keyword = val;
      this.page = 1;
      this.$refs.uWaterfall.clear();
      await this.getData();
    },
    changeTab: function(index) {
      this.tabIndex = index;
      this.tid = this.tabs[index].tid;
    },
    parseUrl: data => {
      return data.split('$$$')[1];
    },
    collect: async function () {
      const {aliCloud} = getApp().globalData;
      const videos = aliCloud.database().collection('videos');
      console.log('collect');
      // const {result: {updated}} = await videos.where("id > 0").update({source: 'kb'});
      // console.log(updated);
      // return;
      let pg = 1360, data = await this.getList(pg);
      while (data.length) {
        const addDatas = [];
        for (const item of data) {
          const {
            vod_id: id, type_id, type_id_1, vod_name, vod_en, vod_status, vod_letter, vod_pic, vod_actor,
            vod_director, vod_blurb, vod_remarks, vod_serial, vod_area, vod_lang, vod_year, vod_time, vod_time_add,
            vod_content, vod_play_from, vod_play_url, vod_down_from, vod_down_url, type_name
          } = item;

          const img = vod_pic;//await getImage(id, vod_pic);
          const videoInfo = {
            tid: type_id, pid: type_id_1, name: vod_name, name_en: vod_en, status: vod_status, letter: vod_letter,
            img, actor: vod_actor, director: vod_director, blurb: vod_blurb, remarks: vod_remarks,
            serial: vod_serial, area: vod_area, lang: vod_lang, year: vod_year, time: vod_time, add_time: vod_time_add,
            content: vod_content, play_from: this.parseUrl(vod_play_from), play_url: this.parseUrl(vod_play_url), down_from: vod_down_from,
            down_url: vod_down_url, type_name, source: 'bd'
          }
          const {result: {data: info}} = await videos.where(`id == ${id}`).get({getOne: true});
          if(info) {
            const {result: {updated}} = await videos.where({id}).update(videoInfo);
            console.log('update:' + pg + '---' + updated);
          } else {
            addDatas.push({...videoInfo, id})
          }
        }
        if(addDatas.length) {
          const {result: {inserted}} = await videos.add(addDatas);
          console.log('insert:' + pg + '---' + inserted);
        }
        // break;
        data = await this.getList(++pg);
      }
      // console.log(info);
      // console.log(videoInfo);

    },
    getList: async function(pg) {
      const {aliCloud} = getApp().globalData;
      try {
        const { result } = await aliCloud.callFunction({
          name: 'collect_video',
          data: { pg }
        });
        return result;
      } catch (e) {
        console.log('请求超时');
        return await this.getList(pg);
      }
    }
  },
  watch: {
    tabIndex: async function(value) {
      if(value === 5) {
        await this.collect();
        return;
      }
      this.keyword = '';
      this.page = 1;
      this.$refs.uWaterfall.clear();
      await this.getData();
    }
  }
}
</script>

<style lang="less" scoped>
page {
  background-color: rgb(240, 240, 240);
}
.status_bar {
  height: var(--status-bar-height);
  width: 100%;
}
.search-form {
  margin: unit(12, rpx) unit(20, rpx) unit(4, rpx);

}
.item {
  border-radius: unit(16, rpx);
  margin: unit(10, rpx);
  background-color: #ffffff;
  padding: unit(16, rpx);
  position: relative;
}
.item-title {
  width: 100%;
  font-size: unit(30, rpx);
  margin-top: unit(10, rpx);
  //color: $u-main-color;
}
.tag-banner {
  position: absolute;
  top: unit(24, rpx);
  right: unit(24, rpx);
  padding: unit(10, rpx) unit(16, rpx);
  z-index: 99;
}
.tag-update {

}
</style>
