<template>
  <div id="webtrees-fan-chart-container"></div>
</template>
<router>
{
    name: 'decendent.show'
}
</router>
<script>
import { FanChart } from "/assets/js/fan-chart/modules/index";
import { ref, computed, useStore } from 'vue';
export default {
  layout: 'auth',
  meta: {
    permission: { name: 'decendentchart menu'},
    title: 'DecendentChart - Show'
  },
  setup() {
    function fetchData() {
      this.$axios
        .$get("/api/trees/show")
        // .then((res) => res.json())
        // .then((response) => response.json())
        .then((res) => {
          // console.log(res.data);
          const pedigreeChart = new FanChart("webtrees-fan-chart-container", {
            labels: {
              zoom: "Use Ctrl + scroll to zoom in the view",
              move: "Move the view with two fingers",
            },
            generations: 4,
            defaultColor: "#0000FF",
            fontColor: "#0000FF",
            showEmptyBoxes: "up",
            treeLayout: "right",
            rtl: "rtl",
          });

          pedigreeChart.cssFile = "assets/css/fanchart/svg.css";
          // Draw chart
          console.log(res);
          pedigreeChart.draw(res);

          const fanChart = FanChart("#webtrees-fan-chart-container", {
            labels: res.labels,
            generations: 4,
            fanDegree: 90,
            defaultColor: "#0000FF",
            fontColor: "#0000FF",
            fontScale: parseInt(storage.read("fontScale")),
            // fontColor: data.fontColor,
            hideEmptySegments: storage.read("hideEmptySegments"),
            showColorGradients: storage.read("showColorGradients"),
            showParentMarriageDates: storage.read("showParentMarriageDates"),
            rtl: true,
            innerArcs: parseInt(storage.read("innerArcs")),
          });
        });
    }
    onMounted(() => {
      this.fetchData();
    })
  }
}

</script>

<style lang="scss">
</style>