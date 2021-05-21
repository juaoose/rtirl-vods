<template>
  <h2>VOD Replay</h2>

  <div class="flex-container">
    <Player :videoId="1023837339" class="flex-child magenta" />
    <Map
      ref="map"
      v-on:move="updateTimeLineTime"
      :playback-time="playbackTime"
      :data="mapData"
      class="flex-child green"
    />
  </div>
  <br />
  <Timeline
    ref="timeline"
    v-on:timechange="updateTime"
    :items="timelineItems"
    :time="timelineTime"
  />

  <form @submit.prevent="handleSubmit">
    <div class="form-group">
      <input type="file" @change="uploadFile" />
    </div>

    <div class="form-group">
      <button class="btn btn-success btn-block btn-lg">Upload</button>
    </div>
  </form>
</template>

<script>
import Map from "./Map.vue";
import Player from "./Player.vue";
import Timeline from "./Timeline.vue";

export default {
  name: "VodPlayer",
  components: { Map, Player, Timeline },
  data() {
    return {
      videoId: 1023837339,
      parentDomains: ["localhost"],
      playerWidth: "100%",
      playerHeight: "100%",
      player: null,
      gpxFile: null,
      mapData: null,
      timelineTime: 0,
      playbackTime: 0,
      timelineItems: [
        {
          start: "2021-05-21",
          end: "2021-05-22",
          content: "Your activity",
        },
        {
          start: "2021-05-21",
          end: "2021-05-22",
          content: "Your twitch VOD",
        },
      ],
    };
  },
  methods: {
    updateTime(time) {
      this.playbackTime = time.getTime();
    },
    updateTimeLineTime(time) {
      this.timelineTime = time;
    },
    uploadFile(event) {
      this.gpxFile = event.target.files[0];
    },
    handleSubmit() {
      var reader = new FileReader();
      reader.onload = (evt) => {
        this.mapData = evt.target.result;
      };
      reader.readAsText(this.gpxFile);
    },
  },
  mounted: function () {},
};
</script>

<style scoped>
.flex-container {
  display: flex;
  height: 40vh;
  min-height: 40vh;
}

.flex-child {
  flex: 1;
  height: 100%;
  border: 2px solid black;
}

.flex-child:first-child {
  margin-right: 20px;
}
</style>
