<template>
  <!-- #ifdef H5 -->
  <view class="player_container"></view>
  <!-- #endif -->

  <!-- #ifndef H5 -->
  <video class="player" id="player" :src="file" autoplay :showProgress="showProgress"
         @error="videoErrorCallback"
         @ended="playnext"
  />
  <!-- #endif -->

</template>

<script>
export default {
  props: ['file','image','showProgress', 'autoplay', 'playnext'],
  data() {
    return {};
  },
  async mounted() {

    console.log(this.file);

    // #ifndef H5
    this.player = uni.createVideoContext('player');
    // #endif

    // #ifdef H5
    this.player = await this.createPlayer();
    this.player.on('complete', this.playnext);
    // #endif

  },
  watch: {
    // #ifdef H5
    file: {
      handler:async function(newVal,oldVal) {
        this.player.load([{file: newVal}]).play();
      }
    }
    // #endif
  },
  methods: {
    async createPlayer() {
      if(!jwplayer.key) jwplayer.key = await this.getJwkey();
      const player = document.createElement('div');
      player.id = player.class = 'player';
      document.querySelector('.player_container').appendChild(player);
      return jwplayer(player).setup({
        file: this.file,
        autostart: this.autoplay,
        width: '100%',
        height: 'auto',
        aspectratio: '16:9'
      });
    },
    getJwkey: async () => {
      const {jwkey, tcbCloud} = getApp().globalData;
      if (jwkey) return jwkey;
      const {result} = await tcbCloud.callFunction({
        name: 'jwkey'
      });
      getApp().globalData.jwkey = result;
      return result;
    },
    videoErrorCallback: function () {
      return true;
    }
  }
}
</script>

<style lang="less" scoped>

.player_container, .player {
  width: 100%;
  height: 100%;
}

</style>