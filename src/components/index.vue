<template>
  <div class='page'>
    <div class='top'>
      <div class='control'>
        <h1>控制面板</h1>
        <el-table :data="Mlist" style="width: 100%" highlight-current-row @current-change="handleCurrentChange">
          <el-table-column prop="0" label="id" width="40"></el-table-column>
          <el-table-column :prop="(index).toString()" :label="index" v-for="index of k" :key="index" />
        </el-table>
      </div>
      <div class='view1'>
        <h1>视图1</h1>
        <el-slider v-model="selectvalue" range show-stops :max="maxx" :min="minn" />
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
import Mdata from '../../static/Mdata.json'

export default {
  name: 'index',
  data() {
    return {
      selectvalue: [0, 10],
      minn: undefined,
      maxx: undefined,
      k: 4,
      Mlist: [],
      currentRow: undefined,
      marColor: "black"
    };
  },
  methods: {
    generateVis2() {
      let that = this;
      console.log('D3开始渲染');
      const svg = d3.select('#chart');
      const width = +svg.attr('width');
      const height = +svg.attr('height');
      const margin = { top: 50, bottom: 150, left: 80, right: 50 };
      const innerwidth = width - margin.left - margin.right;
      const innerheight = height - margin.top - margin.bottom;
      // 初始化元素
      const g = svg.append('g')
        .attr('id', 'maingroup')
        .attr('transform', `translate(${margin.left},${margin.top})`);

      let background = g.append("rect").attr("class", "bg")
      let view = g.append("g").attr("class", "view")
      let linegroup = g.append("g").attr("class", "linegroup");
      let dotgroup = g.append("g").attr("class", "dotgroup");
      let grid = g.append("g").attr("class", "grid")
      let axis = g.append("g").attr("class", "axis")

      // 设置坐标轴
      const xScale = d3.scaleLinear()
        .domain([that.selectvalue[0], that.selectvalue[1]])
        .range([0, innerwidth]);
      const yScale = d3.scaleBand()
        .domain(Vdata.list.map(d => d.name))
        .range([0, innerheight]);
      const yAxis = d3.axisLeft(yScale);
      axis.append('g')
        .call(yAxis);
      const xAxis = d3.axisTop(xScale);
      axis.append('g')
        .call(xAxis);
      // 定义边
      let line = d3.line()
        .y(function (d) {
          return yScale(d.name) + 0.5 * yScale.bandwidth();
        })
        .x(function (d) {
          return xScale(d.time);
        });

      // 定义箭头
      var defs = linegroup.append("defs");

      var arrowMarker = defs.append("marker")
        .attr("id", "arrow")
        .attr("markerUnits", "strokeWidth")
        .attr("markerWidth", "12")
        .attr("markerHeight", "12")
        .attr("viewBox", "0 0 12 12")
        .attr("refX", "6")
        .attr("refY", "6")
        .attr("orient", "auto");

      //绘制边和箭头
      Edata.list.filter(d => (d.time >= that.selectvalue[0] && d.time <= that.selectvalue[1] - 1)).forEach(d => {
        linegroup.append('path')
          .attr('d', line([{
            name: d.source,
            time: d.time
          }, {
            name: d.target,
            time: d.time + 1
          }]))
          .attr('id', `E${d.id}`)
          .attr('fill', 'none')
          .attr('stroke-width', 3)
          // .attr("marker-end", "url(#arrow)")
          .style("stroke", "#DCDCDC")
          .style("stroke-dasharray", 6);

        dotgroup.append("circle")
          .attr("class", `T${d.time}`)
          .attr("cy", yScale(d.source) + 0.5 * yScale.bandwidth())
          .attr("cx", xScale(d.time))
          .attr("r", 5)
          .style("fill", "black");

        dotgroup.append("circle")
          .attr("class", `T${d.time + 1}`)
          .attr("cy", yScale(d.target) + 0.5 * yScale.bandwidth())
          .attr("cx", xScale(d.time + 1))
          .attr("r", 5)
          .style("fill", "black");
      });

      Mdata.markov.forEach(mar => {
        mar.flow.forEach(flow => {
          d3.select(`#E${flow}`)
            .style("stroke", that.marColor);
        })
      });

    },
    handleCurrentChange(currentRow, oldCurrentRow) {
      let that = this;
      console.log(currentRow, oldCurrentRow);
      if (oldCurrentRow != null) {
        Mdata.markov.find(function (e) {
          return e.id === oldCurrentRow[0]
        }).flow.forEach(d => {
          d3.select(`#E${d}`)
            .style("stroke", that.marColor);
        });
      }

      Mdata.markov.find(function (e) {
        return e.id === currentRow[0]
      }).flow.forEach(d => {
        d3.select(`#E${d}`)
          .style("stroke", "green");
      });
    }
  },
  mounted() {
    this.generateVis2();
  },
  updated() {
    d3.select('#maingroup').remove();
    this.generateVis2();
    console.log(this.selectvalue);
    console.log(this.currentRow);
  },
  created() {
    console.log(Edata);
    console.log(Vdata);
    console.log(Mdata);
    console.log(Mdata.list);

    let that = this;
    Mdata.markov.forEach(d => {
      let newlist = [];
      newlist.push(d.id);
      d.flow.forEach((dd, index) => {
        let now = Edata.list.find(function (e) {
          return e.id == dd;
        });
        console.log(now);
        if (index === 0) {
          newlist.push(now.source);
        }
        newlist.push(now.target);
        // 给Edata增加属性
        now.markov = d.id;
      });
      that.Mlist.push(newlist);
    });

    Edata.list.forEach(d => {
      if (that.minn === undefined) {
        that.minn = d.time
      }
      if (that.maxx === undefined) {
        that.maxx = d.time + 1
      }
      that.minn = Math.min(that.minn, d.time);
      that.maxx = Math.max(that.maxx, d.time + 1);
    });
    console.log(that.maxx, that.minn);
    that.selectvalue = [that.minn, that.maxx];
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
  width: 28%;
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
