<template>
  <div class='page'>
    <div class='top'>
      <div class='control'>
        <h1>控制面板</h1>
        <h3>数据集</h3>
      </div>
      <div class='view1'>
        <h1>视图1</h1>
        <svg id="chart" width="800" height="800"></svg>
      </div>
      <div class='view2'>
        <h1>视图2</h1>
        <svg id="rightchart" width="100" height="300"></svg>
      </div>
    </div>
  </div>
</template>
<script>
import * as d3 from 'd3';
import Edata from '../../static/data.json'
import Vdata from '../../static/Vdata.json'

export default {
  name: 'index',
  data() {
    return {
      myColor: 'red',
      myPadding: 0.2,
      curIndex: 0,
      dataset: null,
      clicktimes: 0,
      selection: undefined,
      lastselection: '',
      linedata: [],
    };
  },
  methods: {
    generateVis1() {
      let that = this;
      console.log('D3开始渲染');
      const svg = d3.select('#chart');
      const width = +svg.attr('width');
      const height = +svg.attr('height');
      const margin = { top: 50, bottom: 150, left: 50, right: 50 };
      const innerwidth = width - margin.left - margin.right;
      const innerheight = height - margin.top - margin.bottom;
      // 设置坐标轴
      const yScale = d3.scaleBand()
        .domain(Vdata.list.map(d => d.name))
        .range([0, innerwidth]);
      const xScale = d3.scaleLinear()
        .domain([1, 50])
        .range([0, innerheight]);
      const g = svg.append('g')
        .attr('id', 'maingroup')
        .attr('transform', `translate(${margin.left},${margin.top})`);
      const yAxis = d3.axisLeft(yScale);
      g.append('g')
        .call(yAxis);
      const xAxis = d3.axisTop(xScale);
      g.append('g')
        .call(xAxis);
      
      
      
    },
    generateVis() {
      let that = this;
      console.log('D3开始渲染');
      const svg = d3.select('#chart');
      const width = +svg.attr('width');
      const height = +svg.attr('height');
      const margin = { top: 50, bottom: 150, left: 50, right: 50 };
      const innerwidth = width - margin.left - margin.right;
      const innerheight = height - margin.top - margin.bottom;
      // 设置坐标轴
      const yScale = d3.scaleBand()
        .domain(Vdata.list.map(d => d.name))
        .range([0, innerwidth]);
      const yScale = d3.scaleLinear()
        .domain([1, 50])
        .range([0, innerheight]);
      const g = svg.append('g')
        .attr('id', 'maingroup')
        .attr('transform', `translate(${margin.left},${margin.top})`);
      const yAxis = d3.axisLeft(yScale);
      g.append('g')
        .call(yAxis);
      const xAxis = d3.axisBottom(yScale);
      g.append('g')
        .attr('transform', `translate(${0},${innerheight})`)
        .call(xAxis);
      // 绘制圆点
      for (var i = 1; i <= 50; i++) {
        let newg = g.append('g').attr("id", `t${i}`);
        newg.selectAll("circle")
          .data(Vdata.list)
          .enter()
          .append("circle")
          .attr("cx", function (d) {
            return yScale(d.name) + 0.5 * yScale.bandwidth();
          })
          .attr("cy", function (d) {
            return yScale(i)
          })
          .attr("r", 5)
          .style("fill", "black");
      };
      //绘制消息边
      let line = d3.line()
        .x(function (d) {
          return yScale(d.name) + 0.5 * yScale.bandwidth();
        })
        .y(function (d) {
          return yScale(d.time);
        });
      Edata.list.forEach(d => {
        that.linedata.push({
          name: d.source,
          time: d.time
        });
        that.linedata.push({
          name: d.target,
          time: d.time + 1
        });
      });
      console.log("this is output linedata");
      console.log(that.linedata);
      g.append('path')
        .attr('class', 'line')
        .attr('d', line(that.linedata))
        .attr('fill', 'none')
        .attr('stroke-width', 3)
        .attr('stroke', 'green');
    },
  },
  mounted() {
    this.generateVis1();
  },
  created() {
    console.log(Edata);
    console.log(Vdata);
  }
};
</script>
<style>
.page {
  width: 100%;
  height: 100%;
  display: flex;
  flex-direction: column;
}

.top {
  height: 75%;
  display: flex;
  flex-direction: row;
}

.buttom {
  height: 25%;
}

.control {
  width: 20%;
  display: flex;
  flex-direction: column;
  gap: 5%;
  padding-right: 30px;
}

.view1 {
  width: 50%;
}

.vew2 {
  width: 30%;
}

.dropdown {
  border: 0.1em solid black;
  padding: 10px 10px;
}

.dropdowntext {
  width: auto;
}

.example-showcase .el-dropdown-link {
  cursor: pointer;
  color: var(--el-color-primary);
  display: flex;
  align-items: center;
}

.varcol {
  display: flex;
  flex-direction: row;
  align-content: center;
}

.input {
  width: auto;
}

.controllist {
  display: flex;
  flex-direction: column;
  gap: 10px;
}
</style>
