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
    currentTime: {
      type: Number,
      default: 0,
    },
    offset: {
      type: Number,
      default: 0,
    },
  },
  watch: {
    videoId: {
      deep: false,
      handler: function (videoId) {
        this.player.setVideo(videoId, 0);
      },
    },
    currentTime: {
      deep: true,
      handler: function (timestamp) {
        this.player.seek(timestamp);
      },
    },
  },
  emits: ["load"],
  data() {
    return {
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
      this.player.addEventListener(Twitch.Player.READY, () => {
        // Since I can't figure out why duration cant be retrieved at this point
        this.player.seek(0);
        this.player.setVolume(0.5);
        this.player.play();
      });
      this.player.addEventListener(Twitch.Player.PLAY, () => {
        // Cant figure out why I cant retrieve this thingie on Twitch.Player.READY...
        const vodDuration = this.player.getDuration();
        this.$emit("load", this.player.getDuration(vodDuration));
      });
    },
  },
  mounted() {
    this.initializePlayer();
  },
};
</script>