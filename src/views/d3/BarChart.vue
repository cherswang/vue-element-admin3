<template>
  <div id="container" class="svg-container" align="center">
    <h1>{{ title }}</h1>
    <svg v-if="redrawToggle === true" :width="svgWidth" :height="svgHeight">
      <g>
        <rect
          v-for="item in data"
          class="bar-positive"
          :key="item[xKey]"
          :x="xScale(item[xKey])"
          :y="yScale(0)"
          :width="xScale.bandwidth()"
          :height="0"
        ></rect>
        <text
          v-for="item in data"
          :key="item[xKey]"
          :x="xScale(item[xKey]) + 30"
          :y="yScale(item[yKey]) - 2"
          fill="red"
        >
          {{ item[xKey] }} {{ item[yKey] }}
        </text>
      </g>
    </svg>
  </div>
</template>
<script>
import * as d3 from "../../utils/d3.v7.min.js";

export default {
  name: "BarChart",
  props: {
    title: String,
    xKey: String,
    yKey: String,
    data: Array,
  },
  data() {
    return {
      svgWidth: 0,
      redrawToggle: true,
    };
  },
  mounted() {
    this.initSvgInfo();
    console.log("666");
  },
  computed: {
    dataMax() {
      return d3.max(this.data, (d) => {
        return d[this.yKey];
      });
    },
    dataMin() {
      return d3.min(this.data, (d) => {
        return d[this.yKey];
      });
    },
    xScale() {
      return d3
        .scaleBand()
        .rangeRound([0, this.svgWidth])
        .padding(0.1)
        .domain(
          this.data.map((d) => {
            return d[this.xKey];
          })
        );
    },
    yScale() {
      return (
        d3
          .scaleLinear()
          .rangeRound([this.svgHeight, 0])
          // .domain([this.dataMin > 0 ? 0 : this.dataMin, this.dataMax]);
          .domain([this.dataMin > 0 ? 0 : this.dataMin + 2, this.dataMax + 2])
      );
    },
    svgHeight() {
      return this.svgWidth / 1.61803398875; // 黄金比例
    },
  },
  methods: {
    initSvgInfo() {
      this.svgWidth = document.getElementById("container").offsetWidth * 0.95;
      this.AddResizeListener();
      this.AnimateLoad();
      console.log("999");
    },
    //调整窗口大小后，300毫秒重新绘制图表
    AddResizeListener() {
      window.addEventListener("resize", () => {
        this.$data.redrawToggle = false;
        setTimeout(() => {
          this.$data.redrawToggle = true;
          this.$data.svgWidth =
            document.getElementById("container").offsetWidth * 0.75;
          this.AnimatedLoad();
        }, 300);
      });
    },
    //绘制柱形
    AnimateLoad() {
      d3.selectAll("rect")
        .data(this.data)
        .transition()
        .delay((d, i) => {
          return i * 150;
        })
        .duration(1000)
        .attr("y", (d) => {
          return this.yScale(d[this.yKey]);
        })
        .attr("height", (d) => {
          return this.svgHeight - this.yScale(d[this.yKey]);
        });

      d3.selectAll("text").data(this.data).enter();
    },
  },
};
</script>
<style scoped>
.bar-positive {
  fill: steelblue;
  transition: r 0.2s ease-in-out;
}

.bar-positive:hover {
  fill: brown;
}

.svg-container {
  display: inline-block;
  position: relative;
  width: 100%;
  padding-bottom: 1%;
  vertical-align: top;
  overflow: hidden;
}
</style>
