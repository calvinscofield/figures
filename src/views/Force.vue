<template>
  <div class="force">
    <!-- <img alt="Vue logo" src="../assets/logo.png"> -->
    <canvas width="960" height="600"></canvas>
    <button @click="force">给力</button>
  </div>
</template>

<script>
import axios from "axios";
import * as d3 from "d3";

export default {
  name: "force",
  data() {
    return {
      simulation: null,
      canvas: null,
      context: null,
      width: 0,
      height: 0,
      radius: 20,
      graph: {},
      transform: d3.zoomIdentity
    };
  },
  methods: {
    force() {
      //this.simulation.restart();
    },
    drawArrow() {
      console.log("drawArrow");
    },
    dragsubject() {
      let i,
        x = this.transform.invertX(d3.event.x),
        y = this.transform.invertY(d3.event.y),
        dx,
        dy;
      console.log(d3.event.x, d3.event.y);
      console.log(x, y);
      console.log(d3.event.transform);
      for (i = this.graph.nodes.length - 1; i >= 0; --i) {
        let point = this.graph.nodes[i];
        dx = x - point.x;
        dy = y - point.y;
        if (dx * dx + dy * dy < this.radius * this.radius) {
          console.log("point");
          point.x = this.transform.applyX(point.x);
          point.y = this.transform.applyY(point.y);
          console.log(point);
          return point;
        }
      }
    },
    ticked() {
      console.log("ticked");
      this.context.save();
      this.context.clearRect(0, 0, this.width, this.height);
      this.context.translate(this.transform.x, this.transform.y);
      this.context.scale(this.transform.k, this.transform.k);

      this.context.beginPath();
      this.graph.links.forEach(this.drawLink);
      this.context.strokeStyle = "#aaa";
      this.context.stroke();

      this.context.beginPath();
      this.graph.nodes.forEach(this.drawNode);
      this.context.fillStyle = "#409EFF";
      this.context.fill();
      this.context.fillStyle = "#303133";
      this.context.textAlign = "center";
      this.context.textBaseline = "middle";
      this.graph.links.forEach(this.drawText2);
      this.graph.nodes.forEach(this.drawText);
      this.context.restore();
    },
    dragstarted() {
      //if (!d3.event.active) this.simulation.alphaTarget(0.3).restart();
      //if (!d3.event.active) this.simulation.alphaTarget(0.3).restart();
      //this.simulation.restart();
      //d3.event.subject.fixed = false
      console.log("dragstarted");
      console.log(d3.event.x, d3.event.y);
      console.log(d3.event.subject.x, d3.event.subject.y);
      //d3.event.subject.x = d3.event.x;
      //d3.event.subject.y = d3.event.y;
      // d3.event.subject.fx = d3.event.subject.x;
      // d3.event.subject.fy = d3.event.subject.y;
    },
    dragged() {
      console.log("dragged");
      console.log(d3.event.x, d3.event.y);
      console.log(d3.event.subject.x, d3.event.subject.y);
      // d3.select(this)
      //   .attr('cx', d3.event.x)
      //   .attr('cy', d3.event.y);
      //d3.event.subject.x = this.transform.invertX(d3.event.x);
      //d3.event.subject.y = this.transform.invertY(d3.event.y);
      d3.event.subject.x = this.transform.invertX(d3.event.x);
      d3.event.subject.y = this.transform.invertY(d3.event.y);
      this.ticked();
    },
    dragended() {
      console.log("dd2", d3.event.subject, d3.event.x, d3.event.y);
      //if (!d3.event.active) this.simulation.alphaTarget(0);
      // d3.event.subject.fx = null;
      // d3.event.subject.fy = null;
      // d3.event.subject.fx = d3.event.x;
      // d3.event.subject.fy = d3.event.y;
    },
    drawLink(d) {
      this.context.moveTo(d.source.x, d.source.y);
      this.context.lineTo(d.target.x, d.target.y);
      let x1 = d.source.x,
        y1 = d.source.y,
        x2 = d.target.x,
        y2 = d.target.y,
        a1 = {},
        a2 = {},
        angle = Math.atan((y2 - y1) / (x2 - x1)) + (x2 < x1 ? Math.PI : 0),
        r1 = 5,
        r2 = this.radius,
        theta = Math.PI / 6,
        angle1 = Math.PI + angle + theta,
        angle2 = Math.PI + angle - theta;
      this.context.moveTo(x2 - r2 * Math.cos(angle), y2 - r2 * Math.sin(angle));
      a1.x = x2 + r1 * Math.cos(angle1) - r2 * Math.cos(angle);
      a1.y = y2 + r1 * Math.sin(angle1) - r2 * Math.sin(angle);
      a2.x = x2 + r1 * Math.cos(angle2) - r2 * Math.cos(angle);
      a2.y = y2 + r1 * Math.sin(angle2) - r2 * Math.sin(angle);
      this.context.lineTo(a1.x, a1.y);
      this.context.moveTo(x2 - r2 * Math.cos(angle), y2 - r2 * Math.sin(angle));
      this.context.lineTo(a2.x, a2.y);
    },
    drawNode(d) {
      this.context.moveTo(d.x + this.radius, d.y);
      this.context.arc(d.x, d.y, this.radius, 0, 2 * Math.PI);
    },
    drawText(d) {
      //this.context.font="30px Arial";
      this.context.fillText(d.name, d.x, d.y);
    },
    drawText2(d) {
      this.context.fillText(
        d.relation,
        (d.source.x + d.target.x) / 2,
        (d.source.y + d.target.y) / 2
      );
    },
    zoomed() {
      console.log("zoomed");
      console.log(this.graph);
      this.transform = d3.event.transform;
      this.ticked();
    }
  },
  components: {},
  mounted() {
    this.simulation = d3.forceSimulation();
    this.canvas = document.querySelector("canvas");
    this.context = this.canvas.getContext("2d");
    this.width = this.canvas.width;
    this.height = this.canvas.height;
    this.simulation
      .force(
        "link",
        d3
          .forceLink()
          .id(function(d) {
            return d.id;
          })
          .distance(300)
      )
      .force("charge", d3.forceManyBody())
      .force("center", d3.forceCenter(this.width / 2, this.height / 2));
    console.log(d3.forceSimulation, d3.json);
    axios
      .get("miserables.json")
      .then(response => {
        console.log(response);
        this.graph = response.data;
        this.simulation
          .nodes(this.graph.nodes)
          .on("tick", this.ticked)
          .on("end", () => {
            console.log("end");
            console.log(this.graph);
          });

        this.simulation.force("link").links(this.graph.links);
        d3.select(this.canvas)
          .call(
            d3
              .drag()
              .container(this.canvas)
              .subject(this.dragsubject)
              .on("start", this.dragstarted)
              .on("drag", this.dragged)
              .on("end", this.dragended)
          )
          .call(
            d3
              .zoom()
              .scaleExtent([1 / 4, 8])
              .on("zoom", this.zoomed)
          );
      })
      .catch(function(error) {
        console.log(error);
      });
  }
};
</script>

<style>
.force {
  text-align: left;
}
</style>