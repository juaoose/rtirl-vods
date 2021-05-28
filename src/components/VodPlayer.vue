<template>
  <h2>VOD Replay</h2>

  <div class="flex-container">
    <Player
      :video-id="video.id"
      :current-time="video.time"
      :playing="video.playing"
      @load="buildVODTile"
      class="flex-child magenta"
    />
    <Map
      ref="map"
      :playback-time="playbackTime"
      :data="mapData"
      @move="updateTimeLineTime"
      @playing="updateVideoPlayback"
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
      <button class="btn">Upload</button>
    </div>
  </form>

  <label for="vod-id">VOD Id</label>
  <input
    v-on:keyup.enter="setVodId"
    :value="video.id"
    type="number"
    id="vod-id"
    name="vod-id"
  />
  <br />
  <label for="vod-offset"
    >VOD Offset (negative means video started earlier than gpx)</label
  >
  <input
    v-on:keyup.enter="setOffset"
    :value="video.offset"
    type="number"
    id="vod-offset"
    name="vod-offset"
  />
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
      vodDuration: null,
      gpxFile: null,
      mapData: null,
      playbackTime: 0,
      video: {
        id: 1026142582,
        offset: 0,
        time: 0,
        playing: false,
      },
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

      if (time >= this.video.offset) {
        const desiredVodTime =
          (time.getTime() -
            this.timeline.options.start -
            this.video.offset * 1000) /
          1000;
        this.video.time = desiredVodTime;
      }
    },
    updateTimeLineTime(time) {
      this.timeline.time = time;
    },
    uploadFile(event) {
      this.gpxFile = event.target.files[0];
    },
    setVodId(event) {
      this.video.id = isNaN(event.target.value)
        ? 0
        : parseInt(event.target.value);
    },
    setOffset(event) {
      this.video.offset = isNaN(event.target.value)
        ? 0
        : parseInt(event.target.value);
    },
    updateVideoPlayback(isPlaying) {
      this.video.playing = isPlaying;
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
        var vodStart = this.timeline.options.end;
        vodStart = new Date(vodStart.getTime() - 1000 * this.vodDuration);
        this.video.offset = (vodStart - this.timeline.options.start) / 1000;
        //This fails if you have not set/loaded a VOD?
        this.timeline.items.push(
          {
            start: this.timeline.options.start,
            end: this.timeline.options.end,
            content: "Your activity",
          },
          {
            start: vodStart,
            end: this.timeline.options.end,
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

.btn {
  background-color: #4caf50;
  color: white;
  text-align: center;
  height: 10%;
  width: 10%;
  border-radius: 25px;
}
</style>
