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
        <el-tooltip v-for="(vertex,index) in vertexesDisp" :key="index" placement="top">
          <div slot="content">
            姓名: {{ index }}
            <br />
            所在地: {{ vertex.di }}
            <br />
            单位: {{ vertex.wo }}
            <br />
          </div>
          <el-avatar
            class="human"
            icon="el-icon-user-solid"
            v-bind:style="{left:vertex.left,top:vertex.top}"
          ></el-avatar>
        </el-tooltip>
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
    vertexesDisp: function () {
      const activeVertexColor = "aliceblue";
      const normalVertexColor = "#b1c1d1";
      const lockVertexColor = "#66b1ff";

      let vertexesDisp = [];
      //console.log(this.vertexes.length);
      for (let i = 0; i < this.vertexes.length; i++) {
        let element = this.vertexes[i];
        let t = {
          left: element.x + "px",
          top: element.y + "px",
          color: element.isActive
            ? activeVertexColor
            : element.isLocked
            ? lockVertexColor
            : normalVertexColor,
        };
        for (let key in element) {
          t[key] = element[key];
        }
        vertexesDisp.push(t);
      }
      return vertexesDisp;
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
.relation {
  background: #768b9c;
  width: 100%;
  height: 100%;
}

.human {
  position: absolute;
  background: #768b9c;
  border: 2px floralwhite solid;
  color: floralwhite;
  transform: translateX(-50%) translateY(-50%);
}

.edge {
  position: absolute;
  transform-origin: top left;
  border-top: 3px solid;
  transition: color 500ms cubic-bezier(0.175, 0.885, 0.32, 1.275);
}
</style>