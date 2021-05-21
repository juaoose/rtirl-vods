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
  },
  watch: {
    time: {
      deep: true,
      handler: function (newVal, oldVal) {
        this.timeline.setCustomTime(new Date(newVal));
      },
    },
  },
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
#visualization {
  width: 100%;
  height: 200px;
  border: 1px solid lightgray;
}
</style>
