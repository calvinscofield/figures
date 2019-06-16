<template>
  <div class="force2">
    <canvas width="960" height="600" style="border:1px solid #d3d3d3;"></canvas>
  </div>
</template>

<script>
import * as d3 from "d3";

export default {
  name: "force2",
  data() {
    return {
      points: [],
      canvas: null,
      context: null,
      width: 0,
      height: 0,
      radius: 2.5,
      transform: d3.zoomIdentity
    };
  },
  methods: {
    zoomed() {
      console.log("zoomed", d3.event.transform);
      this.transform = d3.event.transform;
      this.render();
    },
    dragsubject() {
      let i,
        x = this.transform.invertX(d3.event.x),
        y = this.transform.invertY(d3.event.y),
        dx,
        dy;
      console.log(d3.event.x, d3.event.y)
      console.log(x, y)
      console.log(d3.event.transform)
      for (i = this.points.length - 1; i >= 0; --i) {
        let point = this.points[i];
        dx = x - point[0];
        dy = y - point[1];
        if (dx * dx + dy * dy < this.radius * this.radius) {
          console.log("point")
          point.x = this.transform.applyX(point[0]);
          point.y = this.transform.applyY(point[1]);
          console.log(point)
          return point;
        }
      }
    },
    dragged() {
      d3.event.subject[0] = this.transform.invertX(d3.event.x);
      d3.event.subject[1] = this.transform.invertY(d3.event.y);
      this.render();
    },
    render() {
      this.context.save();
      this.context.clearRect(0, 0, this.width, this.height);
      this.context.beginPath();
      this.context.translate(this.transform.x, this.transform.y);
      this.context.scale(this.transform.k, this.transform.k);
      this.points.forEach(this.drawPoint);
      this.context.fill();
      this.context.restore();
    },
    drawPoint(point) {
      this.context.moveTo(point[0] + this.radius, point[1]);
      this.context.arc(point[0], point[1], this.radius, 0, 2 * Math.PI);
    },
    phyllotaxis(radius) {
      var theta = Math.PI * (3 - Math.sqrt(5));
      return i => {
        var r = radius * Math.sqrt(i),
          a = theta * i;
        return [
          this.width / 2 + r * Math.cos(a),
          this.height / 2 + r * Math.sin(a)
        ];
      };
    }
  },
  components: {},
  mounted() {
    console.log(d3.zoomIdentity)
    this.canvas = d3.select("canvas");
    this.context = this.canvas.node().getContext("2d");
    this.width = this.canvas.property("width");
    this.height = this.canvas.property("height");

    this.points = d3.range(2000).map(this.phyllotaxis(10));
    this.points.push([0,0])
    
    this.canvas
      .call(
        d3
          .drag()
          .subject(this.dragsubject)
          .on("drag", this.dragged)
      )
      .call(
        d3
          .zoom()
          .scaleExtent([1 / 2, 8])
          .on("zoom", this.zoomed)
      )
      .call(this.render);
  }
};
</script>

<style>
.force2 {
  text-align: left;
}
</style>
