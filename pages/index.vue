<template >
  <div class="content">
    <v-container>
      <v-row>
        <v-col cols="7" class="pa-0 pr-5">
          <client-only v-if="showPlayer">
            <vimeo-player
              class="video--content"
              ref="player"
              :video-id="videoID"
              :options="options"
              @timeupdate="timeUpdate"
              @ended="ended"
            />
          </client-only>

          <description-video
            v-if="showPlayer"
            :item="items[index]"
          ></description-video>

          <comments-content class="mt-8" v-if="showPlayer"></comments-content>
        </v-col>
        <v-col cols="5" class="white">
          <h1>Contenido</h1>
          <play-list-item
            v-for="(item, index) in items"
            :key="index"
            :item="item"
            :index="index"
          ></play-list-item>
        </v-col>
      </v-row>
    </v-container>
  </div>
</template>

<script>
export default {
  name: "Home",
  middleware: "auth",
  async mounted() {
    await this.init();
    await this.getVideoId();
  },
  components: {
    playListItem: () => import("../components/playList/playListItem"),
    descriptionVideo: () => import("../components/descriptionVideo"),
    commentsContent: () => import("../components/comments/commentsContent"),
  },
  data: () => ({
    videoID: "",
    options: {
      autoplay: true,
      responsive: true,
    },
    items: [],
    showPlayer: false,
    lastSecondUpdatedVideo: 0,
  }),
  methods: {
    timeUpdate(e) {
      const percentProgress = e.percent * 10;
      this.$store.dispatch("updateProgress", percentProgress);

      if (e.seconds >= this.lastSecondUpdatedVideo)
        this.updatedProgress(e.seconds, percentProgress);

      // this.$store.state.listContent[this.index].progress = percentProgress;
    },
    async init() {
      if (this.$store.state.listContent.length > 0) {
        this.items = this.$store.state.listContent;
        return;
      }
      try {
        const { data } = await this.$axios.post("content");
        this.$store.dispatch("listContentAdd", data.content);
        this.items = this.$store.state.listContent;
      } catch (error) {
        console.warn(error);
      }
    },
    getVideoId() {
      const index = this.$store.state.index;
      const video = this.$store.state.listContent[index].video_url;
      this.videoID = video.split("/")[3];
      this.showPlayer = true;
    },
    async updatedProgress(seconds, percentProgress) {
      try {
        this.lastSecondUpdatedVideo = seconds + 5;
        const progress = parseInt(percentProgress);
        const data = {};
        data.progress = progress;
        const idContent = this.$store.state.listContent[this.index].id;
        await this.$axios.post(`content/${idContent}/progress`, data);
      } catch (error) {
        console.warn(error);
      }
    },
    async ended() {
      try {
        const data = {};
        data.progress = 10;
        const idContent = this.$store.state.listContent[this.index].id;
        await this.$axios.post(`content/${idContent}/progress`, data);
      } catch (error) {
        console.warn(error);
      }
    },
  },
  computed: {
    index() {
      return this.$store.state.index;
    },
  },
  watch: {
    index: function () {
      this.lastSecondUpdatedVideo = 0;
      this.getVideoId();
      this.$refs.player.play();
    },
  },
};
</script>

<style lang="scss" scoped>
.video--content {
  width: 100%;
}

.play-list--items {
  max-height: 60vh;
  overflow-y: auto;
  overflow-x: hidden;
}
</style>
