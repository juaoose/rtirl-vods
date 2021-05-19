<template>
  <h2>Hello I am a bot</h2>

  <div class="flex-container">
    <div id="twitch-embed" class="flex-child magenta"></div>

    <div
      id="map"
      class="flex-child green"
      style="width: 100%; height: 100%"
    ></div>
  </div>
  <br />

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
import mapboxgl from "mapbox-gl";
import "mapbox-gl/dist/mapbox-gl.css";
import { gpx } from "@tmcw/togeojson";

export default {
  name: "VodPlayer",
  components: {},
  data() {
    return {
      videoId: 1023837339,
      parentDomains: ["localhost"],
      playerWidth: "100%",
      playerHeight: "100%",
      player: null,
      map: null,
      gpxFile: null,
      geoJson: null,
      marker: null,
      counter: 0,
      sessionSteps: null,
      sessionStartTime: null,
      animation: null,
      currentPlayerTime: 0,
    };
  },
  methods: {
    initializeMap() {
      mapboxgl.accessToken =
        "pk.eyJ1IjoiampqampqampqampqampqampqaiIsImEiOiJja290MThwMzIwNjNkMndwaHR5djlhYThqIn0.zULjRWtxmVmaWe-FDobI-A";
      this.map = new mapboxgl.Map({
        container: "map",
        style: "mapbox://styles/mapbox/streets-v11",
      });

      this.map.on("load", () => console.log(this.map));
    },
    initializePlayer() {
      var options = {
        width: this.playerWidth,
        height: this.playerHeight,
        parent: this.parentDomains,
        video: this.videoId,
        autoplay: false,
        time: "0h0m0s",
      };

      this.player = new Twitch.Player("twitch-embed", options);
    },
    registerEventListeners() {
      this.player.addEventListener(Twitch.Player.PAUSE, this.pauseVideo);
      this.player.addEventListener(Twitch.Player.PLAY, this.resumeVideo);
      this.player.addEventListener(Twitch.Player.READY, () => {
        console.log(this.player.getDuration());
      });
    },
    pauseVideo() {
      console.log("Video was paused");
      this.currentPlayerTime = this.player.getCurrentTime();
      cancelAnimationFrame(this.animation);
    },
    resumeVideo() {
      var positionInSeconds = this.player.getCurrentTime();
      if (this.currentPlayerTime != positionInSeconds) {
        if (this.geoJson != null) {
          var estimatedPlayerDate = new Date(this.sessionStartTime.valueOf());
          console.log(estimatedPlayerDate);
          estimatedPlayerDate.setSeconds(
            estimatedPlayerDate.getSeconds() + positionInSeconds
          );

          var desiredTimestampIndex = this.findClosestTimestamp(
            this.geoJson.features[0].properties.coordinateProperties.times,
            estimatedPlayerDate,
            0,
            this.sessionSteps
          );
          console.log("Desired timestamp " + desiredTimestampIndex);
          this.counter = desiredTimestampIndex;

          this.animate();
        }
      } else if (this.geoJson != null) {
        this.animate();
      }
    },
    endVideo() {
      console.log("Video playback ended");
      cancelAnimationFrame(this.animation);
    },
    uploadFile(event) {
      this.gpxFile = event.target.files[0];
    },
    handleSubmit() {
      var reader = new FileReader();
      reader.onload = (evt) => {
        try {
          var data = new DOMParser().parseFromString(
            evt.target.result,
            "text/xml"
          );
          this.geoJson = gpx(data);
          this.addGeoJson(this.geoJson);
        } catch (e) {
          console.error(e);
        }
      };
      reader.readAsText(this.gpxFile);
    },
    addGeoJson(geoJson) {
      console.log(geoJson);
      this.sessionSteps = geoJson.features[0].geometry.coordinates.length;
      this.sessionStartTime = new Date(geoJson.features[0].properties.time);
      this.map.addSource("route", {
        type: "geojson",
        data: geoJson,
      });

      this.map.addLayer({
        id: "route",
        type: "line",
        source: "route",
        layout: {
          "line-join": "round",
          "line-cap": "round",
        },
        paint: {
          "line-color": "#888",
          "line-width": 3,
        },
      });

      this.marker = new mapboxgl.Marker({
        color: "#EC0868",
        className: "marker",
      });
      this.counter = 0;
      this.marker
        .setLngLat(geoJson.features[0].geometry.coordinates[0])
        .addTo(this.map);
    },
    animate() {
      this.marker.setLngLat(
        this.geoJson.features[0].geometry.coordinates[this.counter]
      );
      this.counter++;
      if (this.counter < this.sessionSteps) {
        this.animation = requestAnimationFrame(this.animate);
      } else {
        this.counter = 0;
      }
    },
    findClosestTimestamp(timestampArray, targetTimestamp, start, finish) {
      const center = Math.floor((start + finish) / 2);
      if (
        targetTimestamp.getTime() == new Date(timestampArray[center]).getTime()
      ) {
        return center;
      }
      if (finish - 1 === start) {
        return start;
      }
      if (
        targetTimestamp.getTime() > new Date(timestampArray[center]).getTime()
      ) {
        return this.findClosestTimestamp(
          timestampArray,
          targetTimestamp,
          center,
          finish
        );
      }
      if (
        targetTimestamp.getTime() < new Date(timestampArray[center]).getTime()
      ) {
        return this.findClosestTimestamp(
          timestampArray,
          targetTimestamp,
          start,
          center
        );
      }
    },
  },
  mounted: function () {
    this.initializeMap();
    this.initializePlayer();
    this.registerEventListeners();
  },
};
</script>

<style scoped>
a {
  color: #42b983;
}

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
