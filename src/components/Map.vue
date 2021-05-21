<template>
  <div
    id="map"
    class="flex-child green"
    style="width: 100%; height: 100%"
  ></div>
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
        if (!this.playback.isPlaying()) {
          this.playback.setCursor(cursor);
        }
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
      //L.Icon.Default.imagePath = './images'
      this.map = new L.Map("map");

      var basemapLayer = new L.TileLayer(
        "https://api.mapbox.com/styles/v1/mapbox/streets-v11/tiles/{z}/{x}/{y}?access_token=pk.eyJ1IjoiampqampqampqampqampqampqaiIsImEiOiJja290MThwMzIwNjNkMndwaHR5djlhYThqIn0.zULjRWtxmVmaWe-FDobI-A"
      );

      // Center map and default zoom level
      this.map.setView([44.61131534, -123.4726739], 9);

      // Adds the background layer to the map
      this.map.addLayer(basemapLayer);

      // Playback options
      var playbackOptions = {
        playControl: true,
        dateControl: false,
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
  },
  mounted: function () {
    this.initializeMap();
  },
};
</script>