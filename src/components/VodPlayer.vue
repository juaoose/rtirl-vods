<template>
  <h2>VOD Replay</h2>

  <div class="flex-container">
    <Player
      :videoId="videoId"
      @load="buildVODTile"
      class="flex-child magenta"
    />
    <Map
      ref="map"
      :playback-time="playbackTime"
      :data="mapData"
      @move="updateTimeLineTime"
      class="flex-child green"
    />
  </div>
  <br />
  <Timeline
    ref="timeline"
    :items="timeline.items"
    :time="timeline.time"
    :options="timeline.options"
    @timechange="updateTime"
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
import {} from "../../lib/LeafletPlayback.min.js";

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
      playbackTime: 0,
      // The way I seem to overwrite the once initialized timeline feels as its not well designed
      timeline: {
        time: null,
        items: [],
        options: {
          width: "100%",
          height: "200px",
          showCurrentTime: false,
          selectable: false,
          start: null,
          end: null,
        },
      },
    };
  },
  methods: {
    buildVODTile(vodDuration) {
      this.vodDuration = vodDuration;
    },
    //Having these two updateTime methods feels wrong
    updateTime(time) {
      this.playbackTime = time.getTime();
    },
    updateTimeLineTime(time) {
      this.timeline.time = time;
    },
    uploadFile(event) {
      this.gpxFile = event.target.files[0];
    },
    handleSubmit() {
      var reader = new FileReader();
      reader.onload = (evt) => {
        try {
          // Should I try to keep this dependency only in the Map component?
          var geoJson = L.Playback.Util.ParseGPX(evt.target.result);
          this.loadTimelineEvents(geoJson);
          this.mapData = geoJson;
        } catch (e) {
          console.error(e);
        }
      };
      reader.readAsText(this.gpxFile);
    },
    loadTimelineEvents(geoJson) {
      this.timeline.options.start = new Date(
        geoJson.features[0].properties.time[0]
      );
      this.timeline.options.end = new Date(
        geoJson.features[0].properties.time[
          geoJson.features[0].properties.time.length - 1
        ]
      );

      if (this.timeline.items.length < 2) {
        var vodEndDate = this.timeline.options.start;
        vodEndDate = new Date(vodEndDate.getTime() + 1000 * this.vodDuration);
        //This fails if you have not set/loaded a VOD?
        this.timeline.items.push(
          {
            start: this.timeline.options.start,
            end: this.timeline.options.end,
            content: "Your activity",
          },
          {
            start: this.timeline.options.start,
            end: vodEndDate,
            content: "Your twitch VOD",
          }
        );
      }
    },
  },
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
