<template>
  <div id="visualization"></div>
</template>


<script>
import { Timeline } from "vis-timeline";
import { DataSet } from "vis-data";

export default {
  name: "Timeline",
  components: {},
  data() {
    return {
      timeline: null,
    };
  },
  props: {
    time: {
      type: Number,
    },
    items: {
      type: [Array, DataSet],
      default: () => [],
    },
    options: {
      type: Object,
    },
  },
  watch: {
    time: {
      deep: true,
      handler: function (newVal) {
        this.timeline.setCustomTime(new Date(newVal));
      },
    },
    items: {
      deep: true,
      handler: function (data) {
        this.timeline.itemsData.update(data);
      },
    },
    options: {
      deep: true,
      handler: function (options) {
        this.timeline.setOptions(options);
        this.timeline.redraw();
      },
    },
  },
  emits: ["timechange"],
  methods: {
    setCustomTime(time) {
      this.timeline.setCustomTime(new Date(time));
    },
  },
  mounted() {
    var container = document.getElementById("visualization");

    var timelineData = this.items;

    var timelineOptions = {
      width: "100%",
      height: "200px",
      showCurrentTime: false,
      selectable: false,
    };

    this.timeline = new Timeline(container, timelineData, timelineOptions);
    this.timeline.addCustomTime(new Date());
    this.timeline.on("timechange", (props) =>
      this.$emit("timechange", props.time)
    );
  },
  beforeUnmount() {
    this.timeline.destroy();
  },
};
</script>

<style scoped>
</style>
