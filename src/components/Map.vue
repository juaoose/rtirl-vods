<template>
  <div class="flex-container">
    <div id="map" class="flex-child map"></div>
    <button
      v-on:click="changePlaybackStatus()"
      id="play-button"
      type="button"
      v-bind:class="buttonClass"
    >
      {{ status }}
    </button>
  </div>
</template>

<script>
import L from "leaflet";
import {} from "../../lib/LeafletPlayback.min.js";

export default {
  name: "Map",
  data() {
    return {
      videoId: 1023837339,
      parentDomains: ["localhost"],
      playerWidth: "100%",
      playerHeight: "100%",
      player: null,
      map: null,
      gpxFile: null,
      playback: null,
    };
  },
  props: {
    playbackTime: {
      type: Number,
    },
    data: {
      type: Object,
    },
  },
  watch: {
    playbackTime: {
      deep: true,
      handler: function (cursor) {
        this.playback.setCursor(cursor);
      },
    },
    data: {
      deep: true,
      handler: function (data) {
        this.playback.setData(data);
      },
    },
  },
  methods: {
    initializeMap() {
      this.map = new L.Map("map");

      var basemapLayer = new L.TileLayer(
        "https://api.mapbox.com/styles/v1/mapbox/streets-v11/tiles/{z}/{x}/{y}?access_token=pk.eyJ1IjoiampqampqampqampqampqampqaiIsImEiOiJja290MThwMzIwNjNkMndwaHR5djlhYThqIn0.zULjRWtxmVmaWe-FDobI-A",
        {
          attribution:
            '© <a href="https://www.mapbox.com/about/maps/">Mapbox</a> © <a href="http://www.openstreetmap.org/copyright">OpenStreetMap</a> <strong><a href="https://www.mapbox.com/map-feedback/" target="_blank">Improve this map</a></strong>',
        }
      );

      // Center above NYC for now, I should pan to the start of the given track
      this.map.setView([40.79623613970058, -73.95152667405937], 11);

      // Adds the background layer to the map
      this.map.addLayer(basemapLayer);

      // Playback options
      var playbackOptions = {
        playControl: false,
        dateControl: true,
        sliderControl: false,
        tracksLayer: false,
        speed: 1,
      };

      // Initialize playback
      this.playback = new L.Playback(
        this.map,
        null,
        (ms) => this.$emit("move", ms),
        playbackOptions
      );

      this.playback.get;
    },
    changePlaybackStatus() {
      if (this.playback.isPlaying() === false) {
        this.playback.start();
        this.$emit("playing", true);
      } else {
        this.playback.stop();
        this.$emit("playing", false);
      }
    },
  },
  computed: {
    status() {
      return this.playback !== null && !this.playback.isPlaying()
        ? "Play"
        : "Stop";
    },
    buttonClass() {
      return {
        "flex-btn": true,
        playing: this.playback !== null && !this.playback.isPlaying(),
        "not-playing": this.playback !== null && this.playback.isPlaying(),
      };
    },
  },
  mounted: function () {
    this.initializeMap();
  },
};
</script>

<style scoped>
.flex-container {
  display: flex;
  height: 100%;
  position: relative;
}

.flex-child {
  flex: 1;
  height: 100%;
}

.flex-btn {
  text-align: center;
  position: absolute;
  height: 10%;
  width: 10%;
  z-index: 1000;
  border-radius: 25px;
  bottom: 5%;
  right: 1%;
}

.playing {
  background-color: #4caf50;
  color: white;
}

.not-playing {
  background-color: rgba(165, 34, 34, 0.87);
  color: white;
}
</style>