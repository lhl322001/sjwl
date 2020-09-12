<template>
  <div class="relation">
    <el-container>
      <el-main>
        <div>
          <div
            class="edge"
            v-for="(edge,index) in edgesDisp"
            :key="index"
            v-bind:style="{
                        width: edge.width,
                        transform: edge.transform,
                        top: edge.top,
                        left: edge.left,
                        borderTopColor: edge.color
                    }"
          ></div>
        </div>
        <el-avatar
          class="human"
          icon="el-icon-user-solid"
          v-for="(vertex,index) in vertexesDisp"
          :key="index"
          v-bind:style="{left:vertex.left,top:vertex.top}"
        ></el-avatar>
      </el-main>
    </el-container>
  </div>
</template>

<script>
export default {
  name: "Relation",
  props: {
    graph: Object,
  },
  computed: {
    vertexesInnerDisp: function () {
      const activeVertexColor = "aliceblue";
      const normalVertexColor = "#b1c1d1";
      const lockVertexColor = "#66b1ff";

      let vertexesDisp = [];
      //console.log(this.vertexes.length);
      for (let i = 0; i < this.vertexes.length; i++) {
        let element = this.vertexes[i];
        vertexesDisp.push({
          index: i,
          left: "50%",
          transform: "rotate("+30*index+")",
          color: element.isActive
            ? activeVertexColor
            : element.isLocked
            ? lockVertexColor
            : normalVertexColor,
        });
      }
      return vertexesDisp;
    },
    vertexesOuterDisp: function () {
      const activeVertexColor = "aliceblue";
      const normalVertexColor = "#b1c1d1";
      const lockVertexColor = "#66b1ff";
    },
    edgesDisp: function () {
      let edgesDisp = [];
      this.edges.forEach((edge) => {
        let u = this.vertexes[edge.from];
        let v = this.vertexes[edge.to];

        let dist = Math.sqrt(
          (u.x - v.x) * (u.x - v.x) + (u.y - v.y) * (u.y - v.y)
        );
        let angle = Math.atan2(v.y - u.y, v.x - u.x);
        const selectedEdgeColor = "#000";
        const activeEdgeColor = "blue";
        const normalEdgeColor = "darkgray";
        edgesDisp.push({
          width: dist + "px",
          transform: "rotate(" + angle + "rad)",
          left: u.x + "px",
          top: u.y + "px",
          color: edge.isSelected
            ? selectedEdgeColor
            : edge.isActive
            ? activeEdgeColor
            : normalEdgeColor,
        });
      });
      return edgesDisp;
    },
  },
  created() {
    this.reset();
  },
  methods: {
    reset() {
      this.vertexes = [];
      this.edges = [];
      this.graph.vertexes.forEach((element) => {
        let t = {
          isActive: false,
          isLocked: false,
        };
        for (let key in element) {
          t[key] = element[key];
        }
        this.vertexes.push(t);
      });
      this.graph.edges.forEach((element) => {
        this.edges.push({
          from: element.from,
          to: element.to,
          dist: element.dist,
          index: element.index,
          isActive: false,
          isSelected: false,
        });
      });
    },
  },
  data: function () {
    return {
      vertexes: [],
      edges: [],
    };
  },
};
</script>

<style scoped>
.human {
  position: absolute;
  background: antiquewhite;
  color: blueviolet;
  transform: translateX(-50%) translateY(-50%);
}

.edge {
  position: absolute;
  transform-origin: top left;
  border-top: 2px solid;
  transition: color 500ms cubic-bezier(0.175, 0.885, 0.32, 1.275);
}
</style>