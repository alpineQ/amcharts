<template>
  <div id="chartdiv" class="chartdiv"></div>
</template>

<script setup lang="ts">
import { onMounted, onUnmounted } from 'vue'

import * as am5 from '@amcharts/amcharts5'
import * as am5map from "@amcharts/amcharts5/map"
import am5geodata_worldLow from "@amcharts/amcharts5-geodata/worldLow"
import am5themes_Animated from "@amcharts/amcharts5/themes/Animated"

// import am5geodata_data_countries2 from "@amcharts/amcharts5-geodata/data/countries2";
// let franceMaps = am5geodata_data_countries2["FR"];
// console.log(franceMaps)

let chart: am5map.MapChart;

function renderChart() {
  let root = am5.Root.new("chartdiv");
  root.setThemes([
    am5themes_Animated.new(root)
  ]);
  chart = root.container.children.push(
    am5map.MapChart.new(root, {
      projection: am5map.geoMercator(),
    })
  );
  let polygonSeries = chart.series.push(
    am5map.MapPolygonSeries.new(root, {
      geoJSON: am5geodata_worldLow,
      exclude: ["AQ"]
    })
  );
  polygonSeries.mapPolygons.template.setAll({
    tooltipText: "{name}",
    toggleKey: "active",
    interactive: true
  });

  polygonSeries.mapPolygons.template.states.create("hover", {
    fill: root.interfaceColors.get("primaryButtonHover")
  });

  polygonSeries.mapPolygons.template.states.create("active", {
    fill: root.interfaceColors.get("primaryButtonHover")
  });

  let dataItem = polygonSeries.getDataItemById("RU");
  console.log(dataItem)
  // let polygon = countryDataItem.get("mapPolygon");

  let previousPolygon: am5map.MapPolygon | undefined;
  polygonSeries.mapPolygons.template.on("active", function (_, target) {
    if (previousPolygon && previousPolygon != target) {
      previousPolygon.set("active", false);
    }
    console.log(target)
    if (target && target.get("active")) {
      const item = target.dataItem
      if (item) {
        console.log(item)
        polygonSeries.zoomToDataItem(item as am5.DataItem<am5map.IMapPolygonSeriesDataItem>);
      }
    }
    else {
      chart.goHome();
    }
    previousPolygon = target;
  });

  chart.set("zoomControl", am5map.ZoomControl.new(root, {}));

  chart.appear(1000, 100);
}

onMounted(() => {
  renderChart();
})
onUnmounted(() => {
  if (chart) {
    chart.dispose();
  }
})

</script>
<style>
.chartdiv {
  width: 100%;
  height: 500px;
}
</style>