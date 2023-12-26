<template>
  <div id="chartdiv" class="chartdiv"></div>
</template>

<script setup lang="ts">
import { onMounted, onUnmounted } from 'vue'
import * as am5 from '@amcharts/amcharts5'
import * as am5map from "@amcharts/amcharts5/map"
import am5geodata_worldLow from "@amcharts/amcharts5-geodata/worldLow"
import am5themes_Animated from "@amcharts/amcharts5/themes/Animated"


const props = defineProps<{
  code: string,
  width: number,
  height: number
}>()

let chart: am5map.MapChart;
let polygonSeries: am5map.MapPolygonSeries
let previousPolygon: am5map.MapPolygon | undefined;

onMounted(() => {
  const root = am5.Root.new("chartdiv");
  root.setThemes([am5themes_Animated.new(root)]);
  chart = root.container.children.push(
    am5map.MapChart.new(root, {
      panX: "rotateX",
      panY: "rotateY",
      projection: am5map.geoOrthographic(),
    })
  );
  polygonSeries = chart.series.push(
    am5map.MapPolygonSeries.new(root, {
      geoJSON: am5geodata_worldLow,
      // exclude: ["AQ"]
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

  let graticuleSeries = chart.series.unshift(
    am5map.GraticuleSeries.new(root, {
      step: 10,
      stroke: am5.color("#ffffff"),
    })
  );
  graticuleSeries.mapLines.template.set("strokeOpacity", 0.1)

  polygonSeries.mapPolygons.template.on("active", function (_, target) {
    if (previousPolygon && previousPolygon != target)
      previousPolygon.set("active", false);
    if (target && target.get("active")) {
      let centroid = target.geoCentroid();
      if (centroid) {
        chart.animate({ key: "rotationX", to: -centroid.longitude, duration: 1500, easing: am5.ease.inOut(am5.ease.cubic) });
        chart.animate({ key: "rotationY", to: -centroid.latitude, duration: 1500, easing: am5.ease.inOut(am5.ease.cubic) });
      }
      chart.animate({ key: 'zoomLevel', to: 2, duration: 1500, easing: am5.ease.inOut(am5.ease.cubic) })
      target.showTooltip()
    }
    else
      chart.goHome();
    previousPolygon = target;
  });

  chart.appear(1000, 100);
  setTimeout(() => zoomToCountry(props.code), 1)
})

onUnmounted(() => chart?.dispose())

function zoomToCountry(code: string) {
  const country = polygonSeries.getDataItemById(code);
  if (!country) return

  const polygon = country.getRaw('mapPolygon')
  previousPolygon = polygon
  polygon.set("active", true);
  polygon.showTooltip()

  const centroid = polygon.geoCentroid()
  if (!centroid) return

  const zoomLevel = code == 'RU' || code == 'US' ? 2 : 3
  chart.animate({ key: "rotationX", to: -centroid.longitude, duration: 1500, easing: am5.ease.inOut(am5.ease.cubic) });
  chart.animate({ key: "rotationY", to: -centroid.latitude, duration: 1500, easing: am5.ease.inOut(am5.ease.cubic) });
  chart.animate({ key: 'zoomLevel', to: zoomLevel, duration: 1500, easing: am5.ease.inOut(am5.ease.cubic) })
}

</script>

<style>
.chartdiv {
  /* width: v-bind(width)px;
  height: v-bind(height)px; */
  width: 500px;
  height: 500px;
}
</style>