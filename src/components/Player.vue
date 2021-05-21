<template>
  <div id="twitch-embed"></div>
</template>

<script>
export default {
  name: "Player",
  props: {
    videoId: {
      type: Number,
    },
  },
  data() {
    return {
      currentTime: 0,
      offset: 0,
      parentDomains: ["localhost"],
      player: null,
    };
  },
  methods: {
    initializePlayer() {
      var options = {
        width: "100%",
        height: "100%",
        parent: this.parentDomains,
        video: this.videoId,
        autoplay: false,
        time: "0h0m0s",
      };

      this.player = new Twitch.Player("twitch-embed", options);
    },
  },
  mounted() {
    this.initializePlayer();
  },
};
</script>